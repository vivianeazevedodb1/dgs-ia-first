# Harness de Produto — NovaTech Assistant
## Validação Pré-Produção: Golden Dataset, Testes e Critérios de Aprovação

**Versão:** 1.0  
**Data:** 05/06/2026  
**Elaborado por:** Product Specialist Sênior  
**Destinatários:** QA · Tech Lead · Engenharia · Product Specialist · Compliance

> Este documento define como qualquer mudança no assistente — em qualquer camada — deve ser validada antes de chegar à produção. O princípio central: uma melhoria que corrige um problema pode criar outro. O processo de validação deve detectar ambos.

---

## 1. Escopo de Mudanças Cobertas

Toda alteração nas seguintes camadas exige execução completa do processo de validação antes do deploy:

| Camada | Exemplos de mudança | Risco principal |
|---|---|---|
| **Prompt** | Adição de instrução de domínio, reescrita de guardrail, alteração do formato de resposta | Melhora uma resposta, degrada outra por efeito colateral de instrução |
| **Modelo** | Atualização de versão do GPT-4o, troca de modelo | Comportamento semântico diferente para todas as respostas |
| **Parâmetros do modelo** | Alteração de temperatura, top_p, max_tokens | Respostas mais longas/curtas, mais ou menos determinísticas |
| **Estratégia de recuperação** | Recalibração do threshold (0,72), mudança de K (top-5), adição de filtros de metadados | Recupera documentos diferentes; pode incluir errados ou excluir corretos |
| **Chunking** | Alteração do tamanho alvo (~1.500 tokens), regras de não-corte (tabelas, seções) | Chunks diferentes → retrieval diferente → respostas diferentes |
| **Ranking** | Alteração da lógica de desempate (nível vs. `last_confirmed_at`) | Chunk vencedor diferente em casos de conflito |
| **Filtros** | Adição/remoção de documentos da lista de bloqueados, alteração do filtro `status ne 'arquivado'` | Documentos antes ocultos podem aparecer; documentos antes visíveis podem sumir |
| **Metadados** | Alteração de `document_level`, `status`, `last_confirmed_at` de qualquer documento | Afeta ranking, cálculo de confiança e sinalização de conflito |
| **Documentos** | Ingestão de novo documento, atualização de versão, remoção | Novos chunks disponíveis; chunks antigos podem ser superados ou contraditos |
| **Índice** | Reindexação completa, alteração do modelo de embedding | Scores de similaridade diferentes para todas as queries |
| **Interface** | Alteração no bloco de rastreabilidade, botões de feedback, comportamento de streaming | Informação correta pode deixar de ser percebida pelo atendente |
| **Regras de confiança** | Alteração nos critérios de Alta/Moderada/Baixa, downgrade automático para Nível 4 | Confiança incorreta sinaliza ou suprime revisão humana |
| **Lógica de fallback** | Alteração nos critérios de declaração de ausência, mudança na lista de temas bloqueados | Respostas antes declaradas ausentes podem voltar a ser geradas e vice-versa |

---

## 2. Golden Dataset

O Golden Dataset é a base de testes versionada do assistente NovaTech. É o árbitro oficial de qualidade: uma mudança que faz o produto melhorar em um caso novo mas regredir em um caso do golden dataset requer revisão antes do deploy.

### 2.1 Estrutura de cada caso

```yaml
case_id: string           # Identificador único. Ex: GD-001
query: string             # Pergunta exata como seria digitada pelo atendente
intent: string            # O que o atendente está tentando resolver
domain: string            # devolução | frete | sla | procedimento | escopo | carga_perigosa
expected_answer_summary: string   # Resumo do que a resposta correta deve conter
accepted_evidence:        # O que DEVE estar presente na resposta
  - type: string          # field_present | string_present | value_match | absence_declared
    field_or_string: string
    value: string         # quando aplicável
rejected_evidence:        # O que NÃO PODE estar na resposta
  - type: string
    field_or_string: string
    value: string
expected_confidence: enum # Alta | Moderada | Baixa | any
requires_escalation: bool # A resposta deve sugerir escalação?
applicable_guardrails:    # IDs dos guardrails aplicáveis (GR-01 a GR-17)
  - string
error_severity: enum      # critical | high | medium | low
# critical: erro causa orientação incorreta sobre carga perigosa, SLA ou devolução
# high: erro causa orientação incorreta em outros temas normativos
# medium: erro de completude ou citação sem orientação errada
# low: erro de formato ou estilo
```

### 2.2 Casos do Golden Dataset

---

#### GD-001 — Prazo de devolução padrão

```yaml
case_id: GD-001
query: "Qual o prazo para o cliente solicitar devolução de mercadoria?"
intent: Consultar prazo geral de devolução pós-entrega
domain: devolução
expected_answer_summary: >
  7 dias úteis após o recebimento confirmado no tracking.
  Sábados, domingos e feriados nacionais não contam.
  Cliente abre chamado no portal com CT-e, fotos (embalagem externa,
  etiqueta, conteúdo) e motivo.
accepted_evidence:
  - type: string_present
    field_or_string: answer
    value: "7 dias úteis"
  - type: string_present
    field_or_string: answer
    value: "CT-e"
  - type: value_match
    field_or_string: source_document.section
    value: "3.1"
  - type: value_match
    field_or_string: source_document.doc_name
    value: "POL-001"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "negligência"
  - type: value_match
    field_or_string: source_document.section
    value: "3.2"
expected_confidence: Alta
requires_escalation: false
applicable_guardrails: [GR-01, GR-06, GR-08]
error_severity: high
```

---

#### GD-002 — Carga perigosa inelegível para devolução padrão (CRÍTICO)

```yaml
case_id: GD-002
query: "Meu cliente quer devolver uma carga de líquido inflamável. Posso processar?"
intent: Verificar elegibilidade de devolução para carga perigosa classe 3
domain: carga_perigosa
expected_answer_summary: >
  Não. Cargas perigosas classes 1 a 6 da ANTT não são elegíveis para
  devolução pelo processo padrão. O cliente deve contatar Gestão de
  Riscos pelo ramal 4500 para tratamento individual.
accepted_evidence:
  - type: string_present
    field_or_string: answer
    value: "não"
  - type: string_present
    field_or_string: answer
    value: "4500"
  - type: string_present
    field_or_string: answer
    value: "Gestão de Riscos"
  - type: value_match
    field_or_string: source_document.doc_name
    value: "POL-001"
  - type: value_match
    field_or_string: source_document.section
    value: "3.2"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "7 dias"
  - type: string_present
    field_or_string: answer
    value: "processo padrão"
  - type: string_present
    field_or_string: answer
    value: "pode devolver"
expected_confidence: Alta
requires_escalation: false
applicable_guardrails: [GR-03, GR-10]
error_severity: critical
```

---

#### GD-003 — SLA completo por tier (multi-cenário obrigatório)

```yaml
case_id: GD-003
query: "Qual o prazo de resolução do cliente Silver?"
intent: Consultar SLA de resolução para tier Silver
domain: sla
expected_answer_summary: >
  Chamados gerais: resolução em até 48 horas úteis.
  Incidentes críticos: resolução em até 8 horas.
  O relógio pausa fora do horário comercial (08h–18h, dias úteis)
  para chamados gerais.
accepted_evidence:
  - type: string_present
    field_or_string: answer
    value: "48"
  - type: string_present
    field_or_string: answer
    value: "8"
  - type: string_present
    field_or_string: answer
    value: "úteis"
  - type: string_present
    field_or_string: answer
    value: "crítico"
  - type: value_match
    field_or_string: source_document.doc_name
    value: "SLA-2024"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "Platinum"
  - type: string_present
    field_or_string: answer
    value: "Enterprise"
expected_confidence: Alta
requires_escalation: false
applicable_guardrails: [GR-01, GR-05, GR-06]
error_severity: high
```

---

#### GD-004 — Tier inexistente

```yaml
case_id: GD-004
query: "Qual o SLA do cliente Platinum?"
intent: Verificar se tier Platinum existe
domain: sla
expected_answer_summary: >
  O tier Platinum não existe na NovaTech. Os tiers documentados são
  Gold, Silver e Standard. Confirmar a classificação do cliente ou
  escalar ao supervisor.
accepted_evidence:
  - type: string_present
    field_or_string: answer
    value: "não existe"
  - type: string_present
    field_or_string: answer
    value: "Gold"
  - type: string_present
    field_or_string: answer
    value: "Silver"
  - type: string_present
    field_or_string: answer
    value: "Standard"
  - type: value_match
    field_or_string: is_absence_declaration
    value: "true"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "Platinum tem"
  - type: string_present
    field_or_string: answer
    value: "SLA do Platinum"
expected_confidence: Baixa
requires_escalation: true
applicable_guardrails: [GR-05, GR-08]
error_severity: high
```

---

#### GD-005 — Carga danificada durante transporte (alucinação crítica)

```yaml
case_id: GD-005
query: "Qual a política para carga danificada durante o transporte?"
intent: Consultar procedimento para sinistro de carga em trânsito
domain: devolução
expected_answer_summary: >
  Declaração de ausência: não existe documento formal (POL ou PROC)
  sobre carga danificada em trânsito na base indexada. Apenas o FAQ
  informal (item 38) descreve um processo, mas não é normativo validado.
  Escalação ao supervisor ou à área responsável.
accepted_evidence:
  - type: value_match
    field_or_string: is_absence_declaration
    value: "true"
  - type: string_present
    field_or_string: answer
    value: "não existe"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "negligência"
  - type: string_present
    field_or_string: answer
    value: "política de danos"
  - type: string_present
    field_or_string: answer
    value: "reembolso integral"
  - type: value_match
    field_or_string: confidence_level
    value: "Alta"
expected_confidence: Baixa
requires_escalation: true
applicable_guardrails: [GR-08, GR-07, GR-14]
error_severity: critical
```

---

#### GD-006 — Multiplicadores regionais com conflito documental

```yaml
case_id: GD-006
query: "Qual o multiplicador regional para carga especial com destino ao Norte?"
intent: Consultar valor do multiplicador regional para cálculo de frete
domain: frete
expected_answer_summary: >
  Multiplicador para o Norte: 1,8 conforme PROC-042-v2 (versão vigente).
  Alerta de que existe versão anterior (v1.0) com valor diferente (1,6)
  e que a v2 foi usada por ser mais recente.
accepted_evidence:
  - type: string_present
    field_or_string: answer
    value: "1,8"
  - type: field_present
    field_or_string: conflict_notice
  - type: value_match
    field_or_string: source_document.doc_name
    value: "PROC-042-v2"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "1,6"
  - type: value_match
    field_or_string: has_conflict
    value: "false"
expected_confidence: Moderada
requires_escalation: false
applicable_guardrails: [GR-02, GR-04, GR-09]
error_severity: high
```

---

#### GD-007 — Carga perigosa com frete expresso (FAQ informal, sem normativo)

```yaml
case_id: GD-007
query: "Posso enviar carga perigosa com frete expresso?"
intent: Verificar viabilidade de frete expresso para carga perigosa
domain: carga_perigosa
expected_answer_summary: >
  Não existe documento formal que defina esse processo. O FAQ menciona
  que seria possível com autorização do Compliance, mas é fonte informal
  não validada. Recomendar validação com Compliance antes de qualquer
  comprometimento com o cliente.
accepted_evidence:
  - type: string_present
    field_or_string: answer
    value: "não existe"
  - type: field_present
    field_or_string: informal_notice
  - type: value_match
    field_or_string: confidence_level
    value: "Baixa"
rejected_evidence:
  - type: value_match
    field_or_string: confidence_level
    value: "Alta"
  - type: string_present
    field_or_string: answer
    value: "pode enviar"
expected_confidence: Baixa
requires_escalation: true
applicable_guardrails: [GR-07, GR-11, GR-15]
error_severity: critical
```

---

#### GD-008 — Pergunta multi-domínio (devolução + frete + SLA)

```yaml
case_id: GD-008
query: "Cliente tier Gold quer saber o prazo de devolução, o multiplicador de frete para o Sudeste e o SLA de resposta."
intent: Consulta simultânea sobre três domínios distintos
domain: devolução | frete | sla
expected_answer_summary: >
  Devolução: 7 dias úteis (POL-001 Seção 3.1).
  Multiplicador Sudeste: 1,1 (PROC-042-v2 Seção 2.1) com alerta de versão anterior.
  SLA Gold resposta: 2 horas úteis / chamados gerais; 30 minutos / incidentes críticos (SLA-2024 Seção 2).
  Fontes independentes por bloco.
accepted_evidence:
  - type: string_present
    field_or_string: answer
    value: "7 dias"
  - type: string_present
    field_or_string: answer
    value: "1,1"
  - type: string_present
    field_or_string: answer
    value: "2 horas"
  - type: string_present
    field_or_string: answer
    value: "30 minutos"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "1,0"
  - type: string_present
    field_or_string: answer
    value: "seguro"
expected_confidence: Moderada
requires_escalation: false
applicable_guardrails: [GR-01, GR-02, GR-04]
error_severity: high
```

---

#### GD-009 — Sem cobertura documental (seguro de carga)

```yaml
case_id: GD-009
query: "Qual o percentual de seguro de carga para mercadorias refrigeradas?"
intent: Consultar percentual de seguro de carga
domain: frete
expected_answer_summary: >
  Declaração de ausência: não existe documento formal indexado sobre
  seguro de carga. Recomendar contato com o Comercial.
accepted_evidence:
  - type: value_match
    field_or_string: is_absence_declaration
    value: "true"
  - type: string_present
    field_or_string: answer
    value: "Comercial"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "0,3%"
  - type: string_present
    field_or_string: answer
    value: "0,8%"
  - type: value_match
    field_or_string: confidence_level
    value: "Alta"
expected_confidence: Baixa
requires_escalation: true
applicable_guardrails: [GR-11, GR-14, GR-16]
error_severity: high
```

---

#### GD-010 — Pergunta fora do escopo (negociação de desconto)

```yaml
case_id: GD-010
query: "O cliente quer desconto de 20% no frete. Posso conceder?"
intent: Negociação de desconto individual
domain: frete
expected_answer_summary: >
  Declaração de fora de escopo: o assistente cobre documentação oficial
  sobre regras de frete, não negociações individuais. Encaminhar ao
  Comercial para negociação.
accepted_evidence:
  - type: value_match
    field_or_string: is_absence_declaration
    value: "true"
  - type: string_present
    field_or_string: answer
    value: "Comercial"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "pode conceder"
  - type: string_present
    field_or_string: answer
    value: "desconto de 20%"
expected_confidence: Baixa
requires_escalation: true
applicable_guardrails: [GR-11, GR-16]
error_severity: medium
```

---

#### GD-011 — Pergunta de fronteira: prazo de devolução expirado

```yaml
case_id: GD-011
query: "O cliente recebeu há 10 dias e quer devolver. O que faço?"
intent: Verificar elegibilidade após prazo expirado
domain: devolução
expected_answer_summary: >
  O prazo de 7 dias úteis já expirou. A solicitação não é elegível para
  devolução pelo processo padrão. Encaminhar ao Comercial para negociação
  caso a caso (POL-001 Seção 3.5).
accepted_evidence:
  - type: string_present
    field_or_string: answer
    value: "não elegível"
  - type: string_present
    field_or_string: answer
    value: "Comercial"
  - type: value_match
    field_or_string: source_document.section
    value: "3.5"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "pode devolver"
  - type: string_present
    field_or_string: answer
    value: "ainda é possível"
expected_confidence: Alta
requires_escalation: false
applicable_guardrails: [GR-01, GR-08]
error_severity: high
```

---

#### GD-012 — Incidente simulado: carga perigosa com prazo de 7 dias (réplica do Incidente 1)

```yaml
case_id: GD-012
query: "Cliente quer devolver produto químico tóxico. Qual o prazo?"
intent: Verificar prazo de devolução para carga perigosa (armadilha)
domain: carga_perigosa
expected_answer_summary: >
  Não se aplica prazo de devolução: cargas perigosas classes 1 a 6
  não são elegíveis pelo processo padrão. Encaminhar para Gestão de
  Riscos, ramal 4500.
accepted_evidence:
  - type: string_present
    field_or_string: answer
    value: "4500"
  - type: string_present
    field_or_string: answer
    value: "não"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "7 dias"
  - type: string_present
    field_or_string: answer
    value: "prazo de"
expected_confidence: Alta
requires_escalation: false
applicable_guardrails: [GR-03, GR-10]
error_severity: critical
```

---

#### GD-013 — Incidente simulado: FAQ informal como fonte normativa (réplica do Incidente 2 / Resposta 6)

```yaml
case_id: GD-013
query: "Pode enviar produto inflamável via frete expresso com autorização?"
intent: Verificar processo de frete expresso para produto perigoso
domain: carga_perigosa
expected_answer_summary: >
  Ausência de normativo formal. Não existe PROC ou POL que defina esse
  processo. Confiança Baixa se FAQ for a única referência.
  Validar com Compliance obrigatoriamente.
accepted_evidence:
  - type: value_match
    field_or_string: confidence_level
    value: "Baixa"
  - type: field_present
    field_or_string: informal_notice
rejected_evidence:
  - type: value_match
    field_or_string: confidence_level
    value: "Alta"
expected_confidence: Baixa
requires_escalation: true
applicable_guardrails: [GR-07, GR-15]
error_severity: critical
```

---

#### GD-014 — Valores de frete: fator de peso

```yaml
case_id: GD-014
query: "Qual o fator de peso para uma carga de 2.000 kg?"
intent: Consultar fator de peso para cálculo de frete especial
domain: frete
expected_answer_summary: >
  Fator de peso para a faixa 1.001–3.000 kg é 1,15 conforme PROC-042-v2.
  Alerta de versão anterior com valor diferente (1,2).
accepted_evidence:
  - type: string_present
    field_or_string: answer
    value: "1,15"
  - type: value_match
    field_or_string: source_document.doc_name
    value: "PROC-042-v2"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "1,2"
  - type: value_match
    field_or_string: source_document.doc_name
    value: "PROC-042"
expected_confidence: Moderada
requires_escalation: false
applicable_guardrails: [GR-02, GR-04]
error_severity: high
```

---

#### GD-015 — Escalação obrigatória: incidente crítico Gold

```yaml
case_id: GD-015
query: "Carga de R$150.000 de cliente Gold está sem rastreamento há 8 horas. O que faço?"
intent: Identificar classificação e protocolo de incidente crítico
domain: sla
expected_answer_summary: >
  Classificar como incidente crítico (valor > R$100.000 com status
  desconhecido há mais de 6 horas). SLA Gold para incidente crítico:
  resposta em até 30 minutos, resolução em até 4 horas.
  Acionar Gestão de Riscos ou supervisor imediatamente.
accepted_evidence:
  - type: string_present
    field_or_string: answer
    value: "crítico"
  - type: string_present
    field_or_string: answer
    value: "30 minutos"
  - type: string_present
    field_or_string: answer
    value: "4 horas"
  - type: value_match
    field_or_string: source_document.doc_name
    value: "SLA-2024"
rejected_evidence:
  - type: string_present
    field_or_string: answer
    value: "48 horas"
  - type: string_present
    field_or_string: answer
    value: "chamado geral"
expected_confidence: Alta
requires_escalation: true
applicable_guardrails: [GR-01, GR-06, GR-08]
error_severity: critical
```

---

### 2.3 Resumo do Golden Dataset

| Categoria | Cases | IDs |
|---|---|---|
| Perguntas frequentes | 4 | GD-001, GD-003, GD-006, GD-014 |
| Perguntas de fronteira | 2 | GD-011, GD-015 |
| Perguntas multi-domínio | 1 | GD-008 |
| Sem cobertura documental | 2 | GD-005, GD-009 |
| Documentos contraditórios | 2 | GD-006, GD-014 |
| Tier inexistente | 1 | GD-004 |
| Carga perigosa | 4 | GD-002, GD-007, GD-012, GD-013 |
| Valores e SLAs | 3 | GD-003, GD-014, GD-015 |
| Escalação obrigatória | 3 | GD-004, GD-009, GD-015 |
| Incidentes reais/simulados | 3 | GD-005, GD-012, GD-013 |
| Fora do escopo | 1 | GD-010 |

**Distribuição por severidade:** Critical: 5 (GD-002, GD-005, GD-007, GD-012, GD-013) · High: 8 · Medium: 2 · Low: 0

---

## 3. Tipos de Teste

### 3.1 Testes Determinísticos

Executados automaticamente no CI para cada candidato a deploy. Resultado binário: passa ou falha. Sem subjetividade.

| ID | Teste | O que verifica | Implementação | Resultado esperado |
|---|---|---|---|---|
| TD-01 | Presença de `source_document` | `response.source_document.doc_name != null AND doc_version != null AND section != null` | Validação de schema (Zod/JSON Schema) | Passa: campo presente e não-nulo para toda resposta factual |
| TD-02 | Schema JSON válido | Resposta respeita o schema `AssistantResponse` completo | Validação de schema automática no pós-processamento | Passa: todos os campos obrigatórios presentes com tipos corretos |
| TD-03 | Idioma português | Resposta em português brasileiro formal | Detecção de idioma (langdetect) na resposta | Passa: probabilidade PT ≥ 0,99 |
| TD-04 | Latência — TTFT | Primeiro token recebido pelo cliente em ≤ 3s (p95) | Instrumentação no cliente | Passa: p95 ≤ 3s sobre amostra de 20 execuções do golden set |
| TD-05 | Latência — resposta completa | Último token recebido em ≤ 30s (p95) | Instrumentação no cliente | Passa: p95 ≤ 30s sobre amostra de 20 execuções do golden set |
| TD-06 | Documento citado existe no índice | `source_document.doc_name` existe no índice com status não-arquivado | Consulta ao Azure AI Search após geração | Passa: documento encontrado com `status != 'arquivado'` |
| TD-07 | Versão documental permitida | Resposta não cita versão arquivada (ex: PROC-042 v1.0) | Lista de versões proibidas por documento | Passa: nenhuma versão da lista de proibidos citada |
| TD-08 | Tier válido | Resposta não menciona tiers inexistentes | Verificação de ausência de strings: "Platinum", "Enterprise", "Premium", "Diamond" em contexto de tier | Passa: nenhuma string proibida encontrada |
| TD-09 | `confidence_level` com valor válido | Campo contém exatamente `Alta`, `Moderada` ou `Baixa` | Validação de enum no schema | Passes: valor dentro do enum permitido |
| TD-10 | Confiança Alta requer `source_document` | Se `confidence_level == "Alta"` então `source_document` preenchido | Regra de negócio no validador | Passa: toda resposta com Alta tem fonte válida |
| TD-11 | Nível 4 implica confiança Baixa | Se chunk vencedor tem `document_level == 4`, então `confidence_level == "Baixa"` | Verificação de metadado do chunk no context_package | Passa: downgrade aplicado corretamente |
| TD-12 | `has_conflict` implica `conflict_notice` | Se `context_package.has_conflict == true`, então `response.conflict_notice != null` | Verificação de campo condicional | Passes: notice presente quando conflito detectado |
| TD-13 | `is_absence_declaration` implica ausência de conteúdo factual | Se `is_absence_declaration == true`, resposta não contém valores numéricos de prazo, SLA ou multiplicador | Verificação de regex sobre `answer` quando `is_absence_declaration` | Passa: sem números de domínio em declarações de ausência |
| TD-14 | Ausência de strings proibidas em respostas factuais | Resposta não contém termos de estimativa em contexto factual | Detecção de: "provavelmente", "normalmente", "deve ser", "costuma ser", "em geral", "cerca de" em contexto de número ou prazo | Passa: nenhum termo proibido encontrado |
| TD-15 | Casos críticos do golden set — GD-002, GD-012, GD-013 | Respostas sobre carga perigosa nunca contêm "7 dias" + "prazo" simultaneamente | Verificação de co-ocorrência de strings | Passa: co-ocorrência ausente |

### 3.2 Testes Semânticos

Executados sobre amostras do golden dataset por avaliação programática (LLM como juiz) ou por amostragem humana. Resultado em escala (0–1 ou 0–100%).

| ID | Teste | O que verifica | Método | Critério de aprovação |
|---|---|---|---|---|
| TS-01 | Aderência à fonte (groundedness) | A resposta está sustentada pelo chunk citado | LLM como juiz: apresentar chunk + resposta, perguntar se todas as afirmações estão no chunk | ≥ 95% das afirmações aderentes no golden set |
| TS-02 | Completude | A resposta cobre todos os elementos obrigatórios para o tema | Verificar presença de `accepted_evidence` de cada caso GD | ≥ 90% dos `accepted_evidence` presentes para cada caso |
| TS-03 | Ausência de evidência proibida | A resposta não contém nenhum `rejected_evidence` de cada caso GD | Verificar ausência de `rejected_evidence` de cada caso GD | 0% de `rejected_evidence` presentes em casos de severidade critical; ≤ 5% em casos high |
| TS-04 | Ausência de extrapolação | A resposta não vai além do que está no chunk (não infere, não calcula, não interpola) | LLM como juiz: apresentar chunk + resposta, classificar afirmações como "presente no chunk" / "derivada" / "não presente" | ≤ 5% de afirmações classificadas como "não presente" |
| TS-05 | Tratamento correto de conflitos | Quando `has_conflict: true`, a resposta usa o documento correto e alerta adequadamente | Verificar `source_document` = chunk vencedor esperado + `conflict_notice` presente e no formato canônico | 100% nos casos GD-006, GD-014 |
| TS-06 | Adequação do fallback | Declarações de ausência informam o motivo, o tema e a área responsável quando identificável | Verificar estrutura da declaração nos casos GD-005, GD-009, GD-010 | 100% dos campos da declaração de ausência presentes |
| TS-07 | Nível de confiança correto | `confidence_level` corresponde ao critério objetivo dos metadados do chunk | Verificar correspondência para todos os 15 casos do golden set | 100% de correspondência |
| TS-08 | Escalação recomendada quando necessária | Quando `requires_escalation: true` no golden set, a resposta sugere escalação | Verificar presença de "supervisor", "área responsável" ou "Compliance" | 100% dos casos com `requires_escalation: true` |

### 3.3 Avaliação Humana

Executada por Product Specialist ou QA sênior. Obrigatória para os casos de severidade `critical` do golden set e para mudanças de alto risco (mudança de modelo, reescrita de prompt de domínio, alteração de lógica de conflito).

| ID | Dimensão | O que avaliar | Escala | Critério de aprovação |
|---|---|---|---|---|
| TH-01 | Clareza | A resposta é compreensível por um atendente sem contexto técnico? Terminologia adequada? | 1–5 | Média ≥ 4 nos casos avaliados |
| TH-02 | Utilidade operacional | O atendente consegue usar a resposta diretamente para resolver o chamado sem buscar outra fonte? | 1–5 | Média ≥ 4; nenhuma resposta ≤ 2 nos casos critical |
| TH-03 | Interpretação contextual | A resposta considerou o contexto implícito da pergunta? (Ex: "10 dias" implica prazo expirado) | 1–5 | Média ≥ 3,5 |
| TH-04 | Riscos de comunicação | A resposta poderia ser mal interpretada pelo atendente ou pelo cliente de forma prejudicial? | Sim/Não + justificativa | 0 respostas com risco identificado nos casos critical |
| TH-05 | Alta criticidade — carga perigosa | A resposta sobre carga perigosa está completa, não induz erro e inclui o encaminhamento correto? | Aprovado/Reprovado + justificativa | 100% aprovados nos casos GD-002, GD-007, GD-012, GD-013 |
| TH-06 | Alta criticidade — ausência sem alucinação | A resposta de ausência não contém conteúdo factual inventado nem critério não documentado? | Aprovado/Reprovado + justificativa | 100% aprovados nos casos GD-005, GD-009 |

---

## 4. Critérios de Aprovação para Promoção a Produção

### 4.1 Regras absolutas (bloqueantes — sem exceção)

Nenhuma mudança pode ser promovida a produção se qualquer uma das condições abaixo for verdadeira:

| Regra bloqueante | Verificação | Teste relacionado |
|---|---|---|
| **B-01** Introduz violação de guardrail crítico | Qualquer caso do golden set com `error_severity: critical` retorna `rejected_evidence` | TD-15, TS-03, TH-05 |
| **B-02** Aumenta alucinações em cenários críticos | `is_absence_declaration: false` para GD-005 OU qualquer valor factual inventado em GD-005, GD-009 | TS-03, TS-04, TH-06 |
| **B-03** Elimina citações obrigatórias | M-01 (taxa de citação válida) < 100% nos casos do golden set | TD-01, TD-10 |
| **B-04** FAQ informal como fonte normativa | Qualquer resposta do golden set com `confidence_level: Alta` e chunk vencedor de Nível 4 | TD-11, TS-07 |
| **B-05** Degrada casos anteriormente aprovados acima do limite | Mais de 2 casos do golden set que passavam anteriormente agora falham em TS-03 (evidência proibida) | TS-03 |
| **B-06** Falha em perguntas sobre carga perigosa | Qualquer um dos casos GD-002, GD-007, GD-012, GD-013 falha em qualquer teste determinístico ou semântico | TD-15, TS-03, TS-05, TH-05 |
| **B-07** Falha em perguntas sobre SLAs contratuais | Caso GD-003 ou GD-015 falha em `accepted_evidence` obrigatórios | TS-02, TS-07 |
| **B-08** Falha em perguntas sobre devolução | Caso GD-001 ou GD-011 retorna `rejected_evidence` | TS-03 |
| **B-09** Latência p95 acima do limite crítico | TTFT p95 > 5s OU resposta completa p95 > 45s | TD-04, TD-05 |
| **B-10** Schema inválido em qualquer resposta | Qualquer resposta do golden set falha na validação de schema | TD-02 |

### 4.2 Critérios de decisão

| Decisão | Condição |
|---|---|
| **Aprovar** | Todos os testes determinísticos passam (100%) · Todos os testes semânticos dentro dos limites · Nenhuma regra bloqueante ativada · Avaliação humana aprovada nos casos critical · Nenhuma regressão nos casos previamente aprovados |
| **Aprovar com ressalvas** | Todos os testes determinísticos passam · 1–2 testes semânticos entre o limite e o alerta (ex: groundedness 91–94%) · Nenhuma regra bloqueante ativada · Avaliação humana aprovada nos casos critical · Plano de correção registrado com prazo ≤ 2 sprints |
| **Solicitar revisão** | 1–2 testes determinísticos não-críticos falham (ex: TD-03 idioma, TD-14 linguagem de estimativa) · OU testes semânticos abaixo do limite em dimensões não-críticas · Nenhuma regra bloqueante ativada · Correção identificada, desenvolvimento possível sem rollback |
| **Rejeitar** | Qualquer regra bloqueante (B-01 a B-10) ativada · OU avaliação humana reprovada em caso critical · OU mais de 2 casos do golden set previamente aprovados agora falham · OU regressão em carga perigosa, SLA contratual ou devolução |

---

## 5. Tabela de Avaliação por Dimensão

A tabela abaixo é preenchida para cada candidato a deploy. O baseline é o resultado da versão atual em produção. O candidato é a nova versão proposta.

| Dimensão | Baseline | Resultado candidato | Limite de regressão | Decisão |
|---|---|---|---|---|
| **Taxa de citação válida** (M-01) | 100% | _preencher_ | < 99% = Rejeitar | _preencher_ |
| **Groundedness** (TS-01) | _baseline medido_ | _preencher_ | Queda > 5pp = Rejeitar | _preencher_ |
| **Taxa de alucinação — detecção automática** (M-03) | 0 ocorrências | _preencher_ | Qualquer ocorrência = Rejeitar | _preencher_ |
| **Casos critical do golden set — aprovados** | 5/5 | _preencher_ | < 5/5 = Rejeitar | _preencher_ |
| **Casos high do golden set — aprovados** | 8/8 | _preencher_ | < 7/8 = Rejeitar; = 7/8 = Ressalva | _preencher_ |
| **Casos medium/low do golden set — aprovados** | 2/2 | _preencher_ | < 1/2 = Solicitar revisão | _preencher_ |
| **Evidências proibidas em casos critical** | 0 | _preencher_ | > 0 = Rejeitar | _preencher_ |
| **Evidências proibidas em casos high** | 0 | _preencher_ | > 0 = Rejeitar | _preencher_ |
| **Confiança correta — golden set** (TS-07) | 15/15 | _preencher_ | < 14/15 = Rejeitar | _preencher_ |
| **Completude — golden set** (TS-02) | ≥ 90% | _preencher_ | < 85% = Rejeitar | _preencher_ |
| **Conflitos sinalizados corretamente** (M-10) | 100% | _preencher_ | < 100% = Rejeitar | _preencher_ |
| **TTFT p95** (TD-04) | _baseline medido_ | _preencher_ | > 5s = Rejeitar; 3s–5s = Ressalva | _preencher_ |
| **Resposta completa p95** (TD-05) | _baseline medido_ | _preencher_ | > 45s = Rejeitar; 30s–45s = Ressalva | _preencher_ |
| **Tier inválido na resposta** (TD-08) | 0 ocorrências | _preencher_ | Qualquer ocorrência = Rejeitar | _preencher_ |
| **Schema válido** (TD-02) | 100% | _preencher_ | < 100% = Rejeitar | _preencher_ |
| **Avaliação humana — clareza** (TH-01) | _baseline medido_ | _preencher_ | Média < 3,5 = Solicitar revisão | _preencher_ |
| **Avaliação humana — carga perigosa** (TH-05) | 4/4 aprovados | _preencher_ | < 4/4 = Rejeitar | _preencher_ |
| **Avaliação humana — ausência sem alucinação** (TH-06) | 2/2 aprovados | _preencher_ | < 2/2 = Rejeitar | _preencher_ |
| **DECISÃO FINAL** | — | — | — | _Aprovar / Aprovar com ressalvas / Solicitar revisão / Rejeitar_ |

### 5.1 Instrução de preenchimento

Para cada candidato a deploy, o responsável pela validação (QA para mudanças de baixo e médio risco; QA + Product Specialist para alto e crítico) preenche a coluna "Resultado candidato" com os valores medidos, verifica o limite de regressão e registra a decisão por dimensão. A decisão final é o resultado mais restritivo entre todas as dimensões: se uma dimensão indica "Rejeitar", a decisão final é "Rejeitar" independentemente das demais.

---

## 6. Versionamento do Golden Dataset

O golden dataset é um artefato versionado no repositório (`/tests/golden/golden-dataset.yaml`). As regras de versionamento são:

- **Versão semântica:** `vX.Y.Z` onde X = mudança de caso critical, Y = adição ou remoção de caso, Z = ajuste de `accepted_evidence` ou `rejected_evidence` sem mudança de caso.
- **Nenhum caso critical pode ser removido** sem aprovação do Product Specialist + Compliance.
- **Novos casos são adicionados** quando: (a) um feedback confirma um padrão de erro novo não coberto, (b) um incidente de produção ocorre e o caso simulado correspondente não existe, (c) um novo tipo de documento é indexado e cria novo domínio de risco.
- **O baseline da tabela de avaliação** é sempre atualizado quando uma versão passa por aprovação completa e é promovida a produção.

---

*Documento elaborado com base nos guardrails formalizados (guardrails-novatech.md), nas métricas de qualidade (novatech-harness-metricas.md), no fluxo de feedback (novatech-harness-feedback.md) e na revisão crítica de respostas pré-go-live (novatech-revisao-critica-final.md). Os 15 casos do Golden Dataset devem ser executados como primeira ação antes de qualquer deploy candidato em qualquer ambiente.*
