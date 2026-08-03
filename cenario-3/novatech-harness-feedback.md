# Harness de Produto — NovaTech Assistant
## Fluxo de Feedback: da Captura ao Monitoramento Pós-Implantação

**Versão:** 1.0  
**Data:** 05/06/2026  
**Elaborado por:** Product Specialist Sênior  
**Destinatários:** Product · Tech Lead · QA · Engenharia · Delivery Manager · Operações · Compliance · Comercial

---

## Visão Geral do Fluxo

```
[1] CAPTURA           Interface Teams / painel web
       ↓
[2] ARMAZENAMENTO     BC-06 · log imutável + contexto completo
       ↓
[3] TRIAGEM           Automática (24h) + humana (48h para críticos)
       ↓
[4] CLASSIFICAÇÃO     Causa raiz identificada por categoria
       ↓
[5] DEFINIÇÃO         Tipo de ação + responsável + prioridade
       ↓
[6] IMPLEMENTAÇÃO     Alteração na camada correspondente
       ↓
[7] TESTES            Suite de regressão + caso novo derivado do feedback
       ↓
[8] APROVAÇÃO         Revisão humana conforme nível de risco
       ↓
[9] IMPLANTAÇÃO       Deploy controlado com rollback definido
       ↓
[10] COMUNICAÇÃO      Notificação ao atendente que originou o feedback
       ↓
[11] MONITORAMENTO    Acompanhamento das métricas afetadas por 14 dias
```

---

## Etapa 1 — Captura do Feedback

### 1.1 Tipos de feedback disponíveis

O atendente acessa o feedback por um painel expansível abaixo de cada resposta no Teams e no painel web. O painel mostra primeiro os três botões rápidos; ao clicar em ⚠️ ou ❌, o painel expande automaticamente para o formulário detalhado.

**Botões rápidos (sempre visíveis após cada resposta factual):**

| Botão | Label | Significado |
|---|---|---|
| ✅ | Resposta útil | O atendente usou a resposta diretamente no atendimento sem ressalvas |
| ⚠️ | Tem um problema | A resposta tem algum problema — expande formulário de detalhe |
| ❌ | Resposta incorreta | A resposta está factualmente errada ou é prejudicial — expande formulário de detalhe |

**Categorias do formulário de detalhe (seleção única obrigatória para ⚠️ e ❌):**

| ID | Categoria | Descrição exibida ao atendente |
|---|---|---|
| `F01` | Resposta incorreta | A informação está errada — contradiz o que sei ou o que o cliente confirmou |
| `F02` | Fonte inadequada | A fonte citada não sustenta o que foi dito, ou é informal sendo tratada como oficial |
| `F03` | Informação incompleta | A resposta está certa, mas faltou uma condição, exceção ou detalhe importante |
| `F04` | Resposta desatualizada | A informação pode ter sido correta antes, mas existe versão mais recente |
| `F05` | Não encontrou informação existente | O assistente disse que não sabia, mas eu sei que o documento existe |
| `F06` | Pergunta fora do escopo | O assistente tentou responder algo que não deveria |
| `F07` | Resposta útil com observação | Útil, mas com detalhe menor a registrar (equivale a ✅ com contexto) |

**Campo de comentário textual (opcional, exibido sempre que formulário expande):**  
Placeholder: "Descreva o problema ou indique a informação correta (opcional — máximo 500 caracteres)."

### 1.2 Contexto obrigatório capturado automaticamente

O sistema registra os seguintes campos em todo evento de feedback, sem ação do atendente. Os campos são coletados pelo BC-05 e enviados ao BC-06 no evento de feedback:

| Campo | Tipo | Origem | Descrição |
|---|---|---|---|
| `feedback_id` | UUID | Gerado pelo sistema | Identificador único do evento de feedback |
| `session_id` | UUID | BC-05 (sessão ativa) | Identificador da sessão de conversa |
| `turn_id` | UUID | BC-05 | Identificador do turno (par pergunta-resposta) avaliado |
| `query_text` | string | BC-05 (pergunta enviada) | Texto da pergunta do atendente, anonimizado conforme LGPD |
| `answer_text` | string | BC-03 (resposta gerada) | Texto completo da resposta entregue ao atendente |
| `chunks_retrieved` | array | BC-02 (context_package) | Lista de chunks recuperados: `doc_name`, `doc_version`, `section`, `score`, `document_level` |
| `source_document_cited` | object | BC-03 (campo da resposta) | `doc_name`, `doc_version`, `section` citados na resposta |
| `confidence_level` | enum | BC-03 / pipeline | `Alta`, `Moderada` ou `Baixa` |
| `has_conflict` | boolean | BC-02 (context_package) | Se conflito documental foi detectado no retrieval |
| `conflict_notice_included` | boolean | BC-03 / validador | Se `conflict_notice` foi incluído na resposta |
| `is_absence_declaration` | boolean | BC-03 | Se a resposta foi uma declaração de ausência |
| `timestamp` | ISO 8601 | Sistema | Data e hora do clique no botão de feedback |
| `user_profile` | string | Azure AD (anonimizado) | Perfil do atendente: `atendente_junior`, `atendente_senior`, `supervisor` |
| `prompt_version` | string | Config do pipeline | Versão do system prompt ativo no momento da resposta |
| `index_version` | string | Azure AI Search | Versão/snapshot do índice documental |
| `model_version` | string | Azure OpenAI | Versão do modelo GPT-4o utilizado |
| `feedback_type` | enum | Botão clicado | `useful`, `has_problem`, `incorrect` |
| `feedback_category` | enum | Formulário de detalhe | `F01`–`F07` ou `null` para ✅ sem detalhe |
| `feedback_comment` | string | Campo textual (opcional) | Comentário livre, máximo 500 caracteres |
| `chamado_id` | string | Integração Azure DevOps | ID do chamado ativo no momento do feedback (quando disponível) |

---

## Etapa 2 — Armazenamento

### 2.1 Responsabilidade e garantias

O armazenamento é responsabilidade do BC-06 (Rastreabilidade e Feedback). Os seguintes requisitos são obrigatórios:

- **Imutabilidade:** nenhum campo do registro de feedback pode ser alterado após a criação. Correções são feitas por adição de registro complementar com referência ao `feedback_id` original.
- **Retenção:** registros de feedback retidos por 24 meses. Registros com `feedback_category: F01` (incorreta) em temas de carga perigosa ou conflito documental retidos por 36 meses para fins de auditoria de conformidade.
- **Anonimização LGPD:** `query_text` e `answer_text` são anonimizados antes do armazenamento: dados pessoais de clientes (nome, CNPJ, número de contrato, endereço) substituídos por tokens genéricos (`[CLIENTE]`, `[CNPJ]`, `[CONTRATO]`). O `user_profile` armazena o perfil, não o identificador pessoal do atendente.
- **Acessibilidade:** logs acessíveis ao time de produto, QA, Tech Lead e Compliance. Não visíveis ao atendente.

### 2.2 Estrutura do registro

```json
{
  "feedback_id": "fb_20260605_001a2b3c",
  "session_id": "sess_20260605_d4e5f6",
  "turn_id": "turn_20260605_a7b8c9",
  "query_text": "Posso devolver carga perigosa classe 3?",
  "answer_text": "Não. Cargas perigosas classes 1 a 6 da ANTT...",
  "chunks_retrieved": [
    {
      "doc_name": "POL-001",
      "doc_version": "v3.1",
      "section": "Seção 3.2",
      "score": 0.89,
      "document_level": 1
    }
  ],
  "source_document_cited": {
    "doc_name": "POL-001",
    "doc_version": "v3.1",
    "section": "Seção 3.2"
  },
  "confidence_level": "Alta",
  "has_conflict": false,
  "conflict_notice_included": false,
  "is_absence_declaration": false,
  "timestamp": "2026-06-05T09:12:34Z",
  "user_profile": "atendente_senior",
  "prompt_version": "v1.2.0",
  "index_version": "idx_20260601",
  "model_version": "gpt-4o-2024-05-13",
  "feedback_type": "has_problem",
  "feedback_category": "F03",
  "feedback_comment": "Faltou informar o ramal 4500 da Gestão de Riscos",
  "chamado_id": "CHA-00492"
}
```

---

## Etapa 3 — Triagem

### 3.1 Triagem automática (imediata após recebimento)

O sistema classifica o feedback em três níveis de urgência com base nos metadados:

| Nível | Critério de classificação automática | SLA de ação humana |
|---|---|---|
| **Crítico** | `feedback_category: F01` (incorreta) E (`confidence_level: Alta` OU tema de carga perigosa detectado na query) | 4 horas úteis |
| **Alto** | `feedback_category: F01` sem critério crítico · `feedback_category: F04` (desatualizada) · qualquer feedback sobre tema com PROC ou POL como fonte | 24 horas úteis |
| **Médio** | `feedback_category: F02`, `F03`, `F05` | 72 horas úteis |
| **Baixo** | `feedback_category: F06`, `F07` · `feedback_type: useful` com comentário | 7 dias úteis |

### 3.2 Triagem humana

O responsável pelo nível de urgência correspondente recebe alerta automático (e-mail + Teams):

| Nível | Responsável da triagem |
|---|---|
| Crítico | Product Specialist + Tech Lead (alerta simultâneo) |
| Alto | Product Specialist |
| Médio | QA |
| Baixo | QA (processamento em lote semanal) |

A triagem humana valida se o nível automático está correto, revisa o comentário livre do atendente e verifica se já existe feedback similar aberto (`feedback_duplicado: true`). Se duplicado, vincula ao feedback original e encerra. Se novo, avança para classificação.

---

## Etapa 4 — Classificação da Causa Raiz

### 4.1 Categorias de causa raiz

| Causa raiz | ID | Descrição | Exemplo concreto no contexto NovaTech |
|---|---|---|---|
| Documento inexistente | `CR-01` | O tema da query não tem cobertura documental na base — gap genuíno | Pergunta sobre política de carga danificada; não existe POL ou PROC indexada |
| Documento incorreto ou desatualizado | `CR-02` | O documento existe e está indexado, mas seu conteúdo está errado ou substituído por versão mais recente não indexada | PROC-042 v1.0 indexada como vigente; multiplicadores incorretos retornados |
| Documento não indexado | `CR-03` | O documento existe nas fontes (SharePoint, Confluence) mas não foi ingerido no pipeline | PROC-088 referenciada pela POL-001 mas ausente do índice |
| Falha de chunking | `CR-04` | O documento está indexado mas foi dividido de forma que a informação relevante foi separada entre dois chunks ou truncada | Tabela de multiplicadores regionais cortada no meio; chunk retorna apenas 3 das 5 regiões |
| Falha de recuperação (threshold) | `CR-05` | O chunk correto existe no índice mas seu score ficou abaixo do threshold (0,72) e não foi recuperado | Query usou terminologia diferente ("taxa regional" em vez de "multiplicador regional"); score ficou em 0,68 |
| Falha de ranking | `CR-06` | Os chunks corretos foram recuperados mas o chunk vencedor foi o errado (versão mais antiga prevaleceu sobre a mais recente) | PROC-042 v1.0 com `last_confirmed_at` mais recente que a v2.0 por erro de metadado |
| Falha de interpretação | `CR-07` | O chunk correto foi recuperado e ranqueado, mas o LLM interpretou o conteúdo incorretamente ou extrapolou além do texto | Prazo de 7 dias (devolução padrão) aplicado incorretamente a carga perigosa (exceção na Seção 3.2) |
| Falha de prompt | `CR-08` | A instrução do system prompt foi insuficiente, ambígua ou ausente para o comportamento esperado | Prompt não instrui incluir SLA de incidente crítico em respostas de SLA por tier |
| Falha de regra de negócio | `CR-09` | Uma regra implementada em código está incorreta ou ausente | Validador de schema não bloqueia confiança Alta sem `source_document`; flag `is_dangerous_cargo` não detecta todos os termos de carga perigosa |
| Conflito documental | `CR-10` | Dois documentos divergem e o pipeline não detectou ou não sinalizou corretamente | PROC-042 v1.0 e v2.0 coexistentes; `has_conflict: false` quando deveria ser `true` |
| Problema de interface | `CR-11` | A informação estava correta no payload mas foi exibida de forma que causou confusão ou erro de leitura | `conflict_notice` colapsado junto ao bloco de rastreabilidade; atendente não viu o alerta |
| Pergunta fora do escopo | `CR-12` | A query envolve tema que não deve ser coberto pelo assistente e o sistema não declarou ausência ou tentou responder | Query sobre negociação individual de desconto; assistente gerou conteúdo baseado em FAQ item 45 |

### 4.2 Como determinar a causa raiz

O classificador (QA ou Product Specialist, conforme nível) segue este roteiro:

```
1. O chunk vencedor foi o documento correto para o tema?
   NÃO → verificar se o documento existe no índice
         SE existe: CR-05 (threshold) ou CR-06 (ranking)
         SE não existe: verificar se existe nas fontes externas
                        SE existe nas fontes: CR-03 (não indexado)
                        SE não existe em lugar nenhum: CR-01 (inexistente)
   SIM → continuar

2. O chunk vencedor foi recuperado com o conteúdo correto?
   NÃO → CR-04 (falha de chunking)
   SIM → continuar

3. A resposta diverge do conteúdo do chunk?
   SIM → CR-07 (falha de interpretação) ou CR-08 (falha de prompt — instrução ausente)
   NÃO → continuar

4. O chunk continha informação desatualizada?
   SIM → CR-02 (documento desatualizado)
   NÃO → continuar

5. O pipeline detectou o problema mas não agiu corretamente?
   (conflito não sinalizado, confiança não rebaixada, FAQ sem aviso)
   SIM → CR-09 (falha de regra de negócio) ou CR-10 (conflito documental)
   NÃO → continuar

6. A resposta estava correta mas o atendente não entendeu?
   SIM → CR-11 (problema de interface) ou treinamento do usuário
   NÃO → CR-12 (fora do escopo) ou reclassificar o feedback
```

---

## Etapa 5 — Definição da Ação

### 5.1 Tabela de causa raiz → ação → responsável

| Causa raiz | Exemplo NovaTech | Ação provável | Tipo de mudança | Responsável principal |
|---|---|---|---|---|
| `CR-01` Documento inexistente | Sem normativo formal sobre carga danificada | Criar normativo ou formalizar gap na base com declaração de ausência; se decisão editorial: acionar área responsável (Operações/Compliance) para publicar documento | Novo documento + atualização de lista de temas bloqueados | Operações ou Compliance (publicação) · Product Specialist (lista de bloqueados) |
| `CR-02` Documento incorreto ou desatualizado | PROC-042 v1.0 com multiplicadores antigos retornada como vigente | Atualizar metadado `status: "arquivado"` da versão antiga; confirmar publicação da versão nova nas fontes; reindexar | Ajuste de metadados + reindexação | Responsável de base da área (Comercial) · Tech Lead (metadado + reindexação) |
| `CR-03` Documento não indexado | PROC-088 referenciada mas ausente do índice | Localizar documento nas fontes externas; validar com área responsável; executar ingestão | Reindexação (ingestão de novo documento) | Responsável de base + Tech Lead |
| `CR-04` Falha de chunking | Tabela de multiplicadores cortada entre dois chunks | Revisar estratégia de chunking para o tipo de documento afetado (tabela, lista numerada, seção com subseções); ajustar parâmetros de divisão no pipeline de ingestão; reindexar documento | Alteração de chunking + reindexação | Tech Lead · Engenharia |
| `CR-05` Falha de recuperação | "taxa regional" não recupera chunk de "multiplicador regional" | Verificar se o problema é de embedding (adicionar sinônimos ao chunk via metadado) ou de threshold (recalibrar 0,72 para o tipo de query); testar no golden set antes de ajustar em produção | Ajuste de metadados (sinônimos) ou recalibração de threshold | Tech Lead · Engenharia |
| `CR-06` Falha de ranking | PROC-042 v1.0 vence PROC-042 v2.0 por metadado incorreto | Verificar `last_confirmed_at` dos dois documentos; corrigir metadado da versão errada; validar que a lógica de desempate usa `last_confirmed_at`, não `last_updated_at` | Ajuste de metadados | Tech Lead · Responsável de base |
| `CR-07` Falha de interpretação | Prazo de devolução padrão aplicado a carga perigosa | Adicionar instrução explícita ao system prompt com a exceção identificada; se recorrente para o mesmo tema, criar instrução de domínio específica | Ajuste de prompt | Product Specialist · Prompt Engineering |
| `CR-08` Falha de prompt | SLA crítico não incluído em resposta de SLA por tier | Adicionar regra de completude ao system prompt: "quando tier e SLA forem mencionados, incluir chamados gerais E incidentes críticos" | Ajuste de prompt | Product Specialist · Prompt Engineering |
| `CR-09` Falha de regra de negócio | Validador não bloqueia confiança Alta sem fonte | Corrigir função de validação de schema; adicionar teste de contrato ao CI; verificar se outros guardrails de código têm falha similar | Alteração de código | Tech Lead · Engenharia · QA |
| `CR-10` Conflito documental | `has_conflict: false` para PROC-042 v1.0 e v2.0 coexistentes | Verificar lógica de comparação de metadados em BC-02; ajustar critério de detecção de conflito; testar com os documentos conflitantes conhecidos | Alteração de código (BC-02) | Tech Lead · Engenharia |
| `CR-11` Problema de interface | Atendente não viu `conflict_notice` colapsado | Alterar comportamento da UI: alertas (`conflict_notice`, `informal_notice`, `stale_notice`) sempre visíveis, não colapsáveis | Mudança na interface | Engenharia · UX |
| `CR-12` Fora do escopo | Assistente gerou resposta para negociação de desconto individual | Adicionar tema à lista de temas bloqueados (`BLOCKED_TOPICS`); reforçar instrução no system prompt de declarar ausência para o domínio identificado; se recorrente, avaliar se é novo caso de uso legítimo | Alteração de código (lista de bloqueados) + ajuste de prompt | Product Specialist · Engenharia |

### 5.2 Quando o tipo de ação é treinamento do usuário

Algumas causas raiz não são problemas do produto — são lacunas de entendimento do atendente sobre como usar o assistente:

- Atendente marcou ❌ porque esperava uma resposta que está fora do escopo declarado (tema bloqueado por fase 1).
- Atendente marcou ⚠️ porque não entendeu o aviso de fonte informal e esperava uma resposta definitiva.
- Atendente marcou ❌ para resposta correta porque discordava de uma política da NovaTech, não de uma falha do assistente.

Nesses casos, a ação é treinamento do usuário — não alteração do produto. Registrar como `action_type: user_training`; não abrir trabalho de engenharia; notificar a Coordenação de Atendimento com os casos identificados para inclusão no próximo ciclo de treinamento.

---

## Etapa 6 — Implementação

### 6.1 Tipos de mudança e janela de implementação

| Tipo de mudança | Janela de implementação | Requer PR aprovado? | Deploy em produção requer aprovação humana? |
|---|---|---|---|
| Novo documento (ingestão) | Qualquer dia útil dentro do SLA de atualização (4h para Nível 1) | Não para ingestão; sim se requer alteração de código | Responsável de base confirma; Tech Lead executa |
| Correção documental / atualização de versão | Qualquer dia útil | Não para ingestão | Responsável de base confirma; Tech Lead executa |
| Reindexação isolada | Qualquer dia útil | Não | Tech Lead executa com monitoramento de M-04 |
| Ajuste de metadados | Qualquer dia útil | Sim (PR no repositório de configuração) | Tech Lead aprova |
| Alteração de chunking | Janela de manutenção (terças ou quintas, 08h–10h) | Sim | Tech Lead + QA aprovam; reindexação completa do documento afetado |
| Ajuste de prompt | Qualquer dia útil | Sim (PR no repositório) | Product Specialist + Tech Lead aprovam |
| Alteração de código (pipeline, validadores, BC-02, BC-03) | Janela de manutenção | Sim | Tech Lead + QA aprovam; ver Etapa 8 |
| Mudança na interface | Sprint regular | Sim | Tech Lead + Product Specialist aprovam |
| Recalibração de threshold | Janela de manutenção | Sim (PR com documentação da recalibração) | Tech Lead + Product Specialist aprovam; executar golden set antes e depois |

### 6.2 Criação de caso de regressão derivado do feedback

Para qualquer feedback com causa raiz `CR-01` a `CR-10` que gere alteração no produto, o QA cria um novo caso de teste de regressão derivado do feedback:

```typescript
// Exemplo: feedback F03 / CR-08 — SLA crítico omitido
{
  regression_id: "REG-042",
  origin_feedback_id: "fb_20260605_001a2b3c",
  query: "Meu cliente é Silver. Qual o prazo de resolução?",
  assertions: [
    (r) => r.answer.toLowerCase().includes('incidente crítico') || r.answer.includes('crítico'),
    (r) => r.answer.includes('8'),   // SLA crítico Silver = 8h
    (r) => r.answer.includes('48'),  // SLA geral Silver = 48h
    (r) => r.answer.includes('úteis')
  ],
  created_from: "feedback_id:fb_20260605_001a2b3c",
  priority: "high"
}
```

O caso é adicionado à suite de regressão (`tests/regression/`) e deve passar antes da aprovação do deploy que corrige o problema.

---

## Etapa 7 — Testes

### 7.1 Sequência obrigatória antes de qualquer deploy

```
[1] Executar o caso de regressão derivado do feedback atual
    → Verificar que o caso FALHA com o código atual (confirma que o bug existe)

[2] Implementar a correção

[3] Executar o caso de regressão derivado do feedback atual
    → Verificar que o caso PASSA (confirma que a correção resolve o problema)

[4] Executar a suite completa de regressão (todos os casos existentes)
    → Verificar que NENHUM caso regredir

[5] Para alterações de prompt: executar golden set de groundedness (M-02)
    com amostra de 10 respostas para o tema afetado
    → Verificar que groundedness não degradou

[6] Para alterações de threshold ou chunking: executar golden set de
    precisão de recuperação (M-04) completo (50 queries)
    → Verificar que precisão manteve ≥ 90%

[7] Para alterações de código: executar testes unitários e de integração
    → Cobertura mínima: 80% das linhas alteradas
```

### 7.2 Testes em staging com atendentes-piloto

Para alterações classificadas como de alto risco (alteração de threshold, reescrita de instrução de domínio no prompt, alteração na lógica de conflito em BC-02), realizar teste de 24 horas em staging com os 5 atendentes-piloto antes do deploy em produção. Coletar feedback dos piloto e verificar M-07 (feedback positivo/negativo) no ambiente de staging.

---

## Etapa 8 — Aprovação Humana (Human-in-the-Loop)

### 8.1 Níveis de aprovação por tipo de mudança e risco

| Nível de risco | Critério | Aprovadores obrigatórios | Aprovação assíncrona aceita? |
|---|---|---|---|
| **Crítico** | Alteração de código que modifica comportamento de guardrails (P-06, P-07, P-08, P-09, P-10); qualquer mudança que afete respostas sobre carga perigosa | Product Specialist + Tech Lead + Compliance | Não — reunião síncrona de 30 min |
| **Alto** | Ajuste de prompt que altera comportamento para tema coberto por POL ou PROC; alteração de threshold; mudança no schema de resposta | Product Specialist + Tech Lead | Não — revisão síncrona de PR |
| **Médio** | Ingestão de novo documento de Nível 1; correção de metadados; ajuste de chunking de documento específico | Tech Lead (execução) + Responsável de base da área (validação do conteúdo) | Sim — aprovação via PR em até 24h |
| **Baixo** | Ingestão de documento de Nível 3 (Confluence); ajuste de UI menor; treinamento de usuário | Tech Lead | Sim — aprovação via PR em até 48h |

### 8.2 Checklist de aprovação para risco alto e crítico

Antes de aprovar qualquer deploy de nível Alto ou Crítico, os aprovadores verificam:

- [ ] O caso de regressão derivado do feedback está passando
- [ ] Nenhum caso da suite de regressão regrediu
- [ ] O golden set foi executado e a métrica afetada está dentro da meta
- [ ] O rollback está definido e testado (qual versão reverter, qual comando executar)
- [ ] O alerta de monitoramento pós-implantação está configurado (quais métricas monitorar, por quantos dias)
- [ ] A comunicação ao atendente que originou o feedback está preparada

---

## Etapa 9 — Implantação

### 9.1 Procedimento de deploy

Todo deploy que originou de um feedback registrado deve:

1. Ser executado fora do horário de pico (evitar 09h–12h e 14h–17h, que concentram a maioria dos chamados).
2. Registrar no log de auditoria do BC-06: `deploy_id`, `feedback_ids_resolvidos[]`, `type_of_change`, `version_before`, `version_after`, `deployed_by`, `timestamp`.
3. Ter rollback definido: se M-07 (taxa de ❌) subir > 3 pontos percentuais em 2 horas após o deploy, ou se M-08 (TTFT p95) ultrapassar 5 segundos, reverter imediatamente para a versão anterior sem esperar aprovação.

### 9.2 Rastreabilidade do deploy

O registro de deploy vincula cada alteração ao(s) feedback(s) que a motivou:

```json
{
  "deploy_id": "dep_20260607_v1.2.1",
  "feedback_ids_resolved": ["fb_20260605_001a2b3c", "fb_20260604_009d8e7f"],
  "type_of_change": "prompt_adjustment",
  "change_description": "Adicionada instrução de completude de SLA: incluir incidente crítico em toda resposta de prazo por tier",
  "version_before": { "prompt_version": "v1.2.0", "index_version": "idx_20260601" },
  "version_after": { "prompt_version": "v1.2.1", "index_version": "idx_20260601" },
  "deployed_by": "tech_lead_01",
  "approved_by": ["ps_01", "tech_lead_01"],
  "timestamp": "2026-06-07T07:30:00Z",
  "rollback_instructions": "git revert prompt/system_prompt.txt to v1.2.0; redeploy"
}
```

---

## Etapa 10 — Comunicação ao Usuário

### 10.1 Notificação ao atendente que originou o feedback

O atendente que registrou o feedback recebe notificação automática via Teams quando o ciclo for encerrado. A notificação é entregue por BC-06 via BC-05 e segue os modelos abaixo:

**Quando a melhoria foi implementada:**
> "Seu feedback de [data] sobre [tema resumido] foi analisado e resolvido. A melhoria está disponível a partir de agora. Obrigado por contribuir com a qualidade do assistente."

**Quando o problema foi identificado como limitação da fase 1 (fora do escopo):**
> "Seu feedback de [data] foi analisado. O tema [tema resumido] está fora do escopo do assistente nesta fase. Esta limitação foi registrada para avaliação em próximas versões."

**Quando o problema foi identificado como treinamento do usuário:**
> "Seu feedback de [data] foi analisado. A resposta estava baseada na documentação oficial disponível. Entre em contato com seu supervisor para orientações sobre [tema]."

**Quando não foi possível resolver (gap documental sem normativo publicado):**
> "Seu feedback de [data] foi analisado. O tema [tema resumido] ainda não tem documentação formal disponível na base. A situação foi registrada e será tratada quando o normativo for publicado."

### 10.2 Comunicação coletiva para melhorias relevantes

Quando uma melhoria resolve um padrão recorrente (mais de 5 feedbacks similares), o Product Specialist envia comunicado ao grupo de atendentes via Teams com:
- O que mudou
- Por que mudou (resumo do problema identificado)
- Como a resposta ficou (exemplo antes/depois, quando aplicável)

---

## Etapa 11 — Monitoramento Pós-Implantação

### 11.1 Janela de monitoramento intensivo

Todo deploy derivado de feedback entra em janela de monitoramento intensivo de **14 dias corridos** a partir da data de implantação.

### 11.2 Métricas monitoradas por tipo de mudança

| Tipo de mudança | Métricas monitoradas | Frequência durante janela | Critério de alerta |
|---|---|---|---|
| Ajuste de prompt | M-02 (groundedness), M-03 (alucinação), M-07 (feedback) | Diária | Degradação > 5% na groundedness ou aumento de ❌ > 3pp |
| Alteração de threshold / retrieval | M-04 (precisão), M-11 (ausência falsa), M-05 (baixa confiança) | Diária | M-04 < 85% ou M-11 > 5% |
| Ingestão de novo documento | M-04 (precisão), M-07 (feedback) para queries do tema novo | Diária | Qualquer feedback ❌ sobre o tema recém-indexado |
| Alteração de código (validadores, BC-02, BC-03) | M-09 (violação de guardrails), M-07 (feedback), M-08 (tempo) | Contínua por 48h, depois diária | Qualquer violação automática de guardrail |
| Mudança na interface | M-07 (feedback), M-12 (aceitação) | Diária | ✅ < 65% ou M-12 < 55% |

### 11.3 Encerramento da janela de monitoramento

Ao final dos 14 dias, o Product Specialist registra no log de feedback:

- `monitoring_closed_at`: timestamp do encerramento
- `outcome`: `resolved` (melhoria efetiva confirmada) · `partial` (melhoria parcial, abrir novo item) · `regressed` (problema piorou, escalar imediatamente)
- `metric_deltas`: variação das métricas monitoradas entre o período pré e pós-deploy
- `new_feedback_on_same_topic`: número de feedbacks sobre o mesmo tema após o deploy (esperado: zero para `resolved`)

O feedback original só é marcado como `status: "closed"` quando `outcome: "resolved"` for registrado. Até lá, permanece como `status: "monitoring"`.

---

## Resumo: Tipos de Mudança × Camada × Responsável

| Tipo de mudança | Camada | Responsável principal | SLA para implementação |
|---|---|---|---|
| Novo documento | Governança documental + Pipeline (ingestão) | Responsável de base + Tech Lead | 4h (Nível 1) · 48h (Nível 3) |
| Correção documental | Governança documental + Pipeline (reindexação) | Responsável de base + Tech Lead | 4h (Nível 1) |
| Reindexação isolada | Pipeline | Tech Lead | 4h |
| Ajuste de metadados | Pipeline + Governança documental | Tech Lead + Responsável de base | 24h |
| Alteração de chunking | Pipeline (ingestão) | Tech Lead + Engenharia | 72h (inclui reindexação e testes) |
| Ajuste de prompt | Prompt | Product Specialist + Prompt Engineering | 48h |
| Alteração de código | Código | Tech Lead + Engenharia | 72h (inclui testes e PR) |
| Mudança na interface | Interface | Engenharia + UX | Sprint regular |
| Treinamento do usuário | Governança (processo) | Coordenação de Atendimento | Próximo ciclo de treinamento |

---

*Documento elaborado com base nas decisões arquiteturais (ADR-0001 a ADR-0004), nos guardrails formalizados (guardrails-novatech.md), nas métricas de qualidade (novatech-harness-metricas.md) e na revisão crítica de respostas pré-go-live (novatech-revisao-critica-final.md). Revisão esperada após 60 dias de produção com base nos dados reais coletados.*
