# Bounded Contexts — NovaTech Assistant
**Versão:** 2.0 (substitui v1.0 de 05/06/2026)  
**Data:** 05/06/2026  
**Elaborado por:** Product Specialist · revisão Tech Lead  
**Alterações em relação à v1.0:** Corrigidas AMB-01 (contrato BC-02→BC-03), AMB-02 (tipos de ambiguidade), AMB-03 (threshold definido), AMB-05 (fronteira de feedback), AMB-06 (detecção de multi-partes), AMB-07 (definição de turno), AMB-09 (streaming), AMB-10 (nível no glossário do sistema), AMB-11 (painel de gaps), AMB-13 (confirmação de vigência vs data do documento).

> Os termos usados neste documento são os definidos no **Glossário v2.0**. Em caso de dúvida sobre um termo, consultar o glossário — não redefinir localmente.

---

## Mapa de Contextos

```
┌──────────────────────────────────────────────────────────────────────────┐
│                        SISTEMA NOVATECH ASSISTANT                        │
│                                                                          │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐   │
│  │   [BC-01]       │────▶│   [BC-02]       │────▶│   [BC-03]       │   │
│  │   Ingestão e    │     │   Recuperação   │     │   Geração de    │   │
│  │   Curadoria     │     │   e Busca       │     │   Resposta      │   │
│  │   Documental    │     │   Semântica     │     │   (LLM)         │   │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘   │
│          │                       │                        │             │
│          ▼                       ▼                        ▼             │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐   │
│  │   [BC-04]       │     │   [BC-05]       │◀────│   [BC-06]       │   │
│  │   Governança    │     │   Interação     │     │   Rastreab. e   │   │
│  │   Documental    │     │   (Interface)   │     │   Feedback      │   │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘   │
│                                                                          │
│  Contrato entre BC-02 e BC-03 — Pacote de Contexto:                    │
│  { chunks[], has_conflict, is_ambiguous, ambiguous_topics[], no_result } │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## BC-01 — Ingestão e Curadoria Documental

### Propósito

Transformar documentos brutos das fontes externas (SharePoint, Confluence, planilhas) em chunks indexáveis, válidos e rastreáveis, aplicando as regras de hierarquia, status e elegibilidade.

### O que está DENTRO

- Receber documentos das fontes externas via push de reindexação do responsável de base.
- Validar metadados mínimos (nome, versão ou data, responsável).
- Aplicar critérios de elegibilidade: formato aceito (`.pdf` com texto, `.docx`, `.md`, `.xlsx`, `.html`), idioma português brasileiro, escopo temático de atendimento ao cliente.
- Classificar o nível hierárquico (1–4) com base nos metadados.
- Executar chunking em ~1.500 tokens, sem cortar tabelas ou seções no meio (solução para problema identificado no protótipo, ADR-0004).
- Gerar embeddings via Azure OpenAI para cada chunk.
- Indexar chunks com metadados no Azure AI Search, incluindo: `doc_name`, `doc_version`, `section`, `last_updated_at`, `last_confirmed_at`, `level`, `status`, `conflict_flag`.
- Gerenciar ciclo de vida: indexar, atualizar versão, arquivar (status `arquivado`), colocar em quarentena (status `quarentena`).
- Aplicar exclusões: bloquear PROC-042-v1 após declaração formal de revogação; bloquear itens específicos do FAQ conforme lista de BC-04.
- Detectar documentos que cobrem o mesmo tema com status potencialmente conflitante → sinalizar para BC-04.
- Executar rollback: reverter indexação de um documento para versão anterior quando instruído por BC-04 (que recebeu decisão de BC-06).

### O que está FORA

- Decidir qual versão de documento é vigente (BC-04).
- Buscar chunks para uma pergunta (BC-02).
- Gerar respostas (BC-03).
- Exibir status de documentos ao atendente (BC-05).
- Gerenciar autenticação nas fontes externas.

### Relações com outros contextos

| Contexto | Direção | Descrição |
|---|---|---|
| BC-02 | BC-01 → BC-02 | BC-01 produz o índice que BC-02 consulta |
| BC-04 | BC-04 → BC-01 | BC-04 instrui ações de status; BC-01 executa |
| BC-06 | BC-06 → BC-01 | Feedbacks críticos resolvidos disparam rollback em BC-01 via BC-04 |

---

## BC-02 — Recuperação e Busca Semântica

### Propósito

Dado o texto de uma pergunta do atendente, pré-processá-la, recuperar os chunks mais relevantes acima do threshold, detectar ambiguidades e conflitos, e montar o pacote de contexto estruturado para BC-03.

### Termos específicos deste contexto

*(Definições completas no Glossário v2.0, Seção II)*

| Termo | Referência |
|---|---|
| Query | Glossário Seção II |
| Score de relevância | Glossário Seção II |
| Threshold de confiança | Glossário Seção II — **valor inicial: 0,72** |
| Ambiguidade de score | Glossário Seção II |
| Pergunta multi-partes | Glossário Seção II |
| Pacote de contexto | Glossário Seção II |
| Chunk vencedor | Glossário Seção II |

### O que está DENTRO

**Pré-processamento da query:**
- Normalizar a query (remoção de pontuação excessiva, correção de siglas conhecidas: CT-e, PROC, POL, SLA, tier Gold/Silver/Standard).
- Detectar pergunta multi-partes por heurística: presença de conectores "e", "+", vírgula entre entidades de domínios distintos. Limite: até 4 partes. Se mais de 4 partes forem detectadas, processar as 4 primeiras e registrar truncamento no log.
- Se multi-partes detectada: executar retrieval independente por parte e montar resultados agregados.

**Retrieval:**
- Executar busca vetorial por cosine similarity no Azure AI Search.
- Aplicar filtros de metadados: excluir chunks com status `arquivado`; priorizar `vigente` e `versao-atual`.
- Retornar todos os chunks com score ≥ 0,72 (threshold), até o máximo de 5. Se menos de 5 chunks superarem o threshold, retornar apenas os que superam. Se nenhum superar, sinalizar `no_result: true`.

**Detecção de ambiguidade de score:**
- Verificar se os top-2 chunks retornados têm scores com diferença < 0,05 e pertencem a domínios distintos.
- Se sim: setar `is_ambiguous: true` e popular `ambiguous_topics[]` com os temas dos chunks candidatos (ex: `["devolução","frete_especial"]`).
- **BC-02 não detecta ambiguidade semântica** — essa detecção é responsabilidade de BC-03.

**Detecção de conflito:**
- Verificar se dois ou mais chunks com scores ≥ threshold cobrem o mesmo tema (mesmo `doc_name` ou mesma `section` em documentos distintos) com valores divergentes.
- Se sim: selecionar chunk vencedor (menor nível; empate → data `last_confirmed_at` mais recente); setar `has_conflict: true`; incluir metadados do chunk perdedor no campo `conflict_doc`.
- O alerta de conflito NÃO menciona o estado do documento no SharePoint — apenas que existe outro documento na base com valor diferente.

**Montagem do pacote de contexto:**
- Estrutura entregue a BC-03:
```json
{
  "chunks": [...],          // top-K chunks com metadados completos
  "has_conflict": bool,     // true se conflito detectado
  "conflict_doc": {...},    // metadados do chunk perdedor (quando has_conflict)
  "is_ambiguous": bool,     // true se ambiguidade DE SCORE detectada
  "ambiguous_topics": [...],// temas candidatos (quando is_ambiguous)
  "no_result": bool,        // true se nenhum chunk ≥ threshold
  "is_multi_part": bool,    // true se multi-partes detectada
  "parts": [...]            // chunks agrupados por parte (quando is_multi_part)
}
```

### O que está FORA

- Detectar ambiguidade semântica (BC-03).
- Gerar ou formatar resposta (BC-03).
- Gerenciar histórico de conversa (BC-05).
- Indexar documentos (BC-01).
- Decidir status de documentos (BC-04).

### Relações com outros contextos

| Contexto | Direção | Descrição |
|---|---|---|
| BC-01 | BC-01 → BC-02 | BC-02 consulta o índice produzido por BC-01 |
| BC-03 | BC-02 → BC-03 | BC-02 entrega o pacote de contexto para BC-03 |
| BC-06 | BC-02 → BC-06 | BC-02 registra chunks recuperados e scores para auditoria |

---

## BC-03 — Geração de Resposta (LLM)

### Propósito

Compor o prompt final, enviar ao GPT-4o via Azure OpenAI com streaming, detectar ambiguidades semânticas nos chunks recebidos, aplicar guardrails, e retornar a resposta estruturada com rastreabilidade completa.

### Termos específicos deste contexto

| Termo | Referência |
|---|---|
| System prompt | Glossário Seção II |
| Nível de confiança | Glossário Seção II — tabela de critérios |
| Streaming | Glossário Seção II |
| Declaração de ausência | Glossário Seção II |
| Ambiguidade semântica | Glossário Seção II |
| Turno | Glossário Seção II |

### O que está DENTRO

**Recebimento e validação do pacote:**
- Receber pacote de contexto de BC-02 e histórico de 3 turnos de BC-05.

**Detecção de ambiguidade semântica:**
- Verificar, independentemente da flag `is_ambiguous` de BC-02, se os chunks retornados cobrem temas distintos para uma query vaga (ex: a palavra "prazo" em chunks de POL-001, PROC-042-v2 e SLA-2024 com scores ≥ threshold mas pertencentes a domínios diferentes).
- Se detectada: gerar resposta de esclarecimento com opções ao atendente, independentemente de BC-02 ter ou não sinalizado `is_ambiguous`.
- **Combinação:** se `is_ambiguous: true` vindo de BC-02 E ambiguidade semântica detectada — usar a lista `ambiguous_topics` de BC-02 como base para as opções, complementada pela análise de conteúdo.

**Montagem do prompt final (conforme ADR-0002):**
- System prompt (~4K tokens): guardrails G1–G4, instruções de formato, critérios de nível de confiança, instrução de streaming.
- Chunks do pacote de contexto (~8K tokens).
- Histórico de 3 turnos.
- Pergunta atual.

**Guardrails (embutidos no system prompt):**
- G1: nunca afirmar prazo, valor ou condição não presente nos chunks recebidos. Proibido: "provavelmente", "normalmente", "deve ser", "costuma ser", "em geral", "aproximadamente" em afirmações sobre prazos, valores e condições contratuais.
- G2: sempre incluir campo `conflict_notice` quando `has_conflict: true`, identificando o documento divergente por nome e versão. Formato: "Existe outro documento na base com valor diferente para este tema: [nome] [versão], [seção]. A resposta acima usa [nome vencedor] por ser [mais recente / maior hierarquia]." O alerta NÃO menciona o estado do documento no SharePoint.
- G3: respostas baseadas exclusivamente em chunks de Nível 4 (FAQ) incluem aviso obrigatório de fonte informal.
- G4: não sugerir contatos (e-mail, ramal) presentes apenas no FAQ sem indicar que não estão formalizados em normativo.

**Cálculo do nível de confiança:**
- Usar tabela de critérios do Glossário Seção II (Alta / Moderada / Baixa).
- O campo `last_confirmed_at` dos metadados do chunk vencedor é a base para o critério de 6 meses — não a `last_updated_at`.

**Formato da resposta estruturada:**
```
[conteúdo factual]

---
FONTE: [doc_name] · [doc_version] · [section]
Atualizado em: [last_updated_at]
Confiança: [Alta / Moderada / Baixa]
[conflict_notice] (se has_conflict)
[informal_notice] (se fonte = Nível 4)
[stale_notice] (se last_confirmed_at > 6 meses)
```

**Entrega via streaming:**
- Resposta entregue progressivamente (token a token) via Server-Sent Events.
- Meta TTFT (Time to First Token): < 3 segundos.
- Meta resposta completa: < 30 segundos (percentil 95, carga normal).

### O que está FORA

- Buscar ou selecionar chunks (BC-02).
- Exibir a resposta ao atendente (BC-05).
- Registrar a resposta para auditoria (BC-06).
- Executar cálculos de frete ou SLA.
- Decidir elegibilidade de documentos (BC-01, BC-04).

### Relações com outros contextos

| Contexto | Direção | Descrição |
|---|---|---|
| BC-02 | BC-02 → BC-03 | BC-03 recebe o pacote de contexto |
| BC-05 | BC-05 → BC-03 | BC-05 fornece histórico de 3 turnos |
| BC-05 | BC-03 → BC-05 | BC-03 entrega resposta estruturada via streaming |
| BC-06 | BC-03 → BC-06 | BC-03 emite evento de resposta para registro de auditoria |

---

## BC-04 — Governança Documental

### Propósito

Gerenciar o estado da base documental: decidir vigência, resolver contradições, processar atualizações dos responsáveis de base, e manter o painel de gaps com dados fornecidos por BC-06.

### O que está DENTRO

- Manter tabela mestre de documentos: `doc_name`, `doc_version`, `level`, `status`, `last_updated_at`, `last_confirmed_at`, `responsible`, `next_review_date`.
- Processar declarações de vigência dos responsáveis de base → atualizar `last_confirmed_at` e instruir BC-01 a atualizar o metadado no índice.
- Detectar e registrar conflitos documentais sinalizados por BC-01 → notificar responsável de base → aguardar resolução → instruir BC-01.
- Gerenciar quarentena: notificar responsável em até 24h; aguardar 5 dias úteis; descartar após prazo sem resposta.
- Monitorar SLA de atualização (4h para Nível 1) → alertar gestor de produto em caso de violação.
- **Painel de gaps:** BC-04 consome dados agregados de BC-06 (volume de perguntas sem resposta por tema, tendência mensal) e usa esses dados para decisões editoriais. BC-04 **não calcula** as métricas — BC-06 calcula e entrega. BC-04 **não exibe** o painel — a exibição é responsabilidade de uma interface administrativa fora do escopo da fase 1.
- Gerenciar alerta de desatualização: quando `last_confirmed_at` de um documento Nível 1 supera 6 meses, notificar responsável de base e instruir BC-03 (via metadado no índice) a incluir `stale_notice` nas respostas.
- Executar revisão trimestral da lista de exclusões com responsáveis de base.

### O que está FORA

- Calcular métricas do painel de gaps (BC-06).
- Exibir o painel de gaps (interface administrativa — fora do escopo fase 1).
- Executar chunking ou indexação (BC-01).
- Gerar respostas (BC-03).

### Relações com outros contextos

| Contexto | Direção | Descrição |
|---|---|---|
| BC-01 | BC-04 → BC-01 | BC-04 instrui ações; BC-01 executa |
| BC-06 | BC-06 → BC-04 | BC-06 fornece métricas agregadas de gaps; BC-04 toma decisões editoriais com base nelas |

---

## BC-05 — Interação com o Atendente (Interface)

### Propósito

Gerenciar a experiência do atendente: receber perguntas, orquestrar o fluxo entre BCs, apresentar respostas formatadas no Teams e painel web, manter histórico de sessão, e **coletar** (mas não processar) feedbacks.

### Termos específicos deste contexto

| Termo | Definição |
|---|---|
| **Turno** | Par pergunta-resposta. Ver Glossário Seção II para definição completa incluindo interações de esclarecimento. |
| **Sessão** | Conjunto de turnos de uma conversa contínua. Histórico persistido em BC-06; histórico enviado ao LLM = últimos 3 turnos. |
| **TTFT** | Time to First Token — primeiro token visível ao atendente. Meta: < 3s. Exibido no timer durante carregamento. |
| **Evento de feedback** | Payload emitido por BC-05 quando o atendente clica em ✅/⚠️/❌, contendo: `response_id`, `feedback_type`, `description` (opcional), `chamado_id` (quando disponível), `assistant_version`, `timestamp`. |

### O que está DENTRO

- Receber pergunta do atendente via Teams Bot ou painel web.
- Manter histórico de sessão: armazenar todos os turnos localmente; enviar apenas os 3 mais recentes ao BC-03.
- Orquestrar: query → BC-02 → BC-03 → formatar → exibir.
- Renderizar resposta via streaming: exibir tokens progressivamente à medida que chegam de BC-03. Exibir timer de TTFT durante carregamento.
- Desabilitar campo de input durante o processamento de streaming para evitar envio concorrente. Reabilitar quando o último token for recebido.
- Formatar bloco de rastreabilidade: exibir `doc_name`, `doc_version`, `section`, `last_updated_at`, nível de confiança. `conflict_notice`, `informal_notice` e `stale_notice` sempre visíveis (não ocultáveis). Versão do assistente NOT exibida ao atendente — apenas nos logs.
- Formatar respostas para legibilidade no Teams: tabelas > 4 colunas convertidas em listas estruturadas.
- **Coletar feedback:** exibir botões ✅/⚠️/❌ após cada resposta com conteúdo factual; ao clicar, emitir evento de feedback para BC-06 e exibir confirmação visual ("Feedback registrado"). BC-05 **não processa nem roteia** o feedback — apenas coleta e emite o evento.
- Exibir banner de atualização quando BC-04 sinaliza que documento sobre o tema consultado está em reindexação.
- Exibir onboarding na primeira sessão: escopo coberto, temas não cobertos na fase 1, chips de sugestão clicáveis (lista configurável de perguntas de exemplo, mantida pelo time de produto; clicar preenche o campo mas não envia automaticamente).
- Não persistir dados pessoais de clientes mencionados nas perguntas.

### O que está FORA

- Buscar documentos ou gerar respostas (BC-02, BC-03).
- Processar ou rotear feedbacks (BC-06).
- Autenticar atendente (Azure AD, externo).
- Exibir painel administrativo de métricas ou gaps (interface fora do escopo fase 1).

### Relações com outros contextos

| Contexto | Direção | Descrição |
|---|---|---|
| BC-02 | BC-05 → BC-02 | BC-05 inicia o fluxo enviando a query |
| BC-03 | BC-05 ↔ BC-03 | BC-05 fornece histórico; BC-03 entrega resposta via streaming |
| BC-04 | BC-04 → BC-05 | BC-04 envia sinais para banners de atualização |
| BC-06 | BC-05 → BC-06 | BC-05 emite eventos de feedback e histórico de sessão |
| BC-06 | BC-06 → BC-05 | BC-06 dispara notificações de encerramento de feedback para entrega |

---

## BC-06 — Rastreabilidade e Feedback

### Propósito

Registrar imutavelmente todas as respostas, ausências e feedbacks; calcular métricas agregadas de qualidade e gaps; rotear feedbacks; monitorar SLAs de resolução; e fornecer dados para BC-04 e para o painel administrativo.

### O que está DENTRO

- Receber e armazenar registros de consulta de BC-03: pergunta anonimizada, resposta, chunks utilizados com scores, `has_conflict`, nível de confiança, `assistant_version`, timestamp.
- Registrar perguntas sem resposta com cenário classificado (A–G), tema inferido e `chamado_id`.
- Receber eventos de feedback de BC-05; rotear automaticamente para área responsável com base no tema.
- Monitorar SLA de feedback: críticos (❌ em tema de risco alto) → 5 dias úteis; moderados (⚠️) → 15 dias úteis. Alertas automáticos em caso de violação.
- Notificar atendente de encerramento de ciclo de feedback via BC-05.
- **Calcular métricas agregadas:** volume de perguntas sem resposta por tema, tendência mensal, taxa de feedbacks negativos, taxa de respostas com confiança Alta, TTFT médio, tempo de resposta completa no percentil 95. Entregar para BC-04 (gaps) e painel administrativo.
- Manter log imutável de auditoria (indexações, remoções, alterações de status, consultas, feedbacks, rollbacks).
- Aplicar retenção: 90 dias para registros de consulta; 24 meses para feedbacks e eventos de governança.
- Aplicar anonimização LGPD antes de qualquer armazenamento.

### O que está FORA

- Exibir logs ao atendente.
- Decidir quais documentos atualizar (BC-04).
- Executar correções nos documentos (áreas da NovaTech).
- Gerar respostas (BC-03).

### Relações com outros contextos

| Contexto | Direção | Descrição |
|---|---|---|
| BC-01 | BC-06 → BC-01 | Feedbacks críticos resolvidos que implicam reindexação instruem BC-04, que instrui BC-01 |
| BC-03 | BC-03 → BC-06 | BC-06 recebe evento de resposta para auditoria |
| BC-04 | BC-06 → BC-04 | BC-06 entrega métricas agregadas de gaps para decisões editoriais de BC-04 |
| BC-05 | BC-05 → BC-06 | BC-06 recebe eventos de feedback e histórico de sessão |
| BC-05 | BC-06 → BC-05 | BC-06 dispara notificações de encerramento para entrega via BC-05 |

---

## Tabela de responsabilidades por decisão-chave

| Decisão / Ação | Responsável |
|---|---|
| Chunkizar e indexar documento | BC-01 |
| Decidir se documento é vigente ou revogado | BC-04 |
| Detectar ambiguidade de score | BC-02 |
| Detectar ambiguidade semântica | BC-03 |
| Detectar pergunta multi-partes | BC-02 (pré-processamento) |
| Resolver conflito entre chunks (chunk vencedor) | BC-02 |
| Calcular nível de confiança | BC-03 |
| Emitir alerta de conflito na resposta | BC-03 (G2) |
| Entregar resposta via streaming | BC-03 → BC-05 |
| Exibir timer de TTFT | BC-05 |
| Desabilitar input durante streaming | BC-05 |
| Coletar feedback (botões ✅/⚠️/❌) | BC-05 |
| Processar e rotear feedback | BC-06 |
| Calcular métricas de gaps | BC-06 |
| Exibir painel de gaps | Interface administrativa (fora do escopo fase 1) |
| Confirmar vigência de documento (`last_confirmed_at`) | BC-04 (processa declaração do responsável de base) |
| Disparar aviso de desatualização (> 6 meses) | BC-04 (via metadado) → BC-03 (inclui na resposta) |
| Rollback de indexação | BC-06 (identifica) → BC-04 (decide) → BC-01 (executa) |

---

*Bounded Contexts v2.0. Resolve AMB-01, AMB-02, AMB-03, AMB-05, AMB-06, AMB-07, AMB-09, AMB-11, AMB-13 da revisão técnica.*
