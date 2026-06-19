# Bounded Contexts — NovaTech Assistant
**Elaborado por:** Product Specialist  
**Data:** 05/06/2026  
**Versão:** 1.0  
**Projeto:** DB1 × NovaTech — Assistente de IA para Atendimento  
**Fontes:** Cenário completo (Partes 1 e 2), Anexo A (documentação NovaTech), histórico de discovery (análises de inconsistências, mapa de gaps, jornada do atendente, especificação de requisitos RAG v2.0, decisões arquiteturais ADR-0001 a ADR-0004)

> Este documento define o recorte do domínio do projeto em bounded contexts. Cada contexto tem linguagem ubíqua própria, fronteiras explícitas e relações mapeadas com os demais. O objetivo é que cada membro do time — desenvolvedor, QA, product specialist — saiba exatamente em qual contexto está trabalhando a cada tarefa e o que não é responsabilidade daquele contexto.

---

## Visão Geral — Mapa de Contextos

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          SISTEMA NOVATECH ASSISTANT                         │
│                                                                             │
│  ┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐  │
│  │                  │      │                  │      │                  │  │
│  │   [BC-01]        │─────▶│   [BC-02]        │─────▶│   [BC-03]        │  │
│  │   Ingestão e     │      │   Recuperação    │      │   Geração de     │  │
│  │   Curadoria      │      │   e Busca        │      │   Resposta       │  │
│  │   Documental     │      │   Semântica      │      │   (LLM)          │  │
│  │                  │      │                  │      │                  │  │
│  └──────────────────┘      └──────────────────┘      └──────────────────┘  │
│           │                        │                         │             │
│           │                        │                         │             │
│           ▼                        ▼                         ▼             │
│  ┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐  │
│  │                  │      │                  │      │                  │  │
│  │   [BC-04]        │      │   [BC-05]        │◀─────│   [BC-06]        │  │
│  │   Governança     │      │   Interação com  │      │   Rastreabilidade│  │
│  │   Documental     │      │   o Atendente    │      │   e Feedback     │  │
│  │                  │      │   (Interface)    │      │                  │  │
│  │                  │      │                  │      │                  │  │
│  └──────────────────┘      └──────────────────┘      └──────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

         Fora do sistema (contextos externos da NovaTech):
         ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
         │  SharePoint  │  │  Confluence  │  │  Teams / Bot │
         │  (fonte)     │  │  (fonte)     │  │  Framework   │
         └──────────────┘  └──────────────┘  └──────────────┘
```

---

## BC-01 — Ingestão e Curadoria Documental

### Propósito

Transformar documentos brutos das fontes externas (SharePoint, Confluence, planilhas) em chunks indexáveis, válidos e rastreáveis, aplicando as regras de hierarquia, status e elegibilidade definidas na especificação de requisitos do pipeline de RAG.

### Linguagem ubíqua

| Termo | Definição neste contexto |
|---|---|
| **Documento bruto** | Arquivo original nas fontes externas, antes de qualquer processamento |
| **Chunk** | Fragmento de texto extraído de um documento, com tamanho alvo de ~1.500 tokens, mantendo coerência semântica (não cortar no meio de uma tabela ou seção) |
| **Metadado** | Conjunto mínimo de atributos do documento: nome, versão, responsável, data de atualização, status, nível de hierarquia |
| **Status** | Classificação do documento: `vigente`, `arquivado`, `em-revisao`, `informal`, `quarentena` |
| **Nível** | Hierarquia de confiabilidade: 1 (normativo formal) → 4 (FAQ informal) |
| **Quarentena** | Estado de documentos que chegaram sem metadados mínimos e aguardam triagem do responsável de base |
| **Push de reindexação** | Evento disparado pelo responsável de base indicando que um documento foi publicado ou atualizado e deve ser reprocessado |
| **Embedding** | Representação vetorial de um chunk, usada pelo contexto de Busca Semântica (BC-02) |
| **Pipeline de ingestão** | Sequência: receber → validar metadados → chunkizar → gerar embedding → indexar |

### O que está DENTRO deste contexto

- Recepção de documentos das fontes externas (SharePoint, Confluence, pasta de rede).
- Validação dos metadados mínimos (nome, versão ou data, responsável).
- Aplicação das regras de elegibilidade: formato de arquivo aceito, idioma (português brasileiro), escopo temático.
- Classificação automática do nível hierárquico com base nos metadados recebidos.
- Processamento de chunking: divisão do documento em chunks com ~1.500 tokens, sem cortar tabelas ou seções no meio (problema identificado no protótipo open-source com ChromaDB, conforme ADR-0004).
- Geração de embeddings para cada chunk via Azure OpenAI.
- Indexação dos chunks com metadados no Azure AI Search.
- Gerenciamento do ciclo de vida do documento: indexar, atualizar versão, arquivar, colocar em quarentena.
- Aplicação das regras de exclusão da especificação: bloquear PROC-042-v1 após declaração formal de revogação, bloquear itens específicos do FAQ, sinalizar documentos em revisão.
- Detecção de documentos que cobrem o mesmo tema com status potencialmente conflitante (trigger para BC-04).
- Rollback: capacidade de reverter a indexação de um documento para a versão anterior em caso de problema identificado via feedback (BC-06).

### O que está FORA deste contexto

- Decidir qual versão de um documento é a vigente (decisão do BC-04 — Governança Documental).
- Buscar chunks relevantes para uma pergunta (responsabilidade do BC-02).
- Gerar ou formatar respostas para o atendente (responsabilidade do BC-03).
- Exibir o status dos documentos para o atendente (responsabilidade do BC-05).
- Armazenar o conteúdo original dos documentos para fins de auditoria (o índice guarda os chunks; o original permanece na fonte).
- Gerenciar autenticação e permissões de acesso às fontes externas (SharePoint, Confluence).
- Decidir o que fazer quando um responsável de base não responde em 5 dias úteis (regra de negócio do BC-04).

### Relações com outros contextos

| Contexto | Tipo de relação | Direção | Descrição |
|---|---|---|---|
| BC-02 — Recuperação e Busca | **Downstream** | BC-01 → BC-02 | BC-01 produz os chunks indexados que BC-02 consulta. BC-02 depende da qualidade e dos metadados gerados aqui. |
| BC-04 — Governança Documental | **Conformista** | BC-04 → BC-01 | BC-01 obedece às decisões de status tomadas em BC-04. Quando BC-04 declara um documento revogado ou em quarentena, BC-01 executa a ação. |
| BC-06 — Rastreabilidade e Feedback | **Downstream** | BC-06 → BC-01 | Feedbacks críticos de atendentes processados em BC-06 podem disparar reindexação ou rollback em BC-01. |

---

## BC-02 — Recuperação e Busca Semântica

### Propósito

Dado o texto de uma pergunta do atendente, recuperar os chunks mais relevantes da base indexada, aplicando a hierarquia de fontes e sinalizando conflitos entre documentos — preparando o contexto para o BC-03 (Geração de Resposta).

### Linguagem ubíqua

| Termo | Definição neste contexto |
|---|---|
| **Query** | Pergunta do atendente em linguagem natural, após pré-processamento |
| **Retrieval** | Processo de busca vetorial por similaridade semântica no Azure AI Search |
| **Top-K chunks** | Os K chunks mais relevantes retornados pela busca (K = 5 conforme ADR-0002, ~1.500 tokens cada) |
| **Score de relevância** | Métrica de similaridade semântica entre a query e cada chunk |
| **Context budget** | Limite de tokens disponíveis para os chunks no prompt final: ~8K tokens (ADR-0002) |
| **Conflito de retrieval** | Situação em que dois ou mais chunks recuperados cobrem o mesmo tema com valores ou regras divergentes |
| **Chunk vencedor** | Chunk de maior hierarquia (nível mais baixo numericamente) em caso de conflito detectado |
| **Metadado de vigência** | Conjunto de atributos (nível, status, data) carregado pelo chunk e usado para desempate (ADR-0003) |
| **Threshold de confiança** | Score mínimo de similaridade abaixo do qual o chunk não é incluído no contexto (valor a calibrar em testes) |
| **Pergunta ambígua** | Query que pode ser respondida por chunks de temas distintos com scores similares |

### O que está DENTRO deste contexto

- Receber a query do atendente (originada em BC-05) e pré-processá-la (normalização, identificação de termos técnicos internos como "CT-e", "PROC-042", "tier Gold").
- Executar a busca por similaridade vetorial no Azure AI Search.
- Aplicar filtros de metadados durante o retrieval: excluir chunks de documentos com status `arquivado`; priorizar chunks de documentos com status `vigente` ou `versao-atual`.
- Detectar conflito de retrieval: verificar se dois ou mais chunks recuperados têm o mesmo tema (identificado por overlap de metadados de documento/seção) com valores divergentes.
- Resolver conflito pela hierarquia de nível e data conforme ADR-0003: chunk de nível mais baixo vence; desempate por data mais recente dentro do mesmo nível.
- Montar o pacote de contexto (top-K chunks + metadados + flags de conflito) dentro do context budget de ~8K tokens.
- Sinalizar pergunta sem resultado suficiente (nenhum chunk acima do threshold de confiança) para que BC-03 declare ausência de resposta.
- Sinalizar pergunta ambígua (múltiplos temas com scores semelhantes) para que BC-03 solicite esclarecimento ao atendente.

### O que está FORA deste contexto

- Gerar ou formatar a resposta em linguagem natural (responsabilidade do BC-03).
- Decidir o conteúdo do system prompt (responsabilidade do BC-03).
- Gerenciar o histórico de turnos da conversa (responsabilidade do BC-03 e BC-05).
- Indexar novos documentos ou atualizar chunks (responsabilidade do BC-01).
- Exibir resultados para o atendente (responsabilidade do BC-05).
- Registrar feedback sobre a qualidade da recuperação (responsabilidade do BC-06).
- Decidir se um documento é vigente ou não (responsabilidade do BC-04).

### Relações com outros contextos

| Contexto | Tipo de relação | Direção | Descrição |
|---|---|---|---|
| BC-01 — Ingestão e Curadoria | **Upstream** | BC-01 → BC-02 | BC-02 consulta o índice produzido por BC-01. A qualidade do retrieval depende diretamente da qualidade da ingestão e do chunking. |
| BC-03 — Geração de Resposta | **Downstream** | BC-02 → BC-03 | BC-02 entrega o pacote de contexto (chunks + metadados + flags) para BC-03 compor a resposta. |
| BC-06 — Rastreabilidade e Feedback | **Downstream** | BC-02 → BC-06 | BC-02 registra quais chunks foram recuperados e seus scores para cada consulta, alimentando a rastreabilidade de BC-06. |

---

## BC-03 — Geração de Resposta (LLM)

### Propósito

Compor a resposta final em linguagem natural usando os chunks recuperados por BC-02, respeitando os guardrails de comportamento (nunca inferir valores não documentados, sempre sinalizar conflitos, distinguir fonte informal de normativo) e retornando a resposta com os campos de rastreabilidade obrigatórios.

### Linguagem ubíqua

| Termo | Definição neste contexto |
|---|---|
| **System prompt** | Instrução de comportamento fixa enviada ao LLM a cada requisição (~4K tokens, ADR-0002), contendo guardrails, formato de resposta e regras de priorização |
| **Prompt final** | Composição completa enviada ao LLM: system prompt + chunks de contexto + histórico limitado + pergunta atual |
| **Guardrail** | Restrição de comportamento embutida no system prompt: sem inferência de valores, sinalizar conflitos, FAQ ≠ normativo, escalada só para contatos formais |
| **Resposta estruturada** | Resposta no formato definido: conteúdo + bloco de rastreabilidade (fonte, versão, seção, data, nível de confiança, alertas) |
| **Nível de confiança** | Alta / Moderada / Baixa — determinado pelas regras da spec: alta = Nível 1 sem conflito e < 6 meses; baixa = FAQ ou documento em revisão |
| **Alerta de conflito** | Elemento obrigatório na resposta quando dois chunks com informações divergentes foram recuperados |
| **Declaração de ausência** | Resposta estruturada para os cenários A-G da spec: tema não coberto, documento não indexado, doc em revisão, pergunta ambígua, pergunta multi-partes, fora de escopo, baixa similaridade |
| **Histórico de turnos** | Últimas 3 trocas da conversa incluídas no prompt (ADR-0002), mantidas por BC-05 |
| **Temperatura** | Parâmetro de geração do LLM; deve ser baixa (0.0–0.2) para respostas factuais baseadas em documentação |

### O que está DENTRO deste contexto

- Receber o pacote de contexto de BC-02 (chunks, metadados, flags de conflito, flag de ausência, flag de ambiguidade) e o histórico de turnos de BC-05.
- Construir o prompt final respeitando o context budget total (~128K tokens do GPT-4o, com ~4K para system prompt, ~8K para chunks, o restante disponível para histórico e pergunta, conforme ADR-0001 e ADR-0002).
- Aplicar os guardrails de comportamento no system prompt:
  - G1: nunca afirmar prazo, valor ou condição não explicitamente presente nos chunks recebidos.
  - G2: sempre incluir alerta quando chunks conflitantes foram recuperados (mesmo que o conflito já tenha sido resolvido pela hierarquia em BC-02).
  - G3: respostas baseadas em chunk do FAQ devem incluir aviso de fonte informal obrigatoriamente.
  - G4: não sugerir contatos (e-mail, ramal) presentes apenas no FAQ sem indicar que não estão formalizados em normativo.
- Formatar a resposta estruturada com conteúdo + bloco de rastreabilidade (todos os campos da spec v2.0, Seção 6.1).
- Calcular e incluir o nível de confiança com base nos critérios objetivos da spec (Seção 6.2).
- Gerar as declarações de ausência nos formatos prescritos para cada cenário (A a G da spec v2.0, Seção 4.2).
- Tratar perguntas com múltiplas partes: responder cada parte individualmente, declarar ausência nas partes sem cobertura.
- Tratar perguntas ambíguas: apresentar as interpretações possíveis com base nos temas identificados pelos chunks recuperados e solicitar esclarecimento.

### O que está FORA deste contexto

- Buscar ou selecionar chunks (responsabilidade do BC-02).
- Exibir a resposta ao atendente (responsabilidade do BC-05).
- Registrar a resposta para auditoria (responsabilidade do BC-06).
- Gerenciar a interface de chat, histórico persistido ou notificações (responsabilidade do BC-05).
- Decidir se um documento é elegível para indexação (responsabilidade do BC-01 e BC-04).
- Interagir diretamente com as fontes externas (SharePoint, Confluence).
- Executar cálculos de frete ou SLA — o LLM reproduz os valores documentados, não os calcula com lógica própria.

### Relações com outros contextos

| Contexto | Tipo de relação | Direção | Descrição |
|---|---|---|---|
| BC-02 — Recuperação e Busca | **Upstream** | BC-02 → BC-03 | BC-03 depende inteiramente do pacote de contexto produzido por BC-02. A qualidade da resposta é limitada pela qualidade do retrieval. |
| BC-05 — Interação com o Atendente | **Downstream** | BC-03 → BC-05 | BC-03 entrega a resposta estruturada para BC-05 formatar e exibir no Teams ou painel web. |
| BC-05 — Interação com o Atendente | **Upstream** (histórico) | BC-05 → BC-03 | BC-05 fornece o histórico de até 3 turnos da conversa para inclusão no prompt. |
| BC-06 — Rastreabilidade e Feedback | **Downstream** | BC-03 → BC-06 | BC-03 passa a resposta gerada (com fontes citadas) para BC-06 registrar no log de auditoria. |

---

## BC-04 — Governança Documental

### Propósito

Gerenciar o estado da base documental: decidir o status de vigência de cada documento, resolver contradições entre versões, processar atualizações dos responsáveis de base, e garantir que as regras de ciclo de vida dos documentos (publicação, revisão, arquivamento, quarentena) sejam cumpridas dentro dos SLAs definidos.

### Linguagem ubíqua

| Termo | Definição neste contexto |
|---|---|
| **Responsável de base** | Pessoa designada por cada área (Operações, Compliance, Comercial, Atendimento) com autoridade para declarar status de documentos, aprovar triagem e notificar o pipeline de novas publicações |
| **Declaração de vigência** | Ato formal do responsável de base confirmando qual versão de um documento está ativa |
| **Conflito documental** | Estado em que dois ou mais documentos cobrem o mesmo tema com regras divergentes e sem hierarquia formal declarada (ex: PROC-042 v1 × v2) |
| **Resolução de conflito** | Processo de decisão formal que resulta em uma declaração de vigência e arquivamento da versão obsoleta |
| **Triagem** | Avaliação de um documento em quarentena para determinar se será indexado, ajustado ou descartado |
| **Ciclo de atualização** | Sequência: publicação na fonte → notificação pelo responsável de base → processamento em BC-01 → disponibilidade no assistente |
| **SLA de atualização** | Prazo máximo entre a publicação e a disponibilidade no assistente (4h para normativos Nível 1; 48h para FAQ) |
| **Painel de gaps** | Visão agregada de temas sem cobertura documental, alimentada pelos registros de perguntas sem resposta de BC-06 |
| **Revisão periódica** | Reunião trimestral entre o time de produto e os responsáveis de base para revisar a lista de exclusões e priorizar novos documentos |

### O que está DENTRO deste contexto

- Manter o registro central de status de cada documento indexado (tabela mestre de documentos com: nome, versão, nível, status, data de última atualização, responsável de base, data de próxima revisão).
- Processar declarações de vigência dos responsáveis de base: quando a Diretoria Comercial declara a PROC-042-v2 como vigente e arquiva a v1, BC-04 registra essa decisão e dispara a execução em BC-01.
- Detectar e registrar conflitos documentais: quando BC-01 sinaliza dois documentos que cobrem o mesmo tema, BC-04 abre um item de resolução e notifica o responsável de base correspondente.
- Gerenciar o processo de quarentena: notificar responsável de base em até 24h, aguardar decisão por 5 dias úteis, descartar após prazo sem resposta.
- Controlar o SLA de atualização: monitorar o tempo entre a notificação do responsável de base e a disponibilidade no assistente; disparar alertas para o gestor de produto quando o SLA é violado.
- Gerenciar o painel de gaps: consolidar perguntas sem resposta por tema (recebidas de BC-06), apresentar tendência mensal e status de cada gap (aberto, em cobertura, resolvido).
- Executar a revisão trimestral da lista de exclusões e comunicar ao BC-01 as alterações resultantes.
- Controlar o prazo de alerta de desatualização: quando um documento Nível 1 ou 2 não é atualizado por mais de 6 meses sem confirmação de vigência, BC-04 notifica o responsável de base e instrui BC-03 a incluir aviso de possível desatualização nas respostas baseadas nele.
- Gerenciar os critérios de entrada e saída do Nível 1 (documento sem atualização por 18+ meses sem confirmação de vigência sai do Nível 1).

### O que está FORA deste contexto

- Executar o chunking ou a indexação dos documentos (responsabilidade do BC-01).
- Executar a busca ou o retrieval (responsabilidade do BC-02).
- Gerar respostas para o atendente (responsabilidade do BC-03).
- Criar, editar ou publicar documentos nas fontes externas (SharePoint, Confluence) — isso é responsabilidade das áreas da NovaTech, fora do sistema.
- Validar o conteúdo dos documentos (se o conteúdo está correto do ponto de vista do negócio) — isso é responsabilidade das áreas da NovaTech.
- Gerenciar permissões de acesso ao SharePoint ou Confluence.

### Relações com outros contextos

| Contexto | Tipo de relação | Direção | Descrição |
|---|---|---|---|
| BC-01 — Ingestão e Curadoria | **Controlador** | BC-04 → BC-01 | BC-04 decide o status dos documentos; BC-01 executa as ações de indexação, arquivamento e quarentena conforme instruído. |
| BC-06 — Rastreabilidade e Feedback | **Upstream** | BC-06 → BC-04 | BC-04 consome os registros de perguntas sem resposta e feedbacks de BC-06 para alimentar o painel de gaps e priorizar ações de cobertura documental. |
| **Fontes externas** (SharePoint, Confluence) | **Anti-Corruption Layer** | BC-04 ↔ Externo | BC-04 é a camada que protege o sistema interno das inconsistências das fontes externas. Traduz o estado das fontes (caótico, sem hierarquia) para o estado interno do sistema (estruturado, com status formal). |

---

## BC-05 — Interação com o Atendente (Interface)

### Propósito

Gerenciar a experiência do atendente ao usar o assistente: receber perguntas em linguagem natural, orquestrar o fluxo entre os contextos de backend (BC-02 e BC-03), apresentar as respostas formatadas no Microsoft Teams e no painel web, e manter o histórico de conversa da sessão.

### Linguagem ubíqua

| Termo | Definição neste contexto |
|---|---|
| **Turno** | Um par pergunta-resposta dentro de uma sessão de conversa |
| **Sessão** | Conjunto de turnos de uma conversa contínua; o histórico é limitado a 3 turnos para o contexto do LLM (ADR-0002) |
| **Canal** | Superfície de interação: Teams Bot ou painel web interno |
| **Bloco de rastreabilidade** | Componente visual colapsável exibido abaixo da resposta com os campos de fonte, versão, confiança e alertas |
| **Banner de atualização** | Aviso exibido quando o tema consultado está com documento em processo de reindexação |
| **Ação rápida** | Botões de feedback inline: ✅ Resposta útil / ⚠ Incompleta / ❌ Incorreta |
| **Escopo declarado** | Mensagem padrão exibida quando o atendente pergunta sobre temas fora do escopo da fase 1 |
| **Onboarding** | Fluxo de introdução ao assistente exibido na primeira sessão do atendente, comunicando escopo e limitações |

### O que está DENTRO deste contexto

- Receber a pergunta do atendente via Microsoft Teams Bot (integração com Bot Framework) ou painel web interno (React).
- Gerenciar a sessão de conversa: manter histórico de até 3 turnos para envio ao BC-03; persistir histórico completo para BC-06.
- Orquestrar o fluxo de uma consulta: encaminhar query para BC-02 → receber pacote de contexto → encaminhar para BC-03 → receber resposta estruturada → formatar e exibir.
- Formatar a resposta para o canal: garantir legibilidade no Teams (sem tabelas de mais de 4 colunas horizontalmente, conforme spec Seção 10), exibir bloco de rastreabilidade colapsável com alertas sempre visíveis.
- Exibir os três formatos de resposta da jornada do atendente: resposta completa (fluxo principal), marcação de fonte informal (fallback C), declaração de ausência estruturada (fallback A e B).
- Apresentar as ações rápidas de feedback (✅ / ⚠ / ❌) após cada resposta e encaminhar seleção para BC-06.
- Exibir banner de atualização quando BC-04 sinaliza que um documento sobre o tema consultado está em processo de reindexação.
- Exibir o escopo declarado quando o atendente pergunta sobre temas fora do escopo da fase 1.
- Exibir fluxo de onboarding na primeira sessão do atendente.
- Não armazenar dados de clientes: qualquer dado pessoal de cliente mencionado na pergunta não deve ser persistido.

### O que está FORA deste contexto

- Buscar documentos ou gerar respostas (responsabilidades de BC-02 e BC-03).
- Indexar documentos ou gerenciar status documental (BC-01 e BC-04).
- Processar ou rotear feedbacks (BC-06 recebe o feedback, BC-05 apenas coleta e encaminha).
- Autenticar o atendente — a autenticação é gerenciada pelo Azure AD / Microsoft 365 (sistema externo).
- O painel de métricas e histórico do dashboard administrativo — esse painel tem lógica própria consumindo BC-06 e pode ser considerado uma interface separada do mesmo BC-05 ou uma extensão de BC-06 (a definir pelo Tech Lead).
- Enviar notificações proativas ao atendente (ex: "seu feedback foi resolvido") — essas notificações são disparadas por BC-06, mas entregues via o canal Teams administrado por BC-05.

### Relações com outros contextos

| Contexto | Tipo de relação | Direção | Descrição |
|---|---|---|---|
| BC-02 — Recuperação e Busca | **Orquestrador** | BC-05 → BC-02 | BC-05 inicia o fluxo de consulta enviando a query para BC-02. |
| BC-03 — Geração de Resposta | **Orquestrador** | BC-05 → BC-03 e BC-03 → BC-05 | BC-05 fornece histórico para BC-03 e recebe a resposta estruturada de volta. |
| BC-04 — Governança Documental | **Receptor de alertas** | BC-04 → BC-05 | BC-04 notifica BC-05 quando um documento está em atualização para exibição do banner. |
| BC-06 — Rastreabilidade e Feedback | **Produtor de eventos** | BC-05 → BC-06 | BC-05 encaminha feedbacks coletados e histórico de sessão para BC-06 registrar. |
| **Microsoft Teams / Bot Framework** | **Anti-Corruption Layer** | BC-05 ↔ Externo | BC-05 abstrai as particularidades do Bot Framework e do Teams para o restante do sistema. |

---

## BC-06 — Rastreabilidade e Feedback

### Propósito

Registrar, rotear e monitorar tudo o que o assistente disse, como o atendente avaliou cada resposta, e quais perguntas não puderam ser respondidas — garantindo auditabilidade, conformidade com a LGPD e um ciclo contínuo de melhoria da base documental e do comportamento do assistente.

### Linguagem ubíqua

| Termo | Definição neste contexto |
|---|---|
| **Registro de consulta** | Log imutável de cada turno: pergunta anonimizada, resposta gerada, chunks utilizados, versão do assistente, timestamp |
| **Registro de ausência** | Log de perguntas que não obtiveram resposta, com cenário de ausência classificado (A a G da spec) e tema inferido |
| **Feedback** | Avaliação do atendente sobre uma resposta: ✅ útil / ⚠ incompleta / ❌ incorreta, com descrição opcional |
| **Ciclo de feedback** | Processo completo: registro → roteamento → análise pela área → ação → notificação ao atendente |
| **SLA de feedback** | Prazo de resolução: 5 dias úteis para feedbacks críticos (❌ em tema de risco alto); 15 dias úteis para feedbacks moderados (⚠) |
| **Roteamento** | Determinação automática da área responsável com base no tema do feedback (Operações, Compliance, Comercial, Atendimento) |
| **Painel de qualidade** | Dashboard com métricas de performance do assistente: taxa de resposta, feedbacks negativos, tempo médio de resolução de feedbacks |
| **Anonimização** | Substituição de dados pessoais de clientes (nome, CNPJ, nº contrato) por tokens genéricos antes do armazenamento |
| **Versão do assistente** | Identificador da versão do sistema que gerou cada resposta, registrado em todos os logs para viabilizar comparação antes/depois de atualizações |
| **Retenção** | 90 dias para registros de consulta; 24 meses para feedbacks, governança documental e auditorias |

### O que está DENTRO deste contexto

- Receber e armazenar os registros de consulta de BC-03 (pergunta anonimizada, resposta, fontes, versão do assistente, timestamp).
- Receber e armazenar os registros de ausência: perguntas sem resposta com cenário classificado, tema inferido e identificador de chamado quando disponível.
- Receber feedbacks de BC-05 (✅ / ⚠ / ❌ com descrição e tipo de problema).
- Rotear feedbacks automaticamente para a área responsável com base no tema.
- Monitorar SLA de resolução de feedbacks e disparar alertas automáticos para o gestor de produto e responsável de base quando prazos são violados.
- Notificar o atendente que originou o feedback quando o ciclo for encerrado (notificação entregue via BC-05).
- Manter o log imutável de auditoria: indexação/remoção de documentos (recebido de BC-01), alterações de status (recebido de BC-04), consultas, feedbacks, rollbacks.
- Aplicar regras de retenção: 90 dias para consultas, 24 meses para feedbacks e eventos de governança.
- Aplicar regras de anonimização antes de qualquer armazenamento (LGPD).
- Fornecer dados agregados para o painel de gaps de BC-04: volume de perguntas sem resposta por tema, tendência mensal, temas mais consultados.
- Fornecer métricas de performance para o painel de qualidade: taxa de resposta com confiança alta, taxa de feedbacks negativos, tempo médio de resolução.
- Viabilizar auditoria: logs acessíveis ao time de produto e ao gestor de qualidade documental da NovaTech, não visíveis ao atendente.

### O que está FORA deste contexto

- Exibir os logs ao atendente (os logs são internos e administrativos).
- Decidir quais documentos atualizar ou arquivar com base no feedback (decisão do BC-04).
- Executar as correções nos documentos (responsabilidade das áreas da NovaTech).
- Gerar respostas para o atendente (BC-03).
- Gerenciar o processo editorial de atualização do FAQ (responsabilidade da Coordenação de Atendimento e BC-04).

### Relações com outros contextos

| Contexto | Tipo de relação | Direção | Descrição |
|---|---|---|---|
| BC-01 — Ingestão e Curadoria | **Downstream** | BC-06 → BC-01 | Feedbacks críticos resolvidos que implicam reindexação disparam ação em BC-01. |
| BC-03 — Geração de Resposta | **Receptor** | BC-03 → BC-06 | BC-06 recebe cada resposta gerada por BC-03 para registro no log de auditoria. |
| BC-04 — Governança Documental | **Produtor de insumos** | BC-06 → BC-04 | BC-06 alimenta BC-04 com dados de gaps recorrentes e feedback não resolvido para priorização editorial. |
| BC-05 — Interação com o Atendente | **Receptor de eventos** | BC-05 → BC-06 | BC-06 recebe feedbacks coletados por BC-05 e histórico de sessão para registro. |
| BC-05 — Interação com o Atendente | **Produtor de notificações** | BC-06 → BC-05 | BC-06 dispara notificações de encerramento de ciclo para entrega via BC-05. |

---

## Resumo das Fronteiras e Relações

### Tabela de responsabilidades por decisão-chave

| Decisão / Ação | Contexto responsável |
|---|---|
| Chunkizar e indexar um documento | BC-01 |
| Decidir se um documento é vigente ou revogado | BC-04 |
| Bloquear um item do FAQ da indexação | BC-04 (decisão) + BC-01 (execução) |
| Buscar chunks relevantes para uma pergunta | BC-02 |
| Resolver conflito entre dois chunks recuperados | BC-02 (hierarquia) |
| Formatar e enviar o prompt ao LLM | BC-03 |
| Calcular o nível de confiança da resposta | BC-03 |
| Exibir a resposta ao atendente no Teams | BC-05 |
| Coletar o feedback do atendente | BC-05 |
| Rotear o feedback para a área responsável | BC-06 |
| Monitorar SLA de resolução de feedback | BC-06 |
| Notificar o atendente de encerramento do feedback | BC-06 (dispara) + BC-05 (entrega) |
| Atualizar o painel de gaps da base documental | BC-04 (consome) + BC-06 (produz) |
| Disparar rollback de um documento com problema | BC-06 (identifica) + BC-04 (decide) + BC-01 (executa) |

### Mapa de dependências críticas

```
BC-04 (Governança)
    │
    │ status dos documentos
    ▼
BC-01 (Ingestão) ──── chunks + metadados ────▶ BC-02 (Busca)
                                                    │
                                        pacote de contexto
                                                    │
                                                    ▼
                              histórico ────▶ BC-03 (LLM)
                              (BC-05)             │
                                          resposta estruturada
                                                    │
                                                    ▼
                                             BC-05 (Interface)
                                                    │
                                            feedback coletado
                                                    │
                                                    ▼
                                             BC-06 (Rastreabilidade)
                                                    │
                               ┌────────────────────┤
                               │                    │
                    gaps p/ BC-04          rollback p/ BC-01
```

---

## Itens Fora do Escopo do Sistema (Contextos Externos)

Os elementos abaixo existem na realidade da NovaTech mas estão **fora do escopo do sistema** a ser desenvolvido. O sistema se integra a eles mas não os controla.

| Elemento externo | Como o sistema se integra | Contexto que faz a interface |
|---|---|---|
| SharePoint corporativo | Fonte de leitura de documentos via API | BC-01 consome; BC-04 monitora |
| Confluence (wiki interna) | Fonte de leitura de páginas via API | BC-01 consome; BC-04 triagem |
| Pasta de rede com planilhas | Fonte de leitura de arquivos Excel | BC-01 consome |
| Microsoft Teams + Bot Framework | Canal de entrega da interface ao atendente | BC-05 abstrai |
| Azure AD / Microsoft 365 | Autenticação e identidade do atendente | BC-05 delega |
| Sistema de chamados (Azure DevOps) | Fonte de identificador do chamado para rastreabilidade | BC-06 consome quando disponível |
| Portal do Cliente (portal.novatech.com.br) | Canal do cliente para abertura de chamados de devolução | Fora do escopo — mencionado nos documentos como destino de orientação ao cliente |
| Áreas internas da NovaTech (Operações, Compliance, Comercial) | Responsáveis de base; decisões editoriais; publicação de documentos | BC-04 interage via processo humano |
| Conteúdo dos contratos individuais de clientes | Não indexado; não faz parte do escopo da base documental | Fora do escopo da fase 1 |

---

*Documento elaborado com base no cenário completo do projeto (Partes 1 e 2), no Anexo A (documentação simulada da NovaTech), e em todo o histórico de discovery: mapa de temas e gaps, análises de inconsistências, cruzamento FAQ × normativos, jornada do atendente com fluxos e guardrails, e especificação de requisitos do pipeline de RAG v2.0. Deve ser revisado pelo Tech Lead antes do início do desenvolvimento para validação das fronteiras com a arquitetura técnica escolhida (ADR-0001 a ADR-0004).*
