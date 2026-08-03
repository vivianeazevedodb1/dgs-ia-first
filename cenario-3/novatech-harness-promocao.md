# Harness de Produto — NovaTech Assistant
## Fluxo de Promoção de Alteração: da Necessidade ao Monitoramento

**Versão:** 1.0  
**Data:** 05/06/2026  
**Elaborado por:** Product Specialist Sênior  
**Destinatários:** Product · Tech Lead · QA · Engenharia · Delivery Manager · Operações · Compliance · Comercial

> Este documento descreve o fluxo completo pelo qual qualquer alteração no assistente — originada de feedback, incidente ou necessidade de produto — percorre desde a identificação até a consolidação em produção ou rollback. Cada etapa tem entrada, atividade, responsável, saída e critério de conclusão verificável.

---

## Visão Geral do Fluxo

```
┌─────────────────────────────────────────────────────────────────────┐
│  ETAPA 1   Feedback ou necessidade                                  │
│            ↓                                                        │
│  ETAPA 2   Triagem                                                  │
│            ↓                                                        │
│  ETAPA 3   Causa raiz                                               │
│            ↓                                                        │
│  ETAPA 4   Proposta de melhoria                                     │
│            ↓                                                        │
│  ETAPA 5   Implementação                                            │
│            ↓                                                        │
│  ETAPA 6   Testes automatizados                                     │
│            ↓                                                        │
│  ETAPA 7   Avaliação semântica                                      │
│            ↓                                                        │
│  ETAPA 8   Revisão humana                                           │
│            ↓                                                        │
│  ETAPA 9   Aprovação                                                │
│            ↓                                                        │
│  ETAPA 10  Implantação controlada                                   │
│            ↓                                                        │
│  ETAPA 11  Monitoramento                                            │
│            ↓                                                        │
│  ETAPA 12  Rollback ou consolidação                                 │
└─────────────────────────────────────────────────────────────────────┘
```

**Regra de progressão:** cada etapa só inicia quando o critério de conclusão da etapa anterior está satisfeito. Nenhuma etapa pode ser pulada. Em situações de hotfix crítico, as etapas 6, 7 e 8 podem ocorrer em paralelo com a 5, mas nenhuma delas pode ser suprimida.

---

## Etapa 1 — Feedback ou Necessidade

### Entrada

Uma das seguintes fontes:

- **Feedback explícito do atendente:** registro com `feedback_category` ⚠️ ou ❌ via interface do Teams, armazenado em BC-06 com os 22 campos de contexto definidos no harness de feedback.
- **Alerta de métrica:** qualquer métrica do harness (M-01 a M-12) ultrapassou o limite de alerta definido no harness de métricas.
- **Incidente de produção:** resposta incorreta identificada por supervisor, Compliance ou Operações fora do ciclo de feedback formal.
- **Necessidade de produto:** nova versão de documento publicada pelo Gestor Documental, expansão de domínio solicitada, atualização de guardrail identificada pelo Product Specialist.
- **Resultado de avaliação periódica:** painel de gaps ou revisão trimestral gerou item de ação.

### Atividade

O Product Specialist (para feedbacks de produto e necessidades) ou o Tech Lead (para alertas técnicos) registra o item no backlog de melhorias com os seguintes campos obrigatórios:

- `item_id` — identificador único
- `origin_type` — `feedback` | `metric_alert` | `incident` | `product_need` | `periodic_review`
- `origin_id` — `feedback_id` do BC-06, identificador do alerta, ou referência ao incidente
- `description` — descrição em linguagem de negócio do problema ou necessidade
- `initial_severity` — `critical` | `high` | `medium` | `low` (estimativa inicial, revisada na triagem)
- `registered_by` — papel e nome de quem registrou
- `registered_at` — timestamp

### Responsável

- Feedback de atendente → **Product Specialist** registra
- Alerta de métrica → **Tech Lead** registra
- Incidente → **Product Specialist** registra após notificação do supervisor ou Compliance
- Necessidade de produto / revisão periódica → **Product Specialist** registra

### Saída

Item de backlog com campos obrigatórios preenchidos, linkado ao `feedback_id` ou ao identificador da origem. O item tem status `registrado`.

### Critério de conclusão

O item está registrado no backlog com todos os campos obrigatórios preenchidos e `origin_id` rastreável ao evento que o gerou. O Product Specialist ou Tech Lead confirmou o recebimento com timestamp.

---

## Etapa 2 — Triagem

### Entrada

Item de backlog com status `registrado`, originado da Etapa 1.

### Atividade

O responsável pela triagem executa as seguintes verificações, nesta ordem:

**1. Verificação de duplicata:** consultar o BC-06 por feedbacks com `origin_type`, `domain` e `feedback_category` similares nos últimos 30 dias. Se existir item aberto sobre o mesmo problema, vincular (`linked_to: item_id`) e encerrar o novo item como duplicata. Avançar apenas com o item original.

**2. Classificação de nível de urgência** com base nos critérios:

| Nível | Critério |
|---|---|
| **Crítico** | `feedback_category: F01` + (`confidence_level: Alta` na resposta original OU tema de carga perigosa) · Qualquer violação dos guardrails GR-03, GR-10 · Alerta de métrica M-03 (alucinação) ou M-09 (guardrail) |
| **Alto** | `feedback_category: F01` sem critério crítico · `feedback_category: F04` (desatualizada) · Alerta de M-07 (❌ > 8%) · Necessidade de novo documento normativo |
| **Médio** | `feedback_category: F02, F03, F05` · Alerta de M-05 (baixa confiança > 20%) · Necessidade de ajuste de metadados ou chunking |
| **Baixo** | `feedback_category: F06, F07` · Melhoria de interface · Treinamento de usuário · Alerta de M-08 somente |

**3. Definição de SLA de ação humana:** Crítico → 4 horas úteis · Alto → 24 horas úteis · Médio → 72 horas úteis · Baixo → 7 dias úteis.

**4. Atribuição de responsável:** conforme o nível — Crítico: Product Specialist + Tech Lead simultaneamente · Alto: Product Specialist · Médio: QA · Baixo: QA em lote semanal.

### Responsável

- Nível Crítico: **Product Specialist** (triagem imediata)
- Nível Alto: **Product Specialist**
- Nível Médio e Baixo: **QA**

### Saída

Item com status `em_triagem` → `triagem_concluida`, com campos adicionados:

- `urgency_level` — nível definido
- `is_duplicate` — boolean
- `linked_to` — item_id original (se duplicata)
- `assigned_to` — papel responsável pela próxima etapa
- `sla_deadline` — timestamp limite para conclusão da causa raiz

### Critério de conclusão

Item classificado com `urgency_level` definido, responsável atribuído, SLA registrado. Se duplicata: item encerrado e vinculado. Se não duplicata: item avança para Etapa 3 dentro do SLA definido.

---

## Etapa 3 — Causa Raiz

### Entrada

Item com status `triagem_concluida`, com `urgency_level` e `assigned_to` preenchidos.

### Atividade

O responsável executa o roteiro de diagnóstico da causa raiz (árvore de decisão definida no harness de feedback, Seção 4.2), seguindo estas perguntas em sequência:

1. O chunk vencedor foi o documento correto? → Se não: `CR-03`, `CR-05` ou `CR-06`
2. O chunk tinha o conteúdo correto? → Se não: `CR-04`
3. A resposta diverge do chunk? → Se sim: `CR-07` ou `CR-08`
4. O chunk tinha informação desatualizada? → Se sim: `CR-02`
5. O pipeline detectou o problema mas não agiu? → Se sim: `CR-09` ou `CR-10`
6. A resposta estava correta mas o atendente não entendeu? → Se sim: `CR-11` ou treinamento
7. O tema estava fora do escopo? → `CR-12`
8. Nenhuma das anteriores: `CR-01` (documento inexistente)

Para cada causa raiz identificada, consultar a tabela de ação provável (harness de feedback, Seção 5.1) para determinar o tipo de mudança correspondente.

**Investigação de artefatos disponíveis:**

- `chunks_retrieved` do registro de feedback: verificar quais chunks foram recuperados e seus scores.
- `source_document_cited` vs. chunk esperado: verificar se a seção citada corresponde ao conteúdo da resposta.
- `index_version` e `prompt_version` do registro: verificar se as versões correspondem às atuais ou a uma versão com problema conhecido.
- Log do Azure AI Search: verificar o score do chunk esperado (estava abaixo de 0,72?).

### Responsável

- Nível Crítico: **Product Specialist** + **Tech Lead** (em conjunto)
- Nível Alto: **Product Specialist** (com suporte do Tech Lead para investigação de retrieval)
- Nível Médio: **QA** (com suporte do Tech Lead se necessário)

### Saída

Item com `root_cause_id` preenchido (um dos `CR-01` a `CR-12`), `change_type` identificado (MA-01 a MA-13), e `action_type` definido dentre: `novo_documento`, `correcao_documental`, `reindexacao`, `ajuste_metadados`, `alteracao_chunking`, `ajuste_prompt`, `alteracao_codigo`, `mudanca_interface`, `treinamento_usuario`. Status: `causa_identificada`.

### Critério de conclusão

`root_cause_id` atribuído com justificativa escrita referenciando os artefatos de diagnóstico (log de retrieval, contexto do feedback, chunk recuperado vs. esperado). `change_type` mapeado para o tipo de mudança correspondente da matriz de aprovação. O responsável da próxima etapa está identificado.

---

## Etapa 4 — Proposta de Melhoria

### Entrada

Item com status `causa_identificada`, com `root_cause_id` e `change_type` preenchidos.

### Atividade

O Product Specialist (para mudanças de produto, prompt, guardrail, domínio) ou o Tech Lead (para mudanças de código, pipeline, chunking, threshold) elabora a proposta de melhoria contendo:

**1. Descrição da mudança:**
- Estado atual (o que o sistema faz hoje)
- Estado proposto (o que o sistema deve fazer após a mudança)
- Diferença exata: para prompt — diff do texto; para código — pseudocódigo ou descrição da função afetada; para metadado — campo e valor atual vs. valor proposto

**2. Casos do golden dataset afetados:**
- Casos que atualmente falham e devem passar após a mudança
- Casos que atualmente passam e podem ser impactados (risco de regressão)

**3. Novo caso de regressão derivado do feedback:**
- `regression_id` novo
- `query`, `assertions`, `origin_feedback_id`
- O caso deve FALHAR com o sistema atual (confirmação de que o bug existe)

**4. Tipo de mudança e aprovações necessárias** conforme a matriz de aprovação (MA-01 a MA-13):
- Aprovadores obrigatórios
- Possíveis bloqueadores
- Evidências obrigatórias para aprovação

**5. Janela de implementação** proposta (conforme o tipo de mudança e nível de urgência).

### Responsável

- Mudanças de produto (prompt, guardrail, domínio, fallback): **Product Specialist**
- Mudanças técnicas (código, pipeline, chunking, threshold, metadados): **Tech Lead**
- Mudanças documentais (vigência, novo documento, conflito): **Gestor Documental** da área responsável

### Saída

Documento de proposta de melhoria com os 5 campos acima preenchidos. Novo caso de regressão criado em `tests/regression/` com status `pendente` (ainda não passou). Status do item: `proposta_elaborada`.

### Critério de conclusão

Proposta revisada e aceita pelo par do responsável (Product Specialist revisa proposta do Tech Lead e vice-versa). Novo caso de regressão confirmado como FALHO com o sistema atual (prova de que o problema existe e o caso é válido). Janela de implementação confirmada com o Delivery Manager.

---

## Etapa 5 — Implementação

### Entrada

Proposta de melhoria com status `proposta_elaborada`, revisada e aceita. Novo caso de regressão confirmado como falho.

### Atividade

A implementação segue o `action_type` definido na Etapa 3:

**Ajuste de prompt:** Product Specialist ou Prompt Engineering edita `/prompts/system_prompt.txt` (ou equivalente no repositório). Commit com mensagem: `fix(prompt): [descrição] | resolves feedback_id:[id] | change_type:MA-07`.

**Alteração de código:** Engenharia abre branch `fix/[item_id]`, implementa a alteração em BC-02, BC-03 ou validadores, abre PR com referência ao `item_id` e ao novo caso de regressão.

**Ajuste de metadados:** Tech Lead executa script de atualização de metadado no Azure AI Search com os campos `doc_name`, `field`, `old_value`, `new_value` registrados no log de auditoria do BC-06.

**Reindexação:** Tech Lead executa o pipeline de ingestão para o documento afetado com a nova versão ou metadados corrigidos. Registra `deploy_id` de ingestão no log.

**Alteração de chunking:** Engenharia ajusta os parâmetros de chunking no pipeline de ingestão, reindexar apenas o documento afetado. Registrar diferença de número de chunks antes e depois.

**Correção documental / novo documento:** Gestor Documental publica o documento na fonte oficial; Tech Lead executa a ingestão após confirmação.

**Mudança de interface:** Engenharia implementa na camada BC-05 com PR e referência ao item.

Em todos os casos, o implementador registra no item:
- `implementation_started_at`
- `implementation_completed_at`
- `artifact_changed` — arquivo, script ou componente alterado
- `version_before` e `version_after` (prompt_version, index_version, ou código)

### Responsável

- Prompt: **Product Specialist** (com revisão do Tech Lead)
- Código / Pipeline: **Engenharia** (sob supervisão do Tech Lead)
- Metadados / Ingestão: **Tech Lead**
- Documento: **Gestor Documental** (publicação) + **Tech Lead** (ingestão)
- Interface: **Engenharia** (com revisão do UX quando aplicável)

### Saída

Artefato implementado em ambiente de staging (não em produção). PR aberto (quando aplicável) com referência ao `item_id` e ao novo caso de regressão. Status: `implementado_em_staging`.

### Critério de conclusão

A implementação está em staging e acessível para testes. O PR está aberto (quando aplicável) e revisado pelo par. O ambiente de staging reflete exatamente o que será promovido a produção — sem diferenças adicionais. O Tech Lead confirmou que nenhuma outra mudança não relacionada foi incluída na mesma branch.

---

## Etapa 6 — Testes Automatizados

### Entrada

Implementação em staging com status `implementado_em_staging`. Golden dataset disponível na versão atual (`/tests/golden/golden-dataset.yaml`). Suite de regressão com o novo caso adicionado.

### Atividade

O QA executa a sequência obrigatória de testes (conforme harness de validação, Seção 3.1):

**Passo 1 — Novo caso de regressão (validação do problema):**
Confirmar que o novo caso de regressão PASSA com a implementação candidata. Se falhar, a implementação não resolveu o problema — retornar à Etapa 5.

**Passo 2 — Suite completa de regressão:**
Executar todos os casos existentes em `tests/regression/`. Nenhum caso previamente aprovado pode regredir.

**Passo 3 — Testes determinísticos (TD-01 a TD-15):**
Executar todos os 15 testes determinísticos sobre o golden dataset com a implementação candidata. Resultado binário por teste.

**Passo 4 — Verificação de latência:**
Executar 20 queries do golden dataset e calcular TTFT p95 e resposta completa p95. Registrar valores.

**Passo 5 — Verificação de schema:**
Confirmar que 100% das respostas do golden dataset retornam schema válido (TD-02).

O QA preenche a coluna "Resultado candidato" da tabela de avaliação por dimensão (harness de validação, Seção 5) para todas as dimensões cobertas por testes determinísticos.

### Responsável

**QA** (execução e registro de resultados). Tech Lead disponível para suporte em caso de falha técnica na execução dos testes.

### Saída

Relatório de testes automatizados com:
- Resultado por teste determinístico (TD-01 a TD-15): Passou / Falhou
- Resultado da suite de regressão: número de casos passados / total
- Resultado do novo caso de regressão: Passou / Falhou
- Valores de latência: TTFT p95 e resposta completa p95
- Colunas "Resultado candidato" preenchidas na tabela de avaliação para as dimensões determinísticas

Status: `testes_automatizados_concluidos` (se todos passaram) ou `testes_automatizados_falharam` (retorno à Etapa 5).

### Critério de conclusão

**Todos** os seguintes são verdadeiros:
- Novo caso de regressão: PASSOU
- Suite de regressão completa: 100% dos casos previamente aprovados continuam passando
- Testes determinísticos TD-01 a TD-15: 100% de aprovação
- TTFT p95 ≤ 5s (limite de rejeição absoluto) e ≤ 3s (meta — se entre 3s e 5s, registrar como ressalva)
- Nenhuma das regras bloqueantes B-01, B-03, B-10 ativadas (citação, schema, tier inválido)

Se qualquer critério não for satisfeito: status `testes_automatizados_falharam`, item retorna à Etapa 5 com relatório de falhas anexado.

---

## Etapa 7 — Avaliação Semântica

### Entrada

Relatório de testes automatizados com status `testes_automatizados_concluidos`. Implementação em staging.

### Atividade

O QA executa os testes semânticos (TS-01 a TS-08) conforme harness de validação, Seção 3.2:

**TS-01 Groundedness:** apresentar os 15 pares (chunk + resposta) do golden dataset a um LLM como juiz com o prompt: "Todas as afirmações factuais desta resposta estão sustentadas pelo trecho de documento a seguir? Classifique cada afirmação como: aderente / extrapolada / sem sustentação." Calcular taxa de aderência.

**TS-02 Completude:** para cada caso do golden dataset, verificar presença de todos os `accepted_evidence` definidos. Calcular percentual de presença.

**TS-03 Ausência de evidência proibida:** para cada caso, verificar ausência de todos os `rejected_evidence`. Registrar qualquer ocorrência.

**TS-04 Ausência de extrapolação:** usar output do TS-01 para calcular percentual de afirmações "extrapoladas" ou "sem sustentação".

**TS-05 Conflitos corretamente tratados:** verificar GD-006 e GD-014 especificamente. `source_document` = chunk vencedor esperado E `conflict_notice` presente.

**TS-06 Adequação do fallback:** verificar estrutura das declarações de ausência nos casos GD-005, GD-009, GD-010.

**TS-07 Confiança correta:** verificar correspondência de `confidence_level` para todos os 15 casos.

**TS-08 Escalação quando necessária:** verificar presença de linguagem de escalação nos casos com `requires_escalation: true`.

O QA preenche as colunas restantes da tabela de avaliação por dimensão (TS-01 a TS-08).

### Responsável

**QA** (execução). Product Specialist revisa os resultados e valida se a análise semântica está correta antes de avançar.

### Saída

Tabela de avaliação por dimensão com colunas "Resultado candidato" preenchidas para TS-01 a TS-08. Status: `avaliacao_semantica_concluida` ou `avaliacao_semantica_falharam`.

### Critério de conclusão

**Todos** os seguintes são verdadeiros:
- TS-01 Groundedness ≥ 95% (ou queda ≤ 5pp em relação ao baseline)
- TS-02 Completude ≥ 90% dos `accepted_evidence` presentes
- TS-03 Zero ocorrências de `rejected_evidence` em casos `critical` e `high`
- TS-04 ≤ 5% de afirmações extrapoladas
- TS-05 100% nos casos GD-006 e GD-014
- TS-06 100% dos campos da declaração de ausência presentes nos casos GD-005, GD-009, GD-010
- TS-07 100% de correspondência de confiança
- TS-08 100% de escalação onde `requires_escalation: true`
- Nenhuma das regras bloqueantes B-01 a B-10 ativadas

Se qualquer critério não for satisfeito: `avaliacao_semantica_falharam`, item retorna à Etapa 5.

---

## Etapa 8 — Revisão Humana

### Entrada

Tabela de avaliação por dimensão com testes determinísticos e semânticos aprovados. Status `avaliacao_semantica_concluida`.

### Atividade

A revisão humana é executada pelo Product Specialist e, para casos críticos, pelo Compliance. Cobre as dimensões TH-01 a TH-06 do harness de validação, Seção 3.3:

**TH-01 Clareza:** o revisor lê cada resposta do golden dataset e atribui nota de 1 a 5. Meta: média ≥ 4.

**TH-02 Utilidade operacional:** o revisor avalia se o atendente conseguiria usar a resposta diretamente no chamado. Meta: média ≥ 4; nenhuma resposta ≤ 2 nos casos `critical`.

**TH-03 Interpretação contextual:** o revisor verifica se a resposta considerou o contexto implícito da pergunta (ex: "10 dias" implica prazo expirado em GD-011). Meta: média ≥ 3,5.

**TH-04 Riscos de comunicação:** o revisor identifica se qualquer resposta poderia ser mal interpretada de forma prejudicial. Meta: 0 riscos identificados em casos `critical`.

**TH-05 Alta criticidade — carga perigosa:** o Compliance revisa os casos GD-002, GD-007, GD-012, GD-013 e emite parecer Aprovado/Reprovado para cada um. Meta: 4/4 aprovados.

**TH-06 Ausência sem alucinação:** o Product Specialist revisa os casos GD-005 e GD-009. Confirmar que nenhum contém conteúdo factual inventado. Meta: 2/2 aprovados.

**Escopo da revisão humana por nível de mudança:**

| Nível | Casos obrigatórios | Opcionais |
|---|---|---|
| Crítico | Todos os 15 casos + TH-04, TH-05, TH-06 | — |
| Alto | Casos do domínio afetado + casos `critical` + TH-05, TH-06 | Outros casos |
| Médio | Casos do domínio afetado + TH-06 | Casos `critical` |
| Baixo | Casos do domínio afetado | — |

### Responsável

- TH-01, TH-02, TH-03, TH-04, TH-06: **Product Specialist**
- TH-05 (carga perigosa): **Compliance** (obrigatório, não substituível)
- Revisão adicional de contexto operacional: **Operações** (para mudanças que afetam procedimentos de atendimento)

### Saída

Parecer de revisão humana com:
- Notas TH-01 a TH-03 por caso avaliado
- Registro de riscos TH-04 (lista vazia = aprovado)
- Parecer TH-05 por caso: Aprovado/Reprovado + justificativa (assinado por Compliance)
- Parecer TH-06: Aprovado/Reprovado + justificativa
- Tabela de avaliação por dimensão completamente preenchida

Status: `revisao_humana_aprovada` ou `revisao_humana_reprovada`.

### Critério de conclusão

**Todos** os seguintes são verdadeiros:
- TH-01 média ≥ 4 e TH-02 média ≥ 4 sem nenhuma nota ≤ 2 em casos `critical`
- TH-03 média ≥ 3,5
- TH-04 zero riscos identificados em casos `critical`
- TH-05 4/4 aprovados pelo Compliance (casos de carga perigosa)
- TH-06 2/2 aprovados pelo Product Specialist

Se qualquer critério não for satisfeito: `revisao_humana_reprovada`, item retorna à Etapa 5 (mudanças de conteúdo) ou Etapa 4 (mudanças de escopo ou guardrail).

---

## Etapa 9 — Aprovação

### Entrada

Tabela de avaliação por dimensão completamente preenchida. Parecer de revisão humana aprovado. Status `revisao_humana_aprovada`.

### Atividade

O Product Specialist convoca os aprovadores obrigatórios conforme a matriz de aprovação (MA-01 a MA-13) para a reunião de aprovação (síncrona para Crítico; assíncrona via PR para Alto e abaixo).

**Checklist de aprovação** verificado por cada aprovador antes de assinar:

- [ ] O caso de regressão derivado do feedback está passando (evidência: relatório da Etapa 6)
- [ ] Nenhum caso previamente aprovado regrediu (evidência: suite de regressão da Etapa 6)
- [ ] O golden set foi executado e nenhuma regra bloqueante B-01 a B-10 foi ativada (evidência: tabela de avaliação completa)
- [ ] A revisão humana está aprovada com pareceres assinados (evidência: saída da Etapa 8)
- [ ] O rollback está definido: versão anterior identificada, comando de reversão documentado, testado em staging
- [ ] A janela de deploy está confirmada (fora do pico 09h–12h e 14h–17h)
- [ ] O alerta de monitoramento pós-implantação está configurado (métricas definidas, thresholds de rollback automático configurados)
- [ ] A comunicação ao atendente está preparada (para mudanças Alto e Crítico)

Cada aprovador registra: `approved_by`, `approved_at`, `approval_comment` (obrigatório para ressalvas).

**Exercício do direito de bloqueio:** qualquer aprovador que identifique problema pode registrar `blocked_by`, `blocked_at`, `block_reason`. O bloqueio suspende o item por até 5 dias úteis para resolução. Após 5 dias sem resolução, o Delivery Manager media a decisão final.

### Responsável

- **Product Specialist:** convoca a reunião, consolida os pareceres, registra a decisão final
- **Aprovadores obrigatórios por tipo de mudança:** conforme MA-01 a MA-13 do harness de aprovação
- **Delivery Manager:** media conflitos de aprovação, não aprova conteúdo

### Saída

Decisão de aprovação registrada no item:

- `decision` — `approved` | `approved_with_caveats` | `review_requested` | `rejected`
- `approved_by` — lista de aprovadores com timestamp
- `caveats` — lista de ressalvas com prazo de correção (para `approved_with_caveats`)
- `rejection_reason` — justificativa (para `rejected`)
- `deploy_scheduled_at` — janela de deploy confirmada

Status: `aprovado` | `aprovado_com_ressalvas` | `revisao_solicitada` | `rejeitado`.

### Critério de conclusão

Todos os aprovadores obrigatórios registraram `approved` ou `approved_with_caveats`. Nenhum bloqueio ativo. `deploy_scheduled_at` confirmado. Para `approved_with_caveats`: plano de correção das ressalvas registrado com prazo ≤ 2 sprints.

---

## Etapa 10 — Implantação Controlada

### Entrada

Item com status `aprovado` ou `aprovado_com_ressalvas`. `deploy_scheduled_at` confirmado. Rollback documentado e testado em staging.

### Atividade

**Pré-deploy (até 30 minutos antes da janela):**
- Tech Lead verifica que staging e produção estão no baseline esperado (sem outras mudanças não relacionadas).
- QA confirma que os alertas de monitoramento das métricas afetadas estão ativos.
- Product Specialist envia comunicado de manutenção ao time de atendimento se a mudança for de nível Alto ou Crítico.

**Durante o deploy:**
- Tech Lead executa o deploy conforme o `action_type`:
  - Prompt: atualização do arquivo de prompt + restart do serviço
  - Código: deploy da nova versão via pipeline CI/CD
  - Metadados/Ingestão: execução do script de atualização no índice
- Registrar no log de auditoria do BC-06: `deploy_id`, `feedback_ids_resolved[]`, `version_before`, `version_after`, `type_of_change`, `deployed_by`, `timestamp`.

**Pós-deploy imediato (primeiros 30 minutos):**
- QA executa manualmente o novo caso de regressão em produção (não em staging). Deve PASSAR.
- QA executa os casos `critical` do golden dataset contra o endpoint de produção. Todos devem PASSAR.
- Tech Lead monitora TTFT p95 em tempo real por 30 minutos.

**Critério de rollback automático nos primeiros 120 minutos:**
- Se M-07 (taxa de ❌) subir > 3 pontos percentuais em relação ao baseline das últimas 24h: rollback imediato.
- Se TTFT p95 ultrapassar 5s em janela de 15 minutos: rollback imediato.
- Se qualquer caso `critical` do golden dataset falhar em produção: rollback imediato.

O rollback não requer aprovação humana se ativado dentro dos primeiros 120 minutos. Após 120 minutos, requer decisão do Tech Lead + Product Specialist.

### Responsável

- **Tech Lead:** executa o deploy e o rollback se necessário
- **QA:** valida em produção pós-deploy
- **Product Specialist:** comunica ao time de atendimento; monitora feedback inicial

### Saída

Registro de deploy no BC-06 com `deploy_id`, versões antes/depois, `feedback_ids_resolved[]`. Resultado da validação pós-deploy registrado: novo caso de regressão em produção = PASSOU. Status: `implantado_em_producao`.

### Critério de conclusão

Deploy executado sem incidente nas primeiras 2 horas. Novo caso de regressão passou em produção. Casos `critical` do golden dataset passaram em produção. Nenhum critério de rollback automático ativado. Log de auditoria registrado com todos os campos obrigatórios.

---

## Etapa 11 — Monitoramento

### Entrada

Implantação em produção com status `implantado_em_producao`. Janela de monitoramento intensivo de 14 dias iniciada.

### Atividade

O Product Specialist e o Tech Lead monitoram as métricas afetadas pela mudança, conforme a tabela de métricas por tipo de mudança (harness de feedback, Seção 11.2):

**Diariamente (dias 1 a 7):**
- Verificar M-07 (feedback positivo/negativo) para o domínio afetado. Alerta se ❌ > 8% ou ✅ < 65%.
- Verificar M-08 (TTFT p95 e resposta completa p95). Alerta se TTFT > 4s por 2 dias consecutivos.
- Verificar se novos feedbacks com `feedback_category: F01` (incorreta) ou `F04` (desatualizada) surgiram sobre o mesmo tema. Qualquer ocorrência = investigar imediatamente.

**A cada 3 dias (dias 3, 6, 9, 12):**
- Verificar M-11 (ausência falsa) para o domínio afetado.
- Verificar M-10 (conflitos sinalizados) se a mudança afetou detecção de conflitos.
- Verificar M-02 (groundedness) para uma amostra de 10 respostas do domínio afetado.

**No dia 7 (revisão de meio de janela):**
- Product Specialist produz relatório intermediário: variação das métricas afetadas vs. baseline pré-deploy. Se qualquer métrica piorou > 5pp, avaliar rollback mesmo após os 120 minutos iniciais.

**No dia 14 (encerramento da janela):**
- Product Specialist produz relatório final de monitoramento com `metric_deltas` para todas as métricas afetadas.
- Contar feedbacks sobre o mesmo tema no período pós-deploy vs. pré-deploy. Esperado: redução.
- Registrar `outcome`: `resolved` | `partial` | `regressed`.

### Responsável

- **Product Specialist:** monitora métricas de produto (M-07, M-11, M-12) e feedbacks do domínio
- **Tech Lead:** monitora métricas técnicas (M-08, M-01) e logs de retrieval
- **QA:** executa amostragem de groundedness (M-02) nos dias 3, 6, 9, 12

### Saída

Relatório de monitoramento com `metric_deltas`, `new_feedback_on_same_topic`, `outcome`. Status: `monitoring`.

### Critério de conclusão

Janela de 14 dias encerrada. Relatório final produzido com `outcome` definido. Nenhuma métrica piorou além do limite de alerta durante a janela. Se `outcome: partial` ou `regressed`: novo item aberto e retorna à Etapa 1 com referência ao item atual.

---

## Etapa 12 — Rollback ou Consolidação

### Entrada

Relatório de monitoramento com `outcome` definido. Uma de duas condições:
- `outcome: resolved` → Consolidação
- `outcome: regressed` OU critério de rollback automático ativado → Rollback
- `outcome: partial` → Novo item aberto (retorna à Etapa 1)

### Atividade — Consolidação (outcome: resolved)

1. Product Specialist registra no item: `status: closed`, `closed_at`, `resolution_summary`.
2. QA atualiza o baseline da tabela de avaliação por dimensão com os resultados do candidato promovido (o candidato vira o novo baseline).
3. Tech Lead atualiza `version_after` como a versão oficial de produção nos documentos de ADR relevantes se a mudança afetou decisões arquiteturais.
4. Product Specialist envia notificação ao atendente que originou o feedback: "Seu feedback de [data] foi resolvido. A melhoria está disponível desde [data de deploy]."
5. Para mudanças de nível Alto e Crítico: Product Specialist envia comunicado coletivo ao grupo de atendentes descrevendo o que mudou e por que.
6. Gestor Documental atualiza o painel de gaps se o item resolveu um gap documental (remove da lista ou muda status para `coberto`).

### Atividade — Rollback (outcome: regressed ou critério automático)

1. Tech Lead executa o rollback para a versão anterior (`version_before`) imediatamente. Para rollback nos primeiros 120 minutos: sem aprovação necessária. Após 120 minutos: Tech Lead + Product Specialist decidem juntos.
2. Tech Lead registra no log de auditoria do BC-06: `rollback_id`, `rollback_reason`, `version_reverted_to`, `rolled_back_by`, `timestamp`.
3. QA confirma que o rollback foi executado corretamente executando os casos `critical` do golden dataset contra a versão revertida. Todos devem PASSAR.
4. Product Specialist registra o motivo do rollback no item e abre novo item filhote com `root_cause_id` atualizado, referenciando o que o rollback revelou.
5. Para rollback por critério automático (primeiros 120 minutos): Product Specialist notifica o time de atendimento de que a manutenção foi revertida.
6. O item original não é fechado — recebe status `rolled_back` e é vinculado ao novo item filhote que continuará o ciclo.

### Responsável

**Consolidação:**
- **Product Specialist:** fecha o item, envia comunicação
- **QA:** atualiza baseline
- **Tech Lead:** atualiza documentação técnica
- **Gestor Documental:** atualiza painel de gaps

**Rollback:**
- **Tech Lead:** executa o rollback
- **QA:** valida o rollback em produção
- **Product Specialist:** comunica e abre novo item

### Saída

**Consolidação:** item com status `closed`. Baseline atualizado. Notificação enviada ao atendente. Comunicado coletivo enviado (para Alto e Crítico).

**Rollback:** sistema em produção na versão anterior confirmada. Item com status `rolled_back`. Novo item filhote aberto com `root_cause_id` revisado. Log de rollback registrado no BC-06.

### Critério de conclusão

**Consolidação:** `status: closed` registrado. Baseline atualizado com os valores do candidato promovido. Notificação ao atendente enviada. QA confirmou que o novo baseline está registrado em `/tests/golden/golden-dataset.yaml`.

**Rollback:** produção revertida confirmada pelos casos `critical` do golden dataset. Novo item filhote aberto com causa raiz revisada. Log de rollback imutável registrado no BC-06.

---

## Resumo por Etapa

| Etapa | Entrada | Responsável principal | Saída | Critério de conclusão |
|---|---|---|---|---|
| **1. Feedback / Necessidade** | Feedback BC-06, alerta de métrica, incidente, necessidade | PS (produto) · TL (técnico) | Item de backlog com `origin_id` rastreável | Item registrado com campos obrigatórios |
| **2. Triagem** | Item registrado | PS (crítico/alto) · QA (médio/baixo) | Item com urgência, responsável e SLA definidos | Nível atribuído, duplicatas tratadas |
| **3. Causa raiz** | Item triado | PS + TL (crítico) · PS (alto) · QA (médio) | `root_cause_id` e `change_type` com justificativa | Root cause identificada com evidência de diagnóstico |
| **4. Proposta** | Causa raiz identificada | PS (produto) · TL (técnico) · Gestor Documental (docs) | Documento de proposta + novo caso de regressão FALHO | Proposta revisada por par; caso falho confirmado |
| **5. Implementação** | Proposta aceita | Engenharia · TL · Gestor Documental | Artefato em staging; PR aberto | Staging pronto, sem mudanças não relacionadas |
| **6. Testes automatizados** | Staging disponível | QA | Relatório + tabela preenchida (TD-01 a TD-15) | 100% dos testes determinísticos; novo caso PASSOU |
| **7. Avaliação semântica** | Testes automatizados aprovados | QA + PS (revisão) | Tabela preenchida (TS-01 a TS-08) | Todos os critérios semânticos ≥ meta |
| **8. Revisão humana** | Semântica aprovada | PS · Compliance (TH-05) | Pareceres TH-01 a TH-06 assinados | 4/4 carga perigosa; 2/2 ausência; zero riscos críticos |
| **9. Aprovação** | Revisão humana aprovada | PS + aprovadores da matriz | Decisão registrada com `deploy_scheduled_at` | Todos aprovadores assinaram; rollback documentado |
| **10. Implantação** | Aprovação concedida | TL (deploy) · QA (validação) | `deploy_id` no log BC-06; validação em produção | Caso regressão PASSOU em produção; zero rollback automático em 2h |
| **11. Monitoramento** | Deploy em produção | PS · TL · QA | Relatório com `metric_deltas` e `outcome` | 14 dias concluídos sem degradação acima do alerta |
| **12. Rollback / Consolidação** | `outcome` definido | PS · TL · QA · Gestor Documental | Item `closed` (consolidação) ou `rolled_back` + novo item (rollback) | Baseline atualizado (consolidação) ou produção revertida confirmada (rollback) |

---

*Documento elaborado com base no harness de métricas (novatech-harness-metricas.md), no fluxo de feedback (novatech-harness-feedback.md), no harness de validação pré-produção (novatech-harness-validacao.md) e na matriz de aprovação (novatech-harness-aprovacao.md). Este documento completa o harness de produto da Tarefa 1 (métricas), integrando todas as etapas em um fluxo operacional único.*
