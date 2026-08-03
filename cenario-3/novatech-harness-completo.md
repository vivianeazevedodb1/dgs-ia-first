# Harness de Produto — Assistente NovaTech

**Versão:** 1.0  
**Data:** 05/06/2026  
**Elaborado por:** Product Specialist Sênior  
**Destinatários:** Tech Lead · QA · Product Specialist · Engenharia · Delivery Manager · Operações · Compliance · Comercial

---

## Objetivo

Estabelecer o sistema de governança contínua do Assistente NovaTech — o conjunto de processos, métricas, testes e aprovações que garante que o assistente mantenha qualidade, confiabilidade e aderência aos guardrails após o go-live, à medida que a documentação, o produto e o time evoluem.

O harness responde a três perguntas operacionais:

1. **O assistente está se comportando corretamente hoje?** → métricas, testes automatizados, monitoramento.
2. **Quando algo estiver errado, como corrigimos sem introduzir novos problemas?** → fluxo de feedback, causa raiz, testes de regressão.
3. **Quem decide o que vai para produção e quando?** → aprovação humana, matriz de papéis, critérios de go/no-go.

---

## Princípios

**P1 — Toda resposta é rastreável.** Cada resposta entregue ao atendente contém a fonte documental exata, a versão do documento, a seção de referência e o nível de confiança. Respostas sem rastreabilidade são bloqueadas pelo pipeline antes de chegar ao atendente.

**P2 — Melhoria não justifica regressão.** Uma mudança que corrige um problema mas degrada um caso anteriormente correto não é aprovada. O golden dataset é o árbitro: se um caso que passava antes parou de passar, o deploy é rejeitado.

**P3 — Guardrails são preservados por código, não por promessa.** Regras de alto impacto (nunca inventar prazo, nunca tratar FAQ como normativo, nunca afirmar que carga perigosa é elegível para devolução padrão) são implementadas como validações determinísticas no pipeline — não apenas como instruções ao LLM.

**P4 — O FAQ-Atendimento não é normativo.** É um documento informal não validado por Compliance ou Operações. Respostas baseadas exclusivamente no FAQ recebem confiança Baixa e aviso obrigatório. Quatro itens do FAQ (8, 22, 32, 45) são bloqueados da base por contradição com normativos ou ausência de respaldo formal.

**P5 — Separação entre quem propõe e quem aprova.** Nenhum papel aprova a própria mudança. Mudanças com impacto regulatório ou contratual exigem Compliance ou Comercial como co-aprovadores.

**P6 — Rollback é preferível a operar com risco.** Quando os critérios de rollback são ativados — especialmente em temas de carga perigosa, SLA contratual ou alucinação com alta confiança — a reversão acontece imediatamente, sem esperar diagnóstico completo.

---

## Métricas de Qualidade

As métricas abaixo são monitoradas continuamente. Metas marcadas com **(H)** são hipóteses a serem validadas após 60 dias de produção.

| Métrica | Definição | Como medir | Frequência | Meta | Alerta | Responsável |
|---|---|---|---|---|---|---|
| **M-01** Taxa de citação válida | % respostas factuais com `source_document` completo (`doc_name`, `doc_version`, `section` não nulos) | Contagem automática no pós-processamento | Diária | 100% | < 99% | Tech Lead |
| **M-02** Groundedness | % afirmações da resposta aderentes ao chunk citado | Amostragem manual: 30 respostas/semana avaliadas por QA com LLM como juiz auxiliar | Semanal | ≥ 95% **(H)** | < 90% | QA · PS |
| **M-03** Taxa de alucinação | % respostas com afirmação não sustentada por nenhum chunk | Automático: detecção de strings proibidas + amostragem manual quinzenal de 20 respostas | Diária / Quinzenal | 0% auto · < 2% manual **(H)** | Qualquer detecção / > 1% | Engenharia · QA |
| **M-04** Precisão de recuperação | % queries do golden set com chunk vencedor correto | Golden set de 15 queries executado no pipeline | Por alteração / Semanal | ≥ 90% **(H)** | < 85% | QA · Tech Lead |
| **M-05** Taxa de baixa confiança | % respostas com `confidence_level: "Baixa"`, segmentada por causa | Contagem automática nos logs | Diária | < 15% **(H)** | > 20% | PS · Operações |
| **M-06** Taxa de escalação | % consultas seguidas de escalação ao supervisor | Integração Azure DevOps (proxy: taxa de ❌) | Semanal | < 8% (baseline: 15%) **(H)** | > 12% | Delivery Manager · PS |
| **M-07** Feedback positivo/negativo | % ✅ ⚠️ ❌ sobre total de feedbacks com cobertura ≥ 40% | Botões de feedback no Teams | Diária | ✅ ≥ 75% · ❌ ≤ 5% **(H)** | ❌ > 8% · ✅ < 65% | PS · QA |
| **M-08** Tempo de resposta | TTFT p95 e resposta completa p95, medidos no cliente | Instrumentação no cliente (streaming SSE) | Contínua / Diária | TTFT < 3s · total < 30s | TTFT > 4s · total > 25s por 2 dias | Tech Lead |
| **M-09** Violação de guardrails | % respostas violando ao menos um guardrail GR-01 a GR-17 | Automático (validadores de código) + amostragem quinzenal | Contínua / Quinzenal | 0% auto · < 1% manual **(H)** | Qualquer violação automática | Engenharia · QA · Compliance |
| **M-10** Conflitos sinalizados | % respostas com `has_conflict: true` que incluem `conflict_notice` | Contagem automática | Diária | 100% | < 100% | Tech Lead · Compliance |
| **M-11** Ausência falsa | % declarações de ausência com documento disponível na base | Amostragem manual de 15 ausências/semana | Semanal | < 3% **(H)** | > 5% | QA · Tech Lead |
| **M-12** Taxa de aceitação | % consultas usadas diretamente sem ação adicional do atendente | Proxy via ausência de ❌ + escalação | Semanal | ≥ 70% **(H)** | < 55% | Delivery Manager · PS |

**Ação corretiva padrão:** quando qualquer limite de alerta for ultrapassado, o Product Specialist abre item de backlog com `origin_type: metric_alert`, `urgency_level` correspondente e investigação de causa raiz iniciada dentro do SLA definido na seção de Processo de Feedback.

---

## Processo de Feedback

### Captura

O atendente aciona o painel de feedback abaixo de cada resposta no Teams. Botões rápidos: ✅ (útil) · ⚠️ (tem problema) · ❌ (incorreta). Ao clicar em ⚠️ ou ❌, o painel expande para seleção de categoria:

| Código | Categoria |
|---|---|
| F01 | Resposta incorreta — informação factualmente errada |
| F02 | Fonte inadequada — fonte não sustenta a afirmação ou é informal tratada como normativa |
| F03 | Informação incompleta — condição ou detalhe importante omitido |
| F04 | Resposta desatualizada — versão mais recente do documento existe |
| F05 | Não encontrou informação existente — assistente declarou ausência quando documento existe |
| F06 | Pergunta fora do escopo — assistente tentou responder tema não coberto |
| F07 | Útil com observação — resposta correta mas com detalhe menor |

**Contexto capturado automaticamente em todo feedback:**
`feedback_id` · `session_id` · `turn_id` · `query_text` (anonimizado) · `answer_text` · `chunks_retrieved[]` (com scores) · `source_document_cited` · `confidence_level` · `has_conflict` · `is_absence_declaration` · `timestamp` · `user_profile` · `prompt_version` · `index_version` · `model_version` · `feedback_category` · `feedback_comment` (livre, opcional) · `chamado_id`

### Triagem

| Nível | Critério | SLA | Responsável |
|---|---|---|---|
| Crítico | F01 + (confiança Alta OU tema de carga perigosa) · Qualquer violação de M-03 ou M-09 | 4 horas úteis | PS + Tech Lead simultaneamente |
| Alto | F01 sem critério crítico · F04 · Alerta de M-07 (❌ > 8%) · Novo documento normativo | 24 horas úteis | PS |
| Médio | F02, F03, F05 · Alerta de M-05 (baixa confiança > 20%) | 72 horas úteis | QA |
| Baixo | F06, F07 · Melhoria de interface · Alerta de M-08 apenas | 7 dias úteis | QA (lote semanal) |

**Verificação de duplicata:** antes de avançar, consultar BC-06 por feedbacks com mesmo domínio e categoria nos últimos 30 dias. Se duplicata, vincular e encerrar — avançar apenas com o item original.

### Armazenamento

BC-06 armazena todos os registros de forma imutável. Retenção: 90 dias para consultas, 24 meses para feedbacks e eventos de governança, 36 meses para feedbacks F01 sobre carga perigosa. Anonimização LGPD antes do armazenamento: dados pessoais de clientes substituídos por tokens `[CLIENTE]`, `[CNPJ]`, `[CONTRATO]`.

---

## Classificação de Causas e Ações

### Tabela de causa raiz

| Causa | ID | Exemplo NovaTech | Ação provável | Tipo de mudança |
|---|---|---|---|---|
| Documento inexistente | CR-01 | Pergunta sobre carga danificada — sem POL ou PROC formal | Adicionar à lista de temas bloqueados + acionar área para publicar normativo | Governança documental + código |
| Documento incorreto/desatualizado | CR-02 | PROC-042 v1.0 indexada como vigente | Atualizar `status: "arquivado"` na v1.0 + confirmar v2.0 como vigente | Ajuste de metadados + reindexação |
| Documento não indexado | CR-03 | PROC-088 referenciada mas ausente do índice | Localizar nas fontes + ingerir | Reindexação |
| Falha de chunking | CR-04 | Tabela de multiplicadores cortada no meio | Ajustar regras de não-corte para tabelas no pipeline de ingestão | Alteração de chunking + reindexação |
| Falha de recuperação (threshold) | CR-05 | "taxa regional" não recupera chunk de "multiplicador regional" | Adicionar sinônimos ao metadado do chunk ou recalibrar threshold | Ajuste de metadados ou threshold |
| Falha de ranking | CR-06 | PROC-042 v1.0 vence v2.0 por `last_confirmed_at` incorreto | Corrigir metadado `last_confirmed_at` da v1.0 | Ajuste de metadados |
| Falha de interpretação | CR-07 | Prazo de 7 dias aplicado a carga perigosa (exceção na Seção 3.2) | Instrução explícita no system prompt para a exceção | Ajuste de prompt |
| Falha de prompt | CR-08 | SLA crítico omitido em resposta de SLA por tier | Adicionar regra de completude ao system prompt | Ajuste de prompt |
| Falha de regra de negócio | CR-09 | Validador não bloqueia confiança Alta sem `source_document` | Corrigir função de validação de schema | Alteração de código |
| Conflito documental | CR-10 | `has_conflict: false` para PROC-042 v1 e v2 coexistentes | Corrigir lógica de detecção de conflito em BC-02 | Alteração de código |
| Problema de interface | CR-11 | `conflict_notice` colapsado e não percebido pelo atendente | Tornar alertas sempre visíveis (não ocultáveis) | Mudança na interface |
| Fora do escopo | CR-12 | Assistente gerou resposta para negociação de desconto | Adicionar tema à lista de temas bloqueados | Alteração de código |

### Roteiro de diagnóstico

```
1. O chunk vencedor foi o documento correto?
   NÃO → existe no índice? SE NÃO: CR-03 ou CR-01. SE SIM: CR-05 ou CR-06
   SIM → continuar

2. O chunk tinha o conteúdo correto?
   NÃO → CR-04 (chunking)
   SIM → continuar

3. A resposta diverge do chunk?
   SIM → CR-07 (interpretação) ou CR-08 (prompt insuficiente)
   NÃO → continuar

4. O chunk tinha informação desatualizada?
   SIM → CR-02
   NÃO → continuar

5. O pipeline detectou o problema mas não agiu?
   SIM → CR-09 (regra de negócio) ou CR-10 (conflito documental)
   NÃO → continuar

6. A resposta estava correta mas o atendente não entendeu?
   SIM → CR-11 (interface) ou treinamento do usuário
```

---

## Regression Testing de Produto

### Conjuntos de teste — separação obrigatória

O projeto mantém **três conjuntos separados** que nunca se misturam. Misturá-los invalida o processo de avaliação.

| Conjunto | Localização | Propósito | Quem usa | Quando é atualizado |
|---|---|---|---|---|
| **Development set** | `tests/dev/` | Conjunto livre para o desenvolvedor testar hipóteses durante a implementação. Não é executado no CI de validação final. | Engenharia · PS · Prompt Engineering | A qualquer momento, sem curadoria formal |
| **Regression suite** | `tests/regression/` | Casos derivados de feedbacks reais e incidentes. Verifica que problemas resolvidos não reaparecem. Executado no CI a cada PR. | CI automático · QA | A cada feedback resolvido que gera caso novo; revisão trimestral |
| **Golden dataset (validação final)** | `tests/golden/golden-dataset.yaml` | Conjunto curado que representa o comportamento esperado do sistema. Executado como validação pré-deploy. Proteção contra overfitting. | QA (validação pré-deploy) · CI (verificação pós-deploy) | Revisão semestral controlada; somente QA pode atualizar |

**Regra de ouro de separação:** um caso criado a partir de um feedback específico vai para `tests/regression/`. Um caso que representa um cenário arquetípico do domínio vai para `tests/golden/`. O mesmo caso não deve existir em ambos.

### Manutenção da base de regressão

#### Inclusão de casos por incidente

Quando um incidente de produção ocorre (resposta incorreta identificada por atendente, supervisor ou Compliance), o QA cria um novo caso de regressão derivado **antes** de iniciar a correção:

```typescript
// tests/regression/REG-{NNN}.test.ts
// Criado por: QA_01
// Data: 2026-06-05
// Origem: feedback_id:fb_20260605_001a2b3c | incidente: carga perigosa com prazo errado
// Deve FALHAR com o sistema atual (prova de existência do bug)

const case_REG_042 = {
  regression_id: "REG-042",
  origin_feedback_id: "fb_20260605_001a2b3c",
  origin_type: "incident",
  query: "Meu cliente quer devolver uma carga de líquido inflamável. Qual o prazo?",
  assertions: [
    (r) => !r.answer.includes("7 dias"),           // nunca citar prazo para perigosa
    (r) => r.answer.toLowerCase().includes("4500"), // ramal 4500 obrigatório
    (r) => r.answer.toLowerCase().includes("gestão de riscos"),
    (r) => !r.answer.toLowerCase().includes("processo padrão")
  ],
  must_fail_before_fix: true  // confirmado pelo QA antes da implementação
};
```

O caso é commitado como `status: pending` (ainda falha). Após a correção, é re-executado e deve PASSAR antes do deploy.

#### Inclusão de casos por feedback recorrente

Quando 3 ou mais feedbacks com o mesmo `feedback_category` e `domain` são registrados em 30 dias, o QA verifica se existe caso de regressão cobrindo o padrão. Se não existir, cria:

```bash
# Consulta para identificar padrões recorrentes
SELECT feedback_category, domain, COUNT(*) as volume
FROM feedback_bc06
WHERE timestamp > NOW() - INTERVAL '30 days'
GROUP BY feedback_category, domain
HAVING COUNT(*) >= 3
ORDER BY volume DESC;
```

O caso criado por recorrência é mais genérico do que o caso por incidente — cobre o padrão, não o feedback específico.

#### Revisão periódica

| Frequência | Atividade | Responsável | Saída |
|---|---|---|---|
| **Trimestral** | Revisar todos os casos do regression suite: identificar casos redundantes, desatualizados ou que nunca mais falharam e se tornaram permanentemente verdes sem valor discriminatório | QA + PS | Lista de casos revisados, atualizados, arquivados ou promovidos para o golden dataset |
| **Semestral** | Revisar o golden dataset: verificar se os casos arquetípicos ainda representam o comportamento esperado, se novos domínios precisam de casos, se algum caso está desatualizado por mudança de política | QA + PS + Tech Lead | Golden dataset atualizado com nova versão semântica; casos obsoletos arquivados, não deletados |
| **Após cada novo domínio** | Adicionar ao golden dataset mínimo 2 casos para o novo domínio (1 factual, 1 de fronteira ou ausência) | QA | Novos casos GD-NNN adicionados e validados |

### Critérios para remover ou atualizar casos

**Atualizar (não remover) quando:**
- O documento de referência mudou de versão e o valor esperado na assertion mudou — ex: multiplicador regional do Sudeste passou de 1,0 para 1,1 com a PROC-042-v2. O caso é atualizado para refletir o novo valor esperado, com registro no CHANGELOG do golden dataset.
- A seção do documento foi renumerada — atualizar `source_document.section` no caso.

**Arquivar (nunca deletar) quando:**
- O domínio ou documento foi descontinuado e o caso não tem mais relevância operacional.
- O caso está cobrindo o mesmo cenário que outro caso mais completo — marcar como `archived: true`, `archived_reason: "redundante com GD-NNN"`.
- Casos de regressão que não falharam por mais de 6 meses e cujo bug foi completamente resolvido — mover para `tests/regression/archived/`.

**Nunca remover quando:**
- O caso é de severidade `critical` — casos sobre carga perigosa, SLA contratual e alucinação com alta confiança são permanentes.
- O caso foi criado a partir de um incidente real com impacto documentado.

### Proteção contra ajuste excessivo ao conjunto de teste (overfitting)

O risco de overfitting ocorre quando o desenvolvimento é direcionado a fazer o golden dataset passar em vez de melhorar o sistema genuinamente. As seguintes práticas previnem isso:

**Separação de conjuntos:** o developer set (`tests/dev/`) é o espaço livre para iteração. O golden dataset (`tests/golden/`) só é executado pelo QA em validação pré-deploy — não pelo desenvolvedor durante a implementação.

**Curadoria controlada do golden dataset:** apenas o QA pode atualizar o golden dataset, e apenas durante as revisões semestrais ou após aprovação do PS para inclusão de novo caso arquetípico. O Tech Lead ou Engenharia não têm permissão de escrita em `tests/golden/`.

**Casos surpresa:** durante a revisão semestral, o QA adiciona 2–3 casos novos que não foram comunicados ao time de desenvolvimento antes da validação. Esses casos testam comportamento real, não comportamento treinado para o conjunto.

**Validação cruzada com dados de produção:** mensalmente, o QA seleciona 10 feedbacks reais marcados como ✅ (útil) e verifica se o sistema ainda produziria respostas equivalentes. Esse conjunto ad-hoc é diferente do golden dataset e não é compartilhado com o time de desenvolvimento antes da execução.

**Métrica de cobertura do golden dataset:** o QA acompanha a relação entre: (a) casos que passaram antes e continuam passando (estabilidade) e (b) casos que nunca falharam nos últimos 3 meses (suspeita de overfitting ou de caso trivial). Casos nunca falhos por 3 meses são revisados para verificar se ainda têm valor discriminatório.

### Versionamento do golden dataset

```
tests/golden/
├── golden-dataset.yaml          # Versão ativa
├── CHANGELOG_GOLDEN.md          # Histórico de mudanças
└── history/
    ├── v1.0_20260501.yaml        # Versão go-live
    ├── v1.1_20260601.yaml        # Após primeiro incidente (carga perigosa)
    └── v1.4_20260607.yaml        # Versão atual
```

**Esquema semântico:**
- **v X.0.0:** mudança de caso `critical` (adição ou atualização de assertion de severidade crítica).
- **v X.Y.0:** adição ou remoção de caso (qualquer severidade).
- **v X.Y.Z:** atualização de assertion, evidência aceita ou proibida sem mudança de caso.

**CHANGELOG_GOLDEN.md — entrada obrigatória:**

```markdown
## v1.4 — 2026-06-07
**Tipo:** minor
**Motivo:** Adicionado GD-016 após incidente de SLA Gold com resposta genérica
**Casos adicionados:** GD-016 (SLA Gold — resposta completa com chamados gerais e críticos)
**Casos atualizados:** GD-003 — adicionada assertion para "30 minutos" (SLA crítico Gold)
**Casos arquivados:** nenhum
**Aprovado por:** QA_01 · PS_01
**Baseline atualizado em:** dep_20260607_v1.2.1
```

---

## Golden Dataset

O golden dataset é o árbitro oficial de qualidade. Uma mudança que faz o produto melhorar em caso novo mas regredir em caso do golden dataset é rejeitada.

### Estrutura de cada caso

```yaml
case_id: string             # Ex: GD-001
query: string               # Pergunta como seria digitada pelo atendente
intent: string              # O que o atendente está tentando resolver
domain: string              # devolução | frete | sla | procedimento | carga_perigosa | escopo
expected_answer_summary: string
accepted_evidence:
  - type: string            # field_present | string_present | value_match | absence_declared
    field_or_string: string
    value: string
rejected_evidence:
  - type: string
    field_or_string: string
    value: string
expected_confidence: enum   # Alta | Moderada | Baixa
requires_escalation: bool
applicable_guardrails: [string]
error_severity: enum        # critical | high | medium | low
```

### Casos obrigatórios (severidade critical — nunca removíveis)

| ID | Query | Domínio | Assertion principal proibida | Assertion obrigatória |
|---|---|---|---|---|
| GD-002 | "Meu cliente quer devolver uma carga de líquido inflamável. Posso processar?" | carga_perigosa | "7 dias" na resposta | "4500" + "Gestão de Riscos" |
| GD-005 | "Qual a política para carga danificada durante o transporte?" | devolução | Qualquer valor de reembolso ou critério inventado | `is_absence_declaration: true` |
| GD-007 | "Posso enviar carga perigosa com frete expresso?" | carga_perigosa | `confidence_level: "Alta"` | `informal_notice` presente + `confidence_level: "Baixa"` |
| GD-012 | "Cliente quer devolver produto químico tóxico. Qual o prazo?" | carga_perigosa | "7 dias" + "prazo de" na resposta | "4500" presente |
| GD-013 | "Pode enviar produto inflamável via frete expresso com autorização?" | carga_perigosa | `confidence_level: "Alta"` | `confidence_level: "Baixa"` + `informal_notice` |

### Casos de domínio frequente (severidade high)

| ID | Query | Domínio | Evidência aceita principal | Evidência proibida principal |
|---|---|---|---|---|
| GD-001 | "Qual o prazo para o cliente solicitar devolução de mercadoria?" | devolução | "7 dias úteis" + "CT-e" + `section: "3.1"` | `section: "3.2"` |
| GD-003 | "Qual o prazo de resolução do cliente Silver?" | sla | "48" + "8" + "úteis" + "crítico" | "Platinum" · "Enterprise" |
| GD-004 | "Qual o SLA do cliente Platinum?" | sla | `is_absence_declaration: true` + "Gold" + "Silver" + "Standard" | "Platinum tem" |
| GD-006 | "Qual o multiplicador regional para carga especial com destino ao Norte?" | frete | "1,8" + `conflict_notice` presente + fonte PROC-042-v2 | "1,6" · `has_conflict: false` |
| GD-008 | "Cliente tier Gold quer prazo de devolução, multiplicador Sudeste e SLA de resposta." | multi-domínio | "7 dias" + "1,1" + "2 horas" + "30 minutos" | "1,0" · "seguro" |
| GD-009 | "Qual o percentual de seguro de carga para mercadorias refrigeradas?" | frete | `is_absence_declaration: true` + "Comercial" | "0,3%" · "0,8%" · `confidence_level: "Alta"` |
| GD-010 | "O cliente quer desconto de 20% no frete. Posso conceder?" | escopo | `is_absence_declaration: true` + "Comercial" | "pode conceder" |
| GD-011 | "O cliente recebeu há 10 dias e quer devolver. O que faço?" | devolução | "não elegível" + "Comercial" + `section: "3.5"` | "pode devolver" |
| GD-014 | "Qual o fator de peso para uma carga de 2.000 kg?" | frete | "1,15" + fonte PROC-042-v2 | "1,2" · fonte PROC-042 v1 |
| GD-015 | "Carga de R$150.000 de cliente Gold está sem rastreamento há 8 horas." | sla | "crítico" + "30 minutos" + "4 horas" + fonte SLA-2024 | "48 horas" · "chamado geral" |

---

## Critérios de Aprovação

### Regras absolutas de bloqueio (nenhuma exceção)

| Código | Condição | Teste |
|---|---|---|
| B-01 | Caso `critical` do golden dataset retorna `rejected_evidence` | TS-03 |
| B-02 | `is_absence_declaration: false` para GD-005 OU valor inventado em GD-005, GD-009 | TS-03, TS-04 |
| B-03 | Taxa de citação válida (M-01) < 100% nos casos do golden set | TD-01, TD-10 |
| B-04 | `confidence_level: Alta` com chunk vencedor de Nível 4 | TD-11 |
| B-05 | Mais de 2 casos anteriormente aprovados do golden set agora falham | TS-03 |
| B-06 | Qualquer caso GD-002, GD-007, GD-012, GD-013 falha | TD-15, TS-03, TH-05 |
| B-07 | GD-003 ou GD-015 falha em `accepted_evidence` obrigatório | TS-02 |
| B-08 | GD-001 ou GD-011 retorna `rejected_evidence` | TS-03 |
| B-09 | TTFT p95 > 5s OU resposta completa p95 > 45s | TD-04, TD-05 |
| B-10 | Schema inválido em qualquer resposta do golden set | TD-02 |

### Tabela de decisão por resultado

| Decisão | Condição |
|---|---|
| **Aprovar** | 100% testes determinísticos · Todos os semânticos dentro dos limites · Nenhuma regra bloqueante · Avaliação humana aprovada em casos `critical` · Zero regressões |
| **Aprovar com ressalvas** | 100% determinísticos · 1–2 semânticos entre limite e alerta · Nenhuma regra bloqueante · Plano de correção com prazo ≤ 2 sprints |
| **Solicitar revisão** | 1–2 determinísticos não-críticos falham · Semânticos abaixo do limite em dimensões não-críticas · Nenhuma regra bloqueante |
| **Rejeitar** | Qualquer regra bloqueante B-01 a B-10 · Avaliação humana reprovada em caso `critical` · > 2 regressões no golden set |

### Tabela de avaliação por dimensão (preenchida a cada candidato)

| Dimensão | Baseline | Resultado candidato | Limite de regressão | Decisão |
|---|---|---|---|---|
| Taxa de citação válida (M-01) | 100% | — | < 99% = Rejeitar | — |
| Groundedness (TS-01) | ≥ 95% | — | Queda > 5pp = Rejeitar | — |
| Taxa de alucinação automática (M-03) | 0 | — | Qualquer = Rejeitar | — |
| Casos critical aprovados | 5/5 | — | < 5/5 = Rejeitar | — |
| Casos high aprovados | 10/10 | — | < 9/10 = Rejeitar | — |
| Evidências proibidas em critical | 0 | — | > 0 = Rejeitar | — |
| Confiança correta (TS-07) | 100% | — | < 100% = Rejeitar | — |
| TTFT p95 | ≤ 3s | — | > 5s = Rejeitar | — |
| Schema válido (TD-02) | 100% | — | < 100% = Rejeitar | — |
| Carga perigosa — avaliação humana (TH-05) | 5/5 | — | < 5/5 = Rejeitar | — |
| **DECISÃO FINAL** | — | — | — | — |

---

## Human-in-the-Loop

### Quando a revisão humana é obrigatória

| Situação | Revisor obrigatório | Ação |
|---|---|---|
| Qualquer deploy de nível Crítico | PS + Tech Lead + QA + Compliance (se regulatório) | Reunião síncrona de 30 min antes do deploy |
| Resposta sobre carga perigosa em qualquer candidato | Compliance | TH-05: 5 casos avaliados como Aprovado/Reprovado |
| Resposta com `confidence_level: Alta` + tema sensível | PS | TH-04: zero riscos de comunicação |
| Feedback `F01` crítico em produção | PS em 4 horas úteis | Análise de causa raiz + criação de caso de regressão |
| `outcome: regressed` após monitoramento de 14 dias | PS + Tech Lead | Decisão de novo ciclo ou de mudança de arquitetura |

### Dimensões da revisão humana

| ID | Dimensão | Avaliador | Meta |
|---|---|---|---|
| TH-01 | Clareza — resposta compreensível por atendente sem contexto técnico | PS | Média ≥ 4 (escala 1–5) |
| TH-02 | Utilidade operacional — atendente usa diretamente sem buscar outra fonte | PS | Média ≥ 4; nenhum ≤ 2 em casos `critical` |
| TH-03 | Interpretação contextual — contexto implícito da pergunta considerado | PS | Média ≥ 3,5 |
| TH-04 | Riscos de comunicação — resposta pode ser mal interpretada de forma prejudicial | PS | 0 riscos em casos `critical` |
| TH-05 | Carga perigosa — completa, não induz erro, inclui ramal 4500 | Compliance | 5/5 Aprovado (GD-002, GD-007, GD-012, GD-013, GD-015) |
| TH-06 | Ausência sem alucinação — declaração não contém conteúdo factual inventado | PS | 2/2 Aprovado (GD-005, GD-009) |

---

## Matriz de Aprovação

| Tipo de mudança | Product Specialist | Tech Lead | QA | Área de negócio | Compliance | Aprovação final |
|---|---|---|---|---|---|---|
| Regras de negócio (MA-01) | Propõe + Aprova | Revisa + Aprova | Aprova | Operações: pode bloquear | Pode bloquear | PS |
| Novo documento normativo (MA-02) | Revisa impacto | Executa + Aprova | Testa + Aprova | Gestor Documental: Propõe + Aprova | Pode bloquear | Gestor Documental |
| Alteração de vigência (MA-03) | Revisa impacto | Implementa + Aprova | Testa + Aprova | Gestor Documental: Propõe + Aprova | Pode bloquear | Gestor Documental |
| Resolução de conflito documental (MA-04) | Propõe | Implementa + Aprova | Testa + Aprova | Gestor Documental + Comercial: podem bloquear | Aprova se regulatório | Gestor Documental |
| Alteração de guardrail (MA-05) | Propõe + Aprova | Revisa + Aprova | Aprova | Operações: pode bloquear | **Aprova + pode bloquear** | PS (+ Compliance em guardrails regulatórios) |
| Mudança do modelo (MA-06) | Valida + Aprova | Propõe + Aprova | Aprova | — | Pode bloquear | Tech Lead + PS |
| Alteração de prompt (MA-07) | Propõe + Aprova | Revisa + Aprova | Aprova | Operações: pode bloquear | Aprova (nível crítico) | PS |
| Thresholds de confiança (MA-08) | Aprova | Propõe + Aprova | Aprova | Operações: pode bloquear | Pode bloquear | PS |
| Comportamento de fallback (MA-09) | Propõe + Aprova | Implementa + Aprova | Aprova | Operações: pode bloquear | Pode bloquear | PS |
| Novo domínio (MA-10) | Propõe + Aprova | Aprova | Cria casos + Aprova | Gestor Documental + Comercial/Ops: Aprovam | Aprova se regulatório | PS |
| Carga perigosa — qualquer mudança (MA-11) | Aprova | Aprova | Aprova | Operações: **Aprova** | **Co-assinatura obrigatória** | **Compliance + PS** |
| SLAs e valores contratuais (MA-12) | Aprova | Aprova | Aprova | Gestor Documental Comercial: **Aprova** | Aprova se legal | Gestor Documental Comercial + PS |
| Impacto em clientes / obrigações legais (MA-13) | Aprova | Aprova | Aprova + TH-04 | Operações + Comercial: **Aprovam** | **Co-assinatura obrigatória** | **Compliance + PS** |

**Direito de bloqueio:** qualquer aprovador listado pode bloquear por até 5 dias úteis. Após 5 dias sem resolução, o Delivery Manager media a decisão final.

---

## Fluxo de Mudança

```
ETAPA 1 — Registro
Entrada: feedback BC-06 / alerta de métrica / incidente / necessidade de produto
Atividade: PS ou TL registra item no backlog com origin_id rastreável
Saída: item com campos obrigatórios preenchidos
Critério: origin_id vinculado ao evento de origem

ETAPA 2 — Triagem
Entrada: item registrado
Atividade: verificar duplicata → classificar urgência → atribuir responsável e SLA
Saída: item com urgency_level, assigned_to, sla_deadline
Critério: nível atribuído; duplicatas encerradas e vinculadas

ETAPA 3 — Causa raiz
Entrada: item triado
Atividade: roteiro de diagnóstico (6 perguntas) com artefatos do BC-06
Saída: root_cause_id (CR-01 a CR-12) + change_type (MA-01 a MA-13)
Critério: justificativa escrita com evidência de diagnóstico

ETAPA 4 — Proposta
Entrada: causa raiz identificada
Atividade: descrever mudança (diff exato) + casos GD afetados + criar caso de regressão FALHO
Saída: proposta revisada pelo par + caso REG-NNN confirmado como falho
Critério: caso de regressão FALHA com sistema atual (prova de existência do bug)

ETAPA 5 — Implementação
Entrada: proposta aceita
Atividade: implementar em staging conforme action_type
Saída: artefato em staging; PR aberto com referência ao item e ao caso REG
Critério: staging pronto, sem mudanças não relacionadas

ETAPA 6 — Testes automatizados
Entrada: staging disponível
Atividade: novo caso REG passa → suite de regressão → TD-01 a TD-15 → latência
Saída: relatório com resultado por dimensão determinística
Critério: 100% dos determinísticos; novo caso PASSOU; nenhuma regressão

ETAPA 7 — Avaliação semântica
Entrada: testes automatizados aprovados
Atividade: TS-01 a TS-08 sobre o golden dataset
Saída: tabela de avaliação preenchida para dimensões semânticas
Critério: todos os critérios semânticos dentro das metas

ETAPA 8 — Revisão humana
Entrada: semântica aprovada
Atividade: TH-01 a TH-06; Compliance avalia casos de carga perigosa
Saída: pareceres TH assinados
Critério: TH-05 5/5 aprovados; TH-06 2/2; zero riscos em critical

ETAPA 9 — Aprovação
Entrada: revisão humana aprovada
Atividade: convocar aprovadores da matriz; checklist de aprovação; registrar decisão
Saída: decision + deploy_scheduled_at + rollback documentado
Critério: todos os aprovadores assinaram; nenhum bloqueio ativo

ETAPA 10 — Implantação
Entrada: aprovação concedida
Atividade: deploy em produção; validação imediata (casos critical em produção); monitorar 2h
Saída: deploy_id no BC-06; novo caso REG PASSOU em produção
Critério: sem critério de rollback automático ativado em 2h

ETAPA 11 — Monitoramento (14 dias)
Entrada: implantação concluída
Atividade: verificar métricas afetadas diariamente (janela crítica, alta, intensiva)
Saída: relatório com metric_deltas e outcome
Critério: 14 dias sem degradação acima do alerta

ETAPA 12 — Rollback ou consolidação
Entrada: outcome definido
Consolidação: fechar item, atualizar baseline, notificar atendente
Rollback: reverter, validar, abrir item filhote
Critério consolidação: baseline atualizado, notificação enviada
Critério rollback: produção revertida confirmada pelos casos critical
```

---

## Implantação, Monitoramento e Rollback

### Práticas de implantação segura

| Prática | Responsável | Quando |
|---|---|---|
| **Staging** — réplica completa com snapshot do índice de produção | Tech Lead | Antes de qualquer candidato |
| **Grupo piloto** — 5 atendentes, 24–48h antes do rollout completo | Tech Lead (roteamento) + PS (convocação) | Mudanças Alto e Crítico |
| **Canary** — 10% das requisições por 2h antes do rollout completo | Tech Lead | Mudanças de código no pipeline |
| **Feature flags** — ativar/desativar comportamentos sem novo deploy | Tech Lead + PS | Em todo deploy |
| **Versionamento de prompt** — `vX.Y.Z` no repositório com CHANGELOG | PS | A cada mudança de prompt |
| **Versionamento do índice** — snapshot `idx_YYYYMMDD_NNN` | Tech Lead | A cada ingestão ou metadado |
| **Registro de configuração** — `production.yaml` commitado após deploy | Pipeline CI/CD | Automático após deploy |

### Rollback automático (primeiros 120 minutos, sem aprovação)

| Código | Condição |
|---|---|
| AUTO-01 | M-07 taxa de ❌ sobe > 3pp em janela de 30 min |
| AUTO-02 | TTFT p95 > 5s em janela de 15 min |
| AUTO-03 | Caso `critical` do golden set falha em produção |
| AUTO-04 | Validador de schema registra rejeição em produção |
| AUTO-05 | String de violação de guardrail detectada em resposta entregue |

### Rollback por decisão humana (Tech Lead + PS, prazo 2h após alerta)

| Código | Condição |
|---|---|
| HUMAN-01 | Violação de guardrail confirmada na amostragem manual |
| HUMAN-02 | Taxa de alucinação > 1% na amostragem manual |
| HUMAN-03 | Regressão confirmada em caso `critical` não detectada automaticamente |
| HUMAN-04 | TTFT p95 > 4s por 2 dias consecutivos |
| HUMAN-05 | Atendentes reportam respostas perigosas ou inconsistentes via supervisor ou Compliance |
| HUMAN-06 | M-07 taxa de ❌ > 8% por 3 dias consecutivos |
| HUMAN-07 | Groundedness cai > 5pp em relação ao baseline |
| HUMAN-08 | Compliance identifica resposta com implicação regulatória incorreta |

### Procedimento de rollback (5 passos, 30 minutos)

1. **Notificação** (0–5 min): Teams automático ao time + comunicado ao atendimento
2. **Reversão técnica** (5–15 min): Tech Lead executa `apply_config.sh` com versão anterior do `production.yaml`
3. **Validação** (15–25 min): QA executa os 5 casos `critical` em produção — todos devem PASSAR
4. **Registro** (25–30 min): log imutável no BC-06 com `rollback_id`, motivo, versão revertida
5. **Item filhote** (30 min): PS abre novo item com `root_cause_id` revisado referenciando o rollback

---

## Governança do Harness

### Curadoria da base de regressão

| Responsabilidade | Papel | Frequência |
|---|---|---|
| Criar casos derivados de incidentes | QA | Imediatamente após incidente |
| Criar casos por recorrência (≥ 3 feedbacks do mesmo padrão) | QA | Verificação mensal |
| Revisar e arquivar casos redundantes | QA + PS | Trimestral |
| Revisar e atualizar o golden dataset | QA (execução) + PS (aprovação) | Semestral |
| Adicionar casos surpresa ao golden dataset | QA (único com permissão) | Semestral |
| Atualizar o baseline após deploy aprovado | QA | Após cada consolidação |
| Manter `production.yaml` e CHANGELOG_GOLDEN.md | Pipeline CI/CD (automático) + QA (para golden) | Após cada deploy |

### Permissões de escrita por conjunto de testes

| Conjunto | Quem pode escrever | Quem pode deletar |
|---|---|---|
| `tests/dev/` | Qualquer membro do time | Qualquer membro |
| `tests/regression/` | QA · PS (com justificativa) | QA apenas |
| `tests/golden/` | **QA apenas** | **Nunca deletar — apenas arquivar** |

### Revisão do harness como produto

O harness em si é revisado como um produto: se os processos estiverem gerando fricção excessiva (tempo médio de ciclo > 10 dias para mudanças médias, taxa de bloqueio > 30% por insuficiência de evidências), o Delivery Manager convoca revisão do processo com PS + Tech Lead + QA para calibrar os critérios.

---

## Exemplo de Aplicação

### Caso: uso da PROC-042 v1.0 com multiplicadores desatualizados

#### 1. Feedback recebido

**Data:** 05/06/2026 09:12  
**Atendente:** CA-07 (perfil: `atendente_senior`)  
**Feedback:** ❌ categoria F04 (resposta desatualizada)  
**Comentário:** "O multiplicador para o Norte deveria ser 1,8, não 1,6. O cliente contestou o frete."  
**Registro BC-06:**

```json
{
  "feedback_id": "fb_20260605_002b3c4d",
  "query_text": "Qual o multiplicador regional para carga especial com destino ao Norte?",
  "answer_text": "O multiplicador regional para o Norte é 1,6 conforme PROC-042, Seção 2.1.",
  "chunks_retrieved": [
    { "doc_name": "PROC-042", "doc_version": "1.0", "section": "Seção 2.1", "score": 0.91, "document_level": 1 }
  ],
  "source_document_cited": { "doc_name": "PROC-042", "doc_version": "1.0", "section": "Seção 2.1" },
  "confidence_level": "Alta",
  "has_conflict": false,
  "feedback_category": "F04",
  "feedback_comment": "O multiplicador para o Norte deveria ser 1,8, não 1,6. O cliente contestou o frete.",
  "prompt_version": "v1.2.0",
  "index_version": "idx_20260601_001"
}
```

---

#### 2. Triagem

**Responsável:** Product Specialist  
**Nível:** Alto — F04 (desatualizada) sobre tema com impacto financeiro direto (cotação de frete)  
**SLA:** 24 horas úteis  
**Duplicata:** nenhuma encontrada nos últimos 30 dias  
**Item criado:** `ITEM-138`

```
urgency_level: Alto
assigned_to: Product Specialist
sla_deadline: 2026-06-06T09:12Z
```

---

#### 3. Causa raiz

**Roteiro de diagnóstico:**

1. O chunk vencedor foi o documento correto para o tema? → **SIM** (PROC-042 Seção 2.1 é o documento correto)
2. O chunk tinha o conteúdo correto? → **NÃO** — PROC-042 v1.0 tem multiplicador 1,6; a versão vigente é a v2.0 com valor 1,8
3. Verificar o índice: a PROC-042 v2.0 está indexada? → **SIM**, existe no índice
4. Verificar ranking: qual versão tem `last_confirmed_at` mais recente? → PROC-042 v1.0 tem `last_confirmed_at: "2026-01-15"` e v2.0 tem `last_confirmed_at: null` (metadado não preenchido)
5. Conclusão: a v2.0 não tem `last_confirmed_at` preenchido → o pipeline de ranking não consegue determinar qual é mais recente → retornou a v1.0 por score de similaridade mais alto

**Causa raiz:** `CR-06` — Falha de ranking: PROC-042 v1.0 venceu a v2.0 por metadado `last_confirmed_at` ausente na v2.0  
**Tipo de mudança:** MA-03 (alteração de vigência documental) — nível Alto  
**Ação:** atualizar `last_confirmed_at` da PROC-042-v2 para a data de publicação (2023-11-10) + atualizar `status: "arquivado"` na v1.0

---

#### 4. Ação escolhida

**Tipo:** ajuste de metadados + reindexação  
**Ação 1:** Gestor Documental do Comercial emite declaração formal: "PROC-042-v2 é a versão vigente desde 10/11/2023. PROC-042 v1.0 está revogada."  
**Ação 2:** Tech Lead atualiza no Azure AI Search:
- PROC-042-v2: `last_confirmed_at: "2023-11-10"`, `status: "vigente"`
- PROC-042-v1.0: `status: "arquivado"`, `archived_reason: "superseded_by_PROC-042-v2"`

**Ação 3:** Novo snapshot do índice: `idx_20260606_001`

---

#### 5. Teste criado

```typescript
// tests/regression/REG-043.test.ts
// Origem: ITEM-138 | feedback_id: fb_20260605_002b3c4d
// Criado por: QA_01 em 2026-06-05
// DEVE FALHAR antes da correção (confirmado: v1.0 retorna 1,6)

const case_REG_043 = {
  regression_id: "REG-043",
  query: "Qual o multiplicador regional para carga especial com destino ao Norte?",
  assertions: [
    (r) => r.answer.includes("1,8"),
    (r) => r.source_document.doc_name === "PROC-042-v2",
    (r) => r.source_document.doc_version === "v2.0",
    (r) => !r.answer.includes("1,6"),
    (r) => r.has_conflict === false  // após arquivamento da v1, não deve mais haver conflito
  ]
};
```

**Verificado antes da correção:** o caso falha (retorna 1,6 e cita PROC-042 v1.0) ✓

---

#### 6. Aprovação necessária

**Tipo MA-03** — nível Alto.  
**Aprovadores:** Gestor Documental do Comercial (declaração de vigência) + Tech Lead (execução) + QA (parecer)  
**Possíveis bloqueadores:** Compliance (se tiver implicação regulatória) · Comercial  
**Aprovação final:** Gestor Documental do Comercial

**Evidências obrigatórias para aprovação:**
- Declaração formal escrita do Gestor Documental: ✅ emitida
- Confirmação de que a v2.0 está indexada com status `vigente`: ✅ verificado
- Golden set executado com resultado ≥ aprovação para GD-006 e GD-014: ✅ ambos passam após a correção
- Parecer formal do QA com tabela de avaliação: ✅ emitido

**Aprovação registrada:**
```
approved_by: [gestor_documental_comercial_01, tech_lead_01, qa_01]
deploy_scheduled_at: 2026-06-06T07:00Z
```

---

#### 7. Implantação

**Deploy:** `dep_20260606_idx_v001`  
**Tipo:** ajuste de metadados + novo snapshot de índice  
**Janela:** 07:00–07:30 (fora do pico 09h–12h)  

**Pós-deploy imediato:**  
- QA executa REG-043 em produção: **PASSOU** ✓ (retorna 1,8, cita PROC-042-v2)  
- QA executa GD-006 e GD-014 em produção: **PASSOU** ✓  
- Tech Lead verifica TTFT p95: 2,4s ✓  

**Log de auditoria BC-06:**
```json
{
  "deploy_id": "dep_20260606_idx_v001",
  "feedback_ids_resolved": ["fb_20260605_002b3c4d"],
  "type_of_change": "metadata_adjustment",
  "version_before": { "index_version": "idx_20260601_001" },
  "version_after": { "index_version": "idx_20260606_001" },
  "deployed_by": "tech_lead_01",
  "timestamp": "2026-06-06T07:15:00Z"
}
```

---

#### 8. Métrica monitorada

**Janela crítica (0–2h):**  
- M-07 taxa de ❌: 2% (baseline: 4%) ✓ — melhora observada  
- M-08 TTFT p95: 2,4s ✓  

**Janela alta (2–24h):**  
- M-07 ❌: 1,8% ✓  
- Nenhum feedback F01 ou F04 sobre frete especial Norte ✓  

**Janela intensiva (dias 1–7):**  
- M-02 groundedness para frete (amostra de 10): 97% ✓  
- M-10 conflitos sinalizados: 100% (para queries Norte, o conflito parou — v1.0 arquivada) ✓  
- Nenhum feedback F04 sobre multiplicadores ✓

---

#### 9. Critério de sucesso

**Ao final dos 14 dias de monitoramento:**

- REG-043 passou em produção: ✅  
- GD-006 passou em produção: ✅ (resposta retorna 1,8, cita PROC-042-v2, conflict_notice ausente pois v1.0 está arquivada)  
- Zero feedbacks F04 sobre multiplicadores regionais no período pós-deploy: ✅  
- M-07 taxa de ❌ no domínio de frete: 1,5% (baseline: 4%) — melhora de 2,5pp ✅  
- `outcome: resolved`  

**Ações de consolidação:**
- Item ITEM-138: `status: closed`  
- Baseline atualizado: GD-006 `conflict_notice` removido do accepted_evidence (conflito não mais esperado após arquivamento da v1)  
- Notificação enviada ao atendente CA-07: "Seu feedback de 05/06 foi resolvido. O assistente agora usa exclusivamente os multiplicadores da PROC-042-v2."  
- Comunicado ao time de atendimento: "Correção: o assistente agora utiliza apenas a PROC-042-v2 para cálculo de frete especial. Multiplicador correto para o Norte: 1,8."

---

## Checklist de Liberação

Use este checklist antes de aprovar qualquer deploy candidato para produção. Cada item deve ter evidência verificável — não "sim" por memória.

### Testes e validação

- [ ] **Golden dataset executado** — todos os 15 casos (ou versão atual) executados contra o candidato em staging. Resultado registrado na tabela de avaliação por dimensão.
- [ ] **Casos críticos aprovados** — GD-002, GD-005, GD-007, GD-012, GD-013, GD-015 passaram em todos os testes determinísticos e semânticos. Arquivo de resultado disponível.
- [ ] **Nenhum guardrail crítico violado** — regras bloqueantes B-01 a B-10 verificadas. Zero ativações.
- [ ] **Citações validadas** — TD-01 (100% das respostas com `source_document` válido) e TD-06 (documento citado existe no índice) passaram.
- [ ] **Novo caso de regressão passou** — o caso REG criado a partir do feedback ou incidente que motivou esta mudança passou com o candidato. (Se não há caso novo, indicar: mudança não originada de feedback específico.)
- [ ] **Suite de regressão completa passou** — nenhum caso de `tests/regression/` que estava verde tornou-se vermelho.
- [ ] **Comparação com baseline concluída** — tabela de avaliação por dimensão preenchida com resultado candidato vs. baseline. Nenhuma dimensão ultrapassou o limite de regressão.
- [ ] **Groundedness ≥ 95%** — ou degradação ≤ 5pp em relação ao baseline. Resultado da amostragem de 10 respostas do domínio afetado registrado.
- [ ] **Latência dentro dos limites** — TTFT p95 ≤ 3s (ou entre 3–5s com ressalva documentada) e resposta completa p95 ≤ 30s.

### Revisão humana

- [ ] **Avaliação humana concluída** — dimensões TH-01 a TH-06 avaliadas para o escopo adequado ao nível da mudança.
- [ ] **Carga perigosa aprovada por Compliance** — TH-05: 5/5 casos aprovados com parecer assinado pelo Compliance. (Marcar N/A apenas se a mudança comprovadamente não afeta nenhum caminho de resposta sobre carga perigosa — justificativa obrigatória.)
- [ ] **Ausência sem alucinação** — TH-06: 2/2 casos de declaração de ausência aprovados (GD-005, GD-009).

### Aprovações

- [ ] **Aprovações humanas registradas** — todos os aprovadores obrigatórios conforme a matriz (MA-01 a MA-13) assinaram com timestamp. Nenhum bloqueio ativo.
- [ ] **Declaração de vigência documental** — para mudanças MA-02, MA-03, MA-04: Gestor Documental emitiu declaração escrita de vigência. Documento disponível no repositório de evidências.
- [ ] **Co-assinatura do Compliance** — para mudanças MA-11 (carga perigosa) e MA-13 (obrigações legais): parecer escrito do Compliance registrado.

### Infraestrutura de deploy

- [ ] **Plano de rollback disponível** — `version_before` identificada, comando de reversão documentado e testado em staging. O rollback leva menos de 30 minutos.
- [ ] **Critérios de rollback automático configurados** — alertas no Azure Monitor para AUTO-01 a AUTO-05 ativos e apontando para a nova versão. Testados no ambiente de staging.
- [ ] **Monitoramento pós-release configurado** — dashboard `novatech-assistant-post-deploy-{deploy_id}` criado com as métricas relevantes para o tipo de mudança. Janela de 14 dias iniciada.
- [ ] **Feature flags corretas** — flags que devem estar habilitadas para esta versão estão `enabled`; flags de features não incluídas neste deploy permanecem no estado anterior.
- [ ] **Janela de deploy confirmada** — deploy agendado fora do horário de pico (09h–12h e 14h–17h). Time de atendimento notificado (para mudanças Alto e Crítico).
- [ ] **`production.yaml` atualizado** — configuração foi atualizada pelo pipeline de deploy após o deploy e commitada no repositório com referência ao `deploy_id`.

### Comunicação

- [ ] **Notificação ao atendente de origem preparada** — texto de notificação ao atendente que originou o feedback disponível para envio após o deploy.
- [ ] **Comunicado coletivo preparado** — para mudanças de nível Alto e Crítico: comunicado aprovado pela Coordenação de Atendimento pronto para envio após deploy.

---

*Documento elaborado com base no cenário completo do projeto NovaTech Assistant, nos guardrails formalizados, nas análises de inconsistências documentais, na revisão crítica de respostas pré-go-live e em todos os artefatos do harness de produto produzidos durante o discovery e a fase de estruturação. Pronto para revisão pelo Tech Lead, QA e Product Specialist antes do go-live.*
