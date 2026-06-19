# requirements.md — Query Endpoint
## NovaTech Assistant · v2.0

**Módulo:** Query Endpoint  
**Versão:** 2.0 (substitui v1.0 de 05/06/2026)  
**Data:** 05/06/2026  
**Contextos cobertos:** Atendimento ao Cliente · Logística de Frete · Políticas de Devolução · SLAs e Contratos · Gestão Documental e Governança do Conhecimento  
**Alterações em relação à v1.0:** Resolvidas AMB-01 (contrato BC-02→BC-03), AMB-02 (tipos de ambiguidade), AMB-03 (threshold definido), AMB-04 (critérios de nível de confiança), AMB-05 (fronteira de feedback), AMB-06 (detecção multi-partes), AMB-07 (definição de turno), AMB-08 (formato do alerta de conflito), AMB-09 (streaming e SLAs), AMB-12 (chips de sugestão), AMB-13 (confirmação de vigência), AMB-14 (versão do assistente), AMB-15 (terminologia tier), AMB-16 (input desabilitado), AMB-17 (condições normais).

> Termos usados neste documento são definidos no **Glossário v2.0**. Decisões arquiteturais referenciadas como ADR-XXXX são imutáveis neste módulo.

---

## Outcomes

### Para o atendente

- Obter uma resposta fundamentada em documentação oficial em menos de 30 segundos (resposta completa) e com o primeiro conteúdo visível em menos de 3 segundos (TTFT), sem precisar abrir nenhuma fonte externa manualmente.
- Saber exatamente de onde veio cada informação recebida — documento, versão e seção — para confiar na resposta e usá-la diretamente no atendimento.
- Receber orientação clara e imediata quando o assistente não tem informação confiável, em vez de receber silêncio ou uma resposta inventada.
- Identificar sem ambiguidade quando duas fontes divergem, para escalar ao supervisor com contexto suficiente.
- Reduzir o tempo de busca de ~12 minutos para menos de 2 minutos por chamado.

### Para o negócio

- Aumentar consistência das respostas ao cliente final eliminando a variação por consulta de versões distintas de documentos.
- Reduzir volume de escaladas desnecessárias ao supervisor: meta de 15% para abaixo de 8% dos chamados que consultam o assistente.
- Criar rastreabilidade auditável de quais informações foram fornecidas em quais momentos.
- Tornar visível o gap documental ao registrar sistematicamente as perguntas sem resposta.

---

## Scope Boundaries

### In Scope

**Atendimento ao Cliente (BC-05 / BC-03)**
- Receber perguntas em linguagem natural via Microsoft Teams Bot ou painel web interno.
- Processar perguntas nas categorias: prazos de entrega, regras de frete especial, política de devolução, SLAs por tier de cliente.
- Processar perguntas multi-partes (até 4 partes, detectadas em BC-02 por heurística de conectores).
- Retornar resposta estruturada com: conteúdo factual, bloco de rastreabilidade (doc_name, doc_version, section, last_updated_at, nível de confiança), alertas contextuais (conflict_notice, informal_notice, stale_notice).
- Entregar resposta via streaming com TTFT < 3s e resposta completa < 30s (p95).
- Desabilitar campo de entrada durante processamento de streaming.
- Solicitar esclarecimento em perguntas ambíguas (ambiguidade de score detectada em BC-02 OU ambiguidade semântica detectada em BC-03).
- Declarar ausência estruturada com motivo e próximo passo quando o tema não tiver cobertura confiável.
- Manter histórico de 3 turnos para contextualizar respostas (ver definição de Turno no Glossário Seção II).
- **Coletar** feedback do atendente (botões ✅/⚠️/❌) e emitir evento de feedback para BC-06. O endpoint não processa nem roteia feedback.
- Exibir onboarding na primeira sessão com escopo, limitações e chips de sugestão configuráveis.

**Logística de Frete (BC-02 / BC-03)**
- Recuperar e apresentar regras de frete especial conforme PROC-042-v2 vigente.
- Alertar quando multiplicadores ou condições forem divergentes entre chunks recuperados.

**Políticas de Devolução (BC-02 / BC-03)**
- Recuperar e apresentar regras da POL-001 v3.1.
- Distinguir devolução padrão de carga danificada em trânsito.

**SLAs e Contratos (BC-02 / BC-03)**
- Recuperar e apresentar SLAs por tier (Gold, Silver, Standard).
- Confirmar que não existe tier Platinum.

**Gestão Documental (BC-01 / BC-02 / BC-04)**
- Aplicar hierarquia de fontes (níveis 1–4 conforme Glossário Seção II).
- Sinalizar resposta baseada exclusivamente em Nível 4 (FAQ).
- Sinalizar documento com `last_confirmed_at` > 6 meses.

---

### Out of Scope

**Contextos externos ao sistema**
- Calcular fretes com valores reais de chamados.
- Consultar dados de contratos individuais de clientes.
- Criar, editar ou arquivar documentos nas fontes externas.
- Autenticar o atendente (Azure AD).
- Acessar o Portal do Cliente.

**Contextos externos ao escopo do produto**
- Tratar exceções de Gestão de Riscos (ramal 4500 orientado, não representado).
- Reproduzir processos do Jurídico/Sinistros.
- Realizar negociações comerciais individuais.
- Fornecer decisões sobre contratos de clientes específicos.

**Funcionalidades de outros módulos**
- Indexar ou reindexar documentos (BC-01).
- Gerenciar status documental (BC-04).
- Processar, rotear ou armazenar feedbacks (BC-06).
- Exibir painel administrativo de métricas ou gaps.
- Enviar notificações proativas de resolução de feedback.

---

## Constraints

### Fundamentação em evidências

Toda resposta deriva exclusivamente dos chunks recuperados. Nenhuma informação pode ser adicionada além do que está nos chunks do pacote de contexto da consulta corrente.

### Exibição obrigatória de fontes

Cada resposta com conteúdo factual deve incluir obrigatoriamente:

| Campo | Descrição |
|---|---|
| `doc_name` | Nome completo do documento |
| `doc_version` | Versão do documento |
| `section` | Seção ou item de referência |
| `last_updated_at` | Data da última atualização do conteúdo do documento |
| Nível de confiança | Alta / Moderada / Baixa (critérios abaixo) |

`conflict_notice`, `informal_notice` e `stale_notice` são sempre visíveis quando presentes — não podem ser ocultados pelo atendente.

A versão do assistente (`assistant_version`) **não é exibida ao atendente** — registrada apenas nos logs de BC-06.

### Critérios de nível de confiança

| Nível | Critérios objetivos |
|---|---|
| **Alta** | Resposta baseada exclusivamente em documento(s) de Nível 1; sem conflito detectado; `last_confirmed_at` ≤ 6 meses |
| **Moderada** | Nível 1 com `last_confirmed_at` > 6 meses; OU Nível 2 ou 3 validado; OU Nível 1 com conflito resolvido (chunk vencedor selecionado) |
| **Baixa** | Nível 4 (FAQ); OU documento com status `em-revisao`; OU `last_confirmed_at` desconhecida |

### Ausência de alucinação

O endpoint não usa termos de estimativa ou inferência — "provavelmente", "normalmente", "costuma ser", "deve ser", "em geral", "aproximadamente" — em afirmações sobre prazos, valores numéricos, condições contratuais ou procedimentos operacionais. Quando a informação não estiver disponível, a resposta é uma declaração de ausência estruturada.

### Tratamento explícito de contradições

Quando `has_conflict: true` no pacote de contexto, o endpoint deve: (a) usar o chunk vencedor como base da resposta, (b) incluir `conflict_notice` no bloco de rastreabilidade com o seguinte formato: "Existe outro documento na base com valor diferente para este tema: [doc_name] [doc_version], [section]. A resposta acima usa [doc_name vencedor] por ser [mais recente / maior hierarquia]." O alerta não menciona o estado do documento no SharePoint.

### Atualização documental

O endpoint opera com base documental atualizada em até 24 horas após publicação oficial de novo documento ou revisão (Nível 1). Durante reindexação, exibir banner informativo ao atendente.

### Tempo de resposta e streaming

O assistente usa streaming (Server-Sent Events). Dois SLAs se aplicam:
- **TTFT (Time to First Token):** < 3 segundos, medido do lado do cliente (Teams ou painel web), percentil 95, em condições normais (até 200 req/hora, latência de rede interna ≤ 50ms para serviços Azure).
- **Resposta completa:** < 30 segundos, mesma base de medição.

O timer exibido ao atendente durante o carregamento mede o TTFT.

### Estado de carregamento

O campo de entrada do atendente deve ser desabilitado durante o processamento de streaming para evitar envio concorrente. O campo é reabilitado quando o último token da resposta for recebido.

### Coleta de feedback

O endpoint é responsável por exibir os botões ✅/⚠️/❌ após cada resposta com conteúdo factual e emitir o evento de feedback para BC-06 quando acionados. O evento de feedback contém: `response_id`, `feedback_type` (useful/incomplete/incorrect), `description` (texto livre opcional), `chamado_id` (quando disponível via integração), `assistant_version`, `timestamp`. Confirmação visual deve ser exibida ao atendente após o clique.

### Chips de sugestão (onboarding)

O onboarding exibe chips de perguntas de sugestão configuráveis (lista mantida pelo time de produto). Ao clicar em um chip, o texto é inserido no campo de entrada mas **não enviado automaticamente** — o atendente confirma o envio.

### Idioma

O endpoint opera exclusivamente em português brasileiro. Perguntas em outros idiomas recebem mensagem informando a limitação sem tentativa de resposta no idioma original.

### Dados pessoais

O endpoint não persiste dados pessoais de clientes (nome, CNPJ, número de contrato, endereço) mencionados nas perguntas. Qualquer dado de cliente é descartado após o processamento da consulta corrente.

### Contexto de conversa

O histórico enviado ao LLM inclui os 3 turnos mais recentes (6 mensagens). Ver definição de Turno no Glossário Seção II para tratamento de interações de esclarecimento.

---

## Prior Decisions

### ADR-0001 — Azure OpenAI (GPT-4o) como modelo principal

GPT-4o via Azure OpenAI. Janela de 128K tokens suporta o orçamento de contexto definido. Não rediscutir neste módulo.

### ADR-0002 — Orçamento de contexto limitado e histórico restrito

Prompt composto por: ~4K tokens (system prompt) + ~8K tokens (top-5 chunks de ~1.500 tokens) + histórico de 3 turnos + pergunta atual. Orçamento fixo — não expandir sem nova ADR.

### ADR-0003 — Preservação de documentos contraditórios com priorização por metadado de vigência

Documentos contraditórios são preservados com status diferenciado. O endpoint prioriza o chunk de menor nível; dentro do mesmo nível, o com `last_confirmed_at` mais recente. A resolução é executada em BC-02. O LLM recebe apenas o chunk vencedor e a flag `has_conflict` para incluir o `conflict_notice`.

### ADR-0004 — Arquitetura RAG com Azure AI Search + Azure OpenAI

Busca vetorial por cosine similarity no Azure AI Search. Threshold inicial: 0,72. Retorna até top-5 chunks com score ≥ threshold. Embeddings gerados via Azure OpenAI na ingestão. Protótipo com ChromaDB validou a abordagem e identificou o problema de chunking em tabelas — resolvido no pipeline de ingestão.

---

## Verification Criteria

### VC-01 — Resposta com fonte obrigatória

**Dado** que existe na base um documento Nível 1 vigente cobrindo o tema com confiança alta,  
**quando** o atendente submete uma pergunta sobre esse tema,  
**então** a resposta deve conter: (a) conteúdo factual correto conforme o documento, (b) `doc_name` completo, (c) `doc_version`, (d) `section`, (e) `last_updated_at`, (f) nível de confiança "Alta" — todos visíveis sem ação adicional do atendente.

---

### VC-02 — Resposta baseada em documento informal sinalizada

**Dado** que o único documento disponível sobre o tema é o FAQ-Atendimento (Nível 4),  
**quando** o atendente submete uma pergunta sobre esse tema,  
**então** a resposta deve: (a) apresentar o conteúdo, (b) exibir `informal_notice` visível (não ocultável), (c) exibir nível de confiança "Baixa", (d) recomendar confirmação com supervisor.

---

### VC-03 — Tratamento de documentos contraditórios

**Dado** que dois documentos indexados cobrem o mesmo tema com valores divergentes (ex: multiplicadores da PROC-042 v1 e v2),  
**quando** o atendente faz uma pergunta sobre esse tema,  
**então** a resposta deve: (a) apresentar apenas o valor do chunk vencedor, (b) exibir `conflict_notice` com nome e versão do documento divergente e justificativa de desempate, (c) não mencionar o estado do documento no SharePoint, (d) não combinar valores dos dois documentos.

---

### VC-04 — Ausência de resposta por tema não coberto

**Dado** que nenhum chunk supera o threshold (0,72) para a query,  
**quando** o atendente submete uma pergunta sobre esse tema,  
**então** a resposta deve: (a) declarar ausência explicitamente, (b) identificar o motivo (tema não indexado / documento não disponível / documento em revisão), (c) indicar área responsável quando identificável, (d) oferecer opção de registrar como gap — sem fornecer conteúdo factual.

---

### VC-05 — Ausência de alucinação em informações numéricas

**Dado** que a pergunta solicita valor numérico não presente em nenhum chunk com score ≥ 0,72,  
**quando** o endpoint processa a pergunta,  
**então** a resposta não contém nenhum valor numérico sobre o tema. Nenhuma frase de estimativa aparece na resposta.

---

### VC-06 — Pergunta multi-partes

**Dado** que a pergunta contém 2 ou mais partes distintas identificáveis por heurística de conectores (ex: "prazo de devolução e multiplicador de frete para o Norte"),  
**quando** o atendente submete essa pergunta,  
**então** a resposta deve: (a) estruturar em blocos separados por parte, (b) responder cada parte com fonte independente, (c) declarar ausência nas partes sem cobertura, (d) não misturar fontes de domínios distintos numa afirmação. Se o sistema não detectar a multi-parte (fallback), deve responder pelo chunk de maior score sem gerar blocos separados — comportamento registrado no log como `multi_part_detection_failed`.

---

### VC-07 — Pergunta ambígua (ambos os tipos)

**Dado** que a pergunta é ambígua por score (top-2 chunks com scores com diferença < 0,05 de domínios distintos) OU ambígua semanticamente (query vaga como "Qual o prazo?"),  
**quando** o atendente submete essa pergunta,  
**então** o endpoint não escolhe interpretação arbitrária. A resposta apresenta as interpretações possíveis identificadas (com fonte de cada opção) e solicita esclarecimento. Nenhum conteúdo factual é fornecido antes da escolha do atendente.

---

### VC-08 — TTFT e tempo de resposta completa

**Dado** que o assistente está em condições normais (até 200 req/hora, latência de rede interna ≤ 50ms para serviços Azure),  
**quando** o atendente submete qualquer pergunta,  
**então**: (a) o primeiro token visível ao atendente deve chegar em menos de 3 segundos (TTFT, percentil 95); (b) a resposta completa deve ser entregue em menos de 30 segundos (percentil 95). Ambas as métricas medidas do lado do cliente.

---

### VC-09 — Documento com confirmação de vigência desatualizada

**Dado** que o metadado `last_confirmed_at` do documento fonte tem data superior a 6 meses em relação à data corrente,  
**quando** o atendente recebe a resposta,  
**então** a resposta deve exibir `stale_notice` explícito: "Este documento não teve sua vigência confirmada há mais de 6 meses. Confirme com a área [responsável] antes de usar em contexto contratual." O aviso é baseado em `last_confirmed_at`, não em `last_updated_at`.

---

### VC-10 — Pergunta fora do escopo

**Dado** que a pergunta envolve tema explicitamente fora do escopo (negociação comercial, sinistro, contrato de cliente específico),  
**quando** o atendente submete essa pergunta,  
**então** a resposta deve: (a) reconhecer o tema, (b) informar que o assistente responde exclusivamente com base na documentação interna e que aquele tema está fora do escopo, (c) indicar área ou canal correto, (d) não fornecer conteúdo factual.

---

### VC-11 — Integridade do bloco de rastreabilidade

**Dado** que o atendente recebe qualquer resposta com conteúdo factual,  
**quando** o atendente visualiza o bloco de rastreabilidade,  
**então**: `conflict_notice`, `informal_notice` e `stale_notice` estão sempre visíveis quando presentes — não podem ser ocultados. O conteúdo do bloco corresponde exatamente aos metadados do chunk vencedor utilizado para gerar aquela afirmação. A versão do assistente não é exibida ao atendente.

---

### VC-12 — Histórico de conversa e definição de turno

**Dado** que o atendente está em uma conversa com mais de 3 turnos anteriores,  
**quando** o atendente submete uma nova pergunta,  
**então** o contexto enviado ao LLM contém exatamente os 3 turnos mais recentes (6 mensagens). Interações de esclarecimento (pergunta ambígua + opções + escolha do atendente) são tratadas como 1 turno único. Turnos anteriores ao limite de 3 não influenciam a resposta.

---

### VC-13 — Coleta de feedback

**Dado** que o atendente recebe uma resposta com conteúdo factual,  
**quando** o atendente clica em ✅, ⚠️ ou ❌,  
**então**: (a) um evento de feedback é emitido para BC-06 com os campos: `response_id`, `feedback_type`, `description` (se preenchida), `chamado_id` (se disponível), `assistant_version`, `timestamp`; (b) confirmação visual é exibida ao atendente imediatamente; (c) o endpoint não executa processamento adicional do feedback.

---

### VC-14 — Estado de carregamento e campo de entrada

**Dado** que o atendente enviou uma pergunta e o endpoint está processando via streaming,  
**quando** o primeiro token ainda não foi recebido pelo cliente,  
**então**: (a) o campo de entrada está desabilitado; (b) o botão de envio está desabilitado; (c) o timer de TTFT está visível e incrementando. Quando o último token for recebido, o campo de entrada é reabilitado automaticamente.

---

### VC-15 — Chips de sugestão no onboarding

**Dado** que o atendente está em sua primeira sessão com o assistente,  
**quando** o atendente visualiza a tela de onboarding,  
**então** os chips de sugestão configuráveis estão visíveis. Ao clicar em um chip, o texto correspondente é inserido no campo de entrada mas não enviado automaticamente — o atendente deve pressionar o botão de envio para submeter.
