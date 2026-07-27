# Revisão Crítica das Respostas do Assistente
## NovaTech Assistant · Avaliação Pré-Go-Live

**Data:** 05/06/2026  
**Elaborado por:** Product Specialist Sênior  
**Fonte de verdade:** Anexo A — Documentação Simulada da NovaTech  
**Respostas avaliadas:** 6  
**Método:** Avaliação principal + segunda avaliação independente + consolidação

---

## Critérios de Avaliação

### Classificação

Cada resposta recebe exatamente uma das três classificações:

- **Correta** — conteúdo factual correto, fonte adequada, seção verificável, confiança compatível com a qualidade da evidência.
- **Parcialmente correta** — conteúdo central correto, mas com omissão de contexto relevante, citação imprecisa ou confiança ligeiramente inadequada. O erro não compromete a orientação principal ao atendente.
- **Incorreta** — conteúdo factual errado ou não sustentado pela documentação, fonte ausente ou inadequada para a afirmação, ou confiança incompatível com a qualidade da evidência de forma que induza erro operacional.

### Princípios aplicados

Fatos e problemas de citação são avaliados separadamente: uma seção errada com conteúdo correto não resulta em "Incorreta", mas é registrada como observação de rastreabilidade. Uma afirmação factualmente incorreta ou sem suporte documental é avaliada como erro de conteúdo, independentemente da forma de citação.

Fontes formais (POL-001, PROC-042-v2, SLA-2024) têm precedência absoluta sobre fontes informais (FAQ-Atendimento). O FAQ-Atendimento é classificado no próprio Anexo A como "Documento informal — NÃO validado por Compliance ou Operações."

Ausência de normativo formal é declarada explicitamente quando nenhum documento POL, PROC ou SLA indexado cobre o tema. O Anexo A documenta os gaps conhecidos: política de carga danificada, seguro de carga e frete padrão abaixo de 500 kg não possuem normativo formal.

### Guardrails aplicados

**DEVE:** toda afirmação factual sustentada por documentação indexada; toda resposta cita o documento; prazos, valores e SLAs correspondem literalmente à documentação; confiança reflete a qualidade e autoridade da fonte.

**NÃO DEVE:** inventar políticas, prazos ou procedimentos; tratar FAQ informal como documento normativo; inventar tiers além de Gold, Silver e Standard; afirmar que cargas perigosas ANTT classes 1–6 são elegíveis para devolução pelo processo padrão; ocultar ausência, conflito ou fragilidade da evidência.

**QUANDO EM DÚVIDA:** informar que não há evidência formal suficiente; reduzir o nível de confiança; sugerir escalação; informar divergência entre versões e priorizar a mais recente.

---

## Avaliação Principal

### Amostra avaliada

| # | Pergunta | Resposta | Fonte citada | Confiança |
|---|---|---|---|---|
| 1 | Qual o prazo de devolução para produtos standard? | O prazo é de 7 dias úteis após o recebimento. O cliente deve abrir chamado no portal e anexar fotos. | POL-001, seção 3.2 | Alta |
| 2 | Meu cliente é Silver. Qual o prazo de resolução? | O cliente Silver tem resolução em até 48h. | SLA-2024 | Alta |
| 3 | Posso devolver carga perigosa classe 3? | Não. Cargas perigosas, classes 1 a 6 da ANTT, não podem ser devolvidas pelo processo padrão. Recomendo escalar para o supervisor. | POL-001, seção 3.2 | Alta |
| 4 | Qual a política para carga danificada durante transporte? | A política de danos prevê reembolso integral quando comprovada negligência da transportadora, mediante laudo técnico e fotos. | Nenhuma | Alta |
| 5 | Qual o SLA do cliente Enterprise? | O tier Enterprise não foi encontrado na documentação. Os tiers documentados são Gold, Silver e Standard. Sugiro confirmar a classificação ou escalar ao supervisor. | — | Baixa |
| 6 | Posso enviar carga perigosa com frete expresso? | Sim, cargas perigosas podem ser enviadas via frete expresso mediante autorização prévia do Compliance e documentação ANTT atualizada. | FAQ-Atendimento, item 32 | Alta |

---

### Resposta 1 — Prazo de devolução para produtos standard

**Classificação: Correta**

O prazo de 7 dias úteis está correto e corresponde literalmente à POL-001 Seção 3.1. O procedimento de abrir chamado no portal e anexar fotos está correto e alinhado com a POL-001 Seção 3.3.

**Observação de citação (não altera a classificação):** A resposta cita a Seção 3.2, que trata das exceções ao prazo geral. A informação correta está na Seção 3.1. O erro é de rastreabilidade — o conteúdo factual está correto e o atendente não recebe orientação errada. A resposta é adequada para uso no atendimento.

**Observação de completude (não altera a classificação):** A POL-001 Seção 3.3 exige também o número do CT-e e especifica os tipos de foto (embalagem externa, etiqueta e conteúdo). A omissão é real, mas não compromete a orientação principal. O atendente consegue iniciar o processo com as informações fornecidas.

---

### Resposta 2 — Prazo de resolução cliente Silver

**Classificação: Correta**

O valor de 48h está correto. A SLA-2024 Seção 2 confirma: tempo de resolução para chamados gerais do tier Silver é de até 48 horas úteis. A fonte está correta.

**Observação de completude (não altera a classificação):** A resposta cobre o cenário mais frequente (chamados gerais). O SLA para incidentes críticos do tier Silver é de 8 horas (SLA-2024 Seção 2) e não foi mencionado. A omissão é real e torna a resposta incompleta para o universo completo da pergunta, mas não torna a informação fornecida incorreta. Para chamados gerais — que representam a maioria dos casos — a resposta está correta. A ausência de "horas úteis" como qualificador (SLA-2024 Seção 5) é observação adicional de precisão.

A resposta é adequada para uso no atendimento com a ressalva de que o atendente deve consultar o SLA de incidentes críticos em situações urgentes.

---

### Resposta 3 — Devolução de carga perigosa classe 3

**Classificação: Correta**

A POL-001 Seção 3.2 confirma explicitamente que cargas perigosas classes 1 a 6 da ANTT não são elegíveis para devolução pelo processo padrão. A classe 3 (líquidos inflamáveis) está incluída nessa lista. Fonte e seção estão corretas. Confiança Alta é adequada para normativo vigente.

**Observação de encaminhamento (não altera a classificação):** A POL-001 Seção 3.2 indica que o cliente deve contatar o setor de Gestão de Riscos pelo ramal 4500. A resposta recomenda escalar para o supervisor, que é encaminhamento interno — não o contato direto ao cliente previsto no documento. A orientação não está errada (escalar para o supervisor é uma ação válida), mas está incompleta em relação ao que o documento prevê. O atendente consegue resolver com ajuda do supervisor; a resposta não causa desorientação.

---

### Resposta 4 — Política para carga danificada durante transporte

**Classificação: Incorreta**

Esta resposta é uma alucinação: apresenta como política oficial um conteúdo que não existe em nenhum documento formal indexado.

O Anexo A documenta explicitamente nos "Gaps identificados": "Política de carga danificada: o FAQ Item 38 descreve um processo, mas não existe documento formal (POL ou PROC) sobre tratamento de carga danificada em trânsito."

O critério "negligência da transportadora" não aparece em nenhum documento do Anexo A. O FAQ item 38 utiliza "responsabilidade comprovada" — semanticamente diferente de negligência, que implica culpa qualificada. A resposta inventou um critério jurídico sem sustentação documental.

A resposta não cita nenhuma fonte e declara confiança Alta. Confiança Alta sem nenhuma fonte é violação direta dos guardrails: "toda afirmação factual deve estar sustentada por documentação indexada" e "a confiança deve refletir a qualidade e autoridade da fonte."

O comportamento correto para este tema é declarar ausência de normativo formal, indicar que a única referência disponível é o FAQ informal (com baixa confiança) e recomendar escalada ao supervisor ou à área responsável.

---

### Resposta 5 — SLA do cliente Enterprise

**Classificação: Correta**

A SLA-2024 Seção 1 declara textualmente que "não existem outros tiers além dos três listados acima." O tier Enterprise não está na documentação. A resposta não inventou SLA, não confundiu com outro tier, listou os três tiers válidos, declarou ausência e recomendou escalada. Confiança Baixa é adequada.

**Observação de rastreabilidade (não altera a classificação):** A afirmação sobre os três tiers poderia citar SLA-2024 Seção 1 para rastreabilidade completa. A ausência da citação é lacuna menor.

---

### Resposta 6 — Carga perigosa com frete expresso

**Classificação: Incorreta**

A resposta usa o FAQ-Atendimento item 32 como única fonte e declara confiança Alta. Dois problemas distintos:

**Ausência de normativo formal:** O Anexo A documenta na contradição número 4: "FAQ Item 32 vs documentação formal: o FAQ diz que carga perigosa pode ser enviada com frete expresso 'com autorização', mas não existe documento formal (PROC ou POL) que defina esse processo. A informação pode ser prática informal não documentada." Não existe nenhuma POL ou PROC indexada que autorize, defina critérios ou regule envio de carga perigosa por frete expresso.

**Confiança Alta inadequada para FAQ informal:** O FAQ-Atendimento é classificado no Anexo A como "Documento informal — NÃO validado por Compliance ou Operações." Confiança Alta para fonte informal viola o guardrail: "a confiança deve refletir a qualidade e autoridade da fonte." A confiança adequada para resposta baseada exclusivamente em FAQ é Baixa, acompanhada de aviso de fonte informal e recomendação de validação com Compliance ou área responsável.

O comportamento correto é: informar que não existe evidência normativa formal suficiente para o tema, citar o FAQ como referência informal com confiança Baixa, e sugerir validação com o Compliance antes de qualquer comprometimento com o cliente.

---

## Avaliação do Segundo Avaliador

*Avaliação realizada de forma independente, sem consulta à avaliação principal.*

### Resposta 1

O prazo de 7 dias úteis está correto e a POL-001 é a fonte adequada. O procedimento descrito (portal + fotos) está alinhado com a Seção 3.3. A seção 3.2 citada trata de exceções — a seção correta é 3.1. Erro de rastreabilidade, não de conteúdo. O atendente recebe orientação correta e consegue conduzir o chamado. CT-e e tipos de foto são detalhes omitidos, mas a resposta é funcional para o atendimento.

**Segunda classificação: Correta**

### Resposta 2

O valor 48h está correto para chamados gerais do tier Silver (SLA-2024 Seção 2). A fonte é adequada. O SLA para incidentes críticos (8h) não foi coberto. A omissão é real, mas a pergunta não especificou o tipo de chamado — a resposta cobriu o caso mais comum. Para uso no atendimento rotineiro, a resposta está correta.

**Segunda classificação: Correta**

### Resposta 3

A POL-001 Seção 3.2 sustenta a resposta: cargas perigosas classes 1–6 não são elegíveis para devolução pelo processo padrão, e a classe 3 está na lista. Fonte e seção corretas, confiança Alta adequada. O encaminhamento ao supervisor não é o procedimento completo — o documento prevê o ramal 4500 da Gestão de Riscos para o cliente —, mas não é uma orientação errada. O supervisor repassará a informação.

**Segunda classificação: Correta**

### Resposta 4

Nenhum documento do Anexo A sustenta a afirmação de uma "política de danos." O gap está explicitamente documentado: não existe POL ou PROC sobre carga danificada em trânsito. O critério "negligência da transportadora" não aparece em nenhum documento, incluindo o FAQ item 38 que usa "responsabilidade comprovada." A resposta não cita nenhuma fonte e declara confiança Alta. Alucinação com confiança Alta sem fonte.

**Segunda classificação: Incorreta**

### Resposta 5

SLA-2024 Seção 1 confirma que os únicos tiers são Gold, Silver e Standard. A resposta não inventou SLA nem confundiu tiers. Ausência declarada corretamente, confiança Baixa adequada. Resposta correta dentro dos guardrails.

**Segunda classificação: Correta**

### Resposta 6

O FAQ item 32 é a única fonte disponível e é documento informal não validado. O Anexo A documenta que não existe normativo formal para esse processo. Confiança Alta é incompatível com fonte informal. A resposta deveria declarar ausência de evidência normativa e sugerir validação com Compliance.

**Segunda classificação: Incorreta**

---

## Comparação entre as Avaliações

### Tabela de concordância

| ID | Avaliação principal | Segunda avaliação | Concordância | Observação |
|---|---|---|---|---|
| 1 | Correta | Correta | ✅ Sim | Ambas identificaram seção 3.2 como observação de rastreabilidade, não como erro que compromete a classificação. Ambas reconhecem CT-e e tipos de foto como omissões secundárias. |
| 2 | Correta | Correta | ✅ Sim | Ambas reconhecem que 48h está correto para chamados gerais e que a omissão do SLA crítico não torna a resposta incorreta para o uso principal. |
| 3 | Correta | Correta | ✅ Sim | Ambas reconhecem que a parte central está correta e que o encaminhamento pelo supervisor, embora menos preciso que o ramal 4500, não constitui erro que compromete a orientação. |
| 4 | Incorreta | Incorreta | ✅ Sim | Concordância total: alucinação por ausência de normativo formal, critério inventado ("negligência"), confiança Alta sem fonte. |
| 5 | Correta | Correta | ✅ Sim | Concordância total. |
| 6 | Incorreta | Incorreta | ✅ Sim | Concordância total: FAQ informal como fonte para tema sem normativo, confiança Alta inadequada. |

### Concordâncias reais

Em todas as seis respostas, as classificações finais coincidem. As duas avaliações chegam ao mesmo resultado sobre quais respostas são adequadas (1, 2, 3, 5) e quais são problemáticas (4, 6).

Em relação às respostas 1, 2 e 3, ambas as avaliações adotaram o mesmo critério: observações de completude e citação são registradas, mas não elevam a classificação para "Parcialmente correta" ou "Incorreta" quando o conteúdo central está correto e o atendente consegue conduzir o chamado adequadamente.

### Divergências

Não há divergência de classificação. Há diferença de ênfase nas observações secundárias: a primeira avaliação detalhou mais as omissões de completude nas respostas 1, 2 e 3; a segunda foi mais concisa nas observações e focou nos problemas que efetivamente impedem o uso da resposta.

### Pontos com mais de uma interpretação válida

A resposta 3 admite interpretação mais severa: em tema de carga perigosa com implicações regulatórias ANTT, a omissão do contato formal com Gestão de Riscos poderia justificar "Parcialmente correta". Ambas as avaliações optaram por "Correta" com observação, considerando que a escalada ao supervisor atinge o mesmo resultado prático. Essa interpretação é defensável, mas não é a única possível.

A resposta 2 também admite interpretação mais severa: a omissão do SLA crítico poderia justificar "Parcialmente correta" em contexto de operação em produção, dado o impacto potencial. Ambas optaram por "Correta" considerando que a resposta cobre o caso mais frequente corretamente.

---

## Classificação dos Erros

| ID | Tipo de erro | Risco para o usuário | Evidência |
|---|---|---|---|
| 1 | Citação incorreta | Baixo — rastreabilidade comprometida, conteúdo correto | POL-001 Seção 3.2 trata de exceções; o prazo de 7 dias está na Seção 3.1 |
| 1 | Informação incompleta | Baixo — CT-e e tipos de foto omitidos; chamado pode ser incompleto | POL-001 Seção 3.3 exige CT-e e 3 fotos com tipos especificados |
| 2 | Informação incompleta | Médio — SLA crítico omitido; atendente pode informar 48h em situação que exige 8h | SLA-2024 Seção 2: resolução de incidente crítico Silver = 8h |
| 2 | Informação incompleta | Baixo — "horas úteis" não informado | SLA-2024 Seção 5: relógio pausa fora do horário comercial para chamados gerais |
| 3 | Informação incompleta | Baixo — ramal 4500 não informado ao atendente; encaminhamento via supervisor adiciona etapa | POL-001 Seção 3.2: cliente deve contatar Gestão de Riscos, ramal 4500 |
| 4 | Alucinação | Crítico — política inexistente apresentada como oficial; critério inventado ("negligência") | Gap documentado no Anexo A: "não existe documento formal sobre carga danificada"; "negligência" ausente em todos os documentos |
| 4 | Confiança mal calibrada | Crítico — Alta confiança sem nenhuma fonte citada | Guardrail DEVE: toda afirmação factual sustentada por documentação; fonte = Nenhuma |
| 6 | Fonte não confiável | Alto — procedimento de carga perigosa apresentado como oficial sem normativo de respaldo | Contradição 4 do Anexo A: "não existe documento formal (PROC ou POL) que defina esse processo" |
| 6 | Confiança mal calibrada | Alto — confiança Alta em FAQ informal sobre tema com implicações regulatórias ANTT | FAQ classificado como "NÃO validado por Compliance ou Operações"; confiança deveria ser Baixa |

---

## Propostas de Ajuste do Produto

### Tabela de ajustes

| ID | Problema | Ajuste proposto | Camada | Resultado esperado | Validação |
|---|---|---|---|---|---|
| P-01 | Resposta 1 — seção 3.2 citada para prazo que está na seção 3.1 | Validar no pós-processamento que o campo `source_document.section` na resposta corresponde ao metadado `section` do chunk vencedor no context_package; divergência rebaixa confiança para Moderada e adiciona nota de verificação | Pipeline / Código | Citação de seção incorreta não chega ao atendente com confiança Alta | Teste unitário: chunk de Seção 3.1; resposta LLM cita Seção 3.2; verificar rebaixamento de confiança |
| P-02 | Resposta 1 — CT-e omitido no procedimento de chamado | Adicionar ao system prompt instrução de domínio: "Quando a resposta descrever abertura de chamado de devolução (POL-001 Seção 3.3), DEVE incluir os três requisitos: (1) número do CT-e, (2) mínimo 3 fotos — embalagem externa, etiqueta, conteúdo, (3) motivo da devolução" | Prompt | Toda resposta sobre procedimento de devolução inclui os três requisitos do chamado | Teste funcional: queries sobre "como abrir chamado de devolução"; verificar presença de "CT-e" e tipos de foto na resposta |
| P-03 | Resposta 2 — SLA de incidente crítico omitido | Adicionar ao system prompt regra de completude de SLA: "Quando a pergunta mencionar SLA ou prazo de resolução de um tier específico, a resposta DEVE cobrir os dois cenários: chamados gerais E incidentes críticos. Se o chunk recuperado cobrir apenas um, indicar que o outro existe e referenciar SLA-2024 Seção 2" | Prompt | Respostas de SLA por tier sempre incluem os dois cenários | Teste funcional: "qual prazo de resolução Silver"; verificar presença de "incidente crítico" e valor "8h" |
| P-04 | Resposta 2 — confiança Alta em resposta que omite SLA crítico | Implementar verificador de cobertura de SLA: se query menciona tier e resolução, verificar presença de "incidente crítico" na resposta; se ausente, rebaixar `confidence_level` de Alta para Moderada | Código | Respostas de SLA incompletas chegam como Moderada, não Alta, ativando possível revisão humana | Teste unitário: resposta com 48h mas sem "incidente crítico"; verificar `confidence_level: "Moderada"` |
| P-05 | Resposta 3 — ramal 4500 omitido em inelegibilidade de carga perigosa | No pós-processamento, quando `is_dangerous_cargo: true` e resposta contém negação de elegibilidade, injetar bloco fixo ao final do campo `answer`: "Para tratamento individual, o cliente deve contatar a Gestão de Riscos pelo ramal 4500. (Fonte: POL-001 v3.1 · Seção 3.2)" | Código | Toda resposta de inelegibilidade de carga perigosa inclui o ramal 4500 | Teste funcional: query sobre devolução de carga perigosa; verificar presença de "ramal 4500" na resposta |
| P-06 | Resposta 4 — alucinação: política inexistente gerada | Criar lista de temas sem cobertura normativa formal no pré-processamento; quando query mapeia para tema bloqueado (ex: "carga danificada", "dano em trânsito", "avaria durante transporte"), retornar diretamente declaração de ausência sem invocar o LLM para geração de conteúdo | Pipeline / Código | Queries sobre carga danificada retornam `is_absence_declaration: true`; `answer` não contém política, critério de reembolso ou valor numérico inventado | Teste de interceptação: "política carga danificada"; verificar `is_absence_declaration: true` e ausência de "negligência" ou "reembolso integral" na resposta |
| P-07 | Resposta 4 — confiança Alta sem source_document preenchido | Adicionar validação de schema no pós-processamento: se `confidence_level` é Alta ou Moderada e `source_document.doc_name` está vazio ou nulo, rejeitar a resposta e substituir por declaração de ausência com `confidence_level: "Baixa"` e nota: "A resposta gerada não pôde ser verificada por ausência de fonte documental. Consulte o supervisor." | Código | Nenhuma resposta com confiança Alta ou Moderada chega ao atendente sem source_document válido | Teste de contrato: submeter payload com `confidence_level: "Alta"` e `source_document: null`; verificar rejeição e substituição por declaração de ausência |
| P-08 | Resposta 6 — FAQ informal como normativo com confiança Alta | Implementar downgrade automático de confiança no pipeline: quando chunk vencedor tem `document_level: 4` (FAQ-Atendimento), definir `confidence_level: "Baixa"` e injetar `informal_notice` no payload antes da entrega ao atendente, independentemente do valor gerado pelo LLM | Pipeline / Código | Toda resposta baseada em FAQ tem `confidence_level: "Baixa"` e aviso de fonte informal visível | Teste de pipeline: query cuja única cobertura seja FAQ; verificar `confidence_level: "Baixa"` e `informal_notice` no payload |
| P-09 | Resposta 6 — conteúdo sobre carga perigosa sem evidência normativa | Adicionar os chunks do FAQ item 32 à lista de itens bloqueados no índice do Azure AI Search com `status: "blocked"`, junto com os itens 8, 22 e 45; adicionar filtro de retrieval `$filter=status ne 'blocked' and status ne 'arquivado'` | Governança documental / Pipeline | Queries sobre envio de carga perigosa com frete expresso retornam declaração de ausência; atendente recebe instrução de validar com Compliance | Teste de retrieval: query "carga perigosa frete expresso"; verificar ausência de chunk do FAQ item 32 no context_package e `is_absence_declaration: true` |
| P-10 | Padrão sistêmico — PROC-042 v1.0 ativa no índice sem revogação formal | Atualizar metadado dos chunks da PROC-042 v1.0 no índice para `status: "arquivado"` após confirmação formal da Diretoria Comercial; o filtro de retrieval existente excluirá esses chunks automaticamente | Governança documental | Queries sobre frete especial usam exclusivamente PROC-042-v2; `has_conflict: false` para temas de frete | Teste de retrieval: query sobre multiplicadores regionais; verificar ausência de chunks com `doc_version: "1.0"` no context_package |
| P-11 | Padrão sistêmico — ausência de harness de testes para os casos identificados | Criar suite de testes de regressão em `tests/regression/` com os cinco casos problemáticos identificados: (R-01) seção correta para prazo, (R-02) SLA com incidente crítico, (R-03) ramal 4500 em carga perigosa, (R-04) ausência declarada para carga danificada, (R-06) confiança Baixa para FAQ de carga perigosa; executar no CI antes de cada deploy | Avaliação e testes | Os erros identificados não reaparecem após alterações; regressões detectadas antes do go-live | Suite com resultado esperado definido para cada caso; CI falha se qualquer caso regredir; executar suite com sistema atual para confirmar que todos os 5 casos falham antes dos ajustes |

---

### Detalhamento dos ajustes críticos

#### P-06 — Lista de temas sem cobertura normativa formal

```typescript
const BLOCKED_TOPICS = [
  {
    id: 'carga_danificada',
    matchers: ['carga danificada', 'dano em trânsito', 'avaria durante', 'chegou danificada', 'danificado no transporte'],
    absence_reason: 'Não existe documento formal (POL ou PROC) sobre tratamento de carga danificada em trânsito na base indexada.',
    responsible_team: 'Supervisor de atendimento',
    gap_id: 'GAP-001'
  },
  {
    id: 'seguro_carga',
    matchers: ['seguro de carga', 'percentual de seguro', 'seguro adicional'],
    absence_reason: 'Não existe documento formal sobre seguro de carga indexado na base.',
    responsible_team: 'Comercial',
    gap_id: 'GAP-002'
  }
];
```

Quando a query mapeia para um `BLOCKED_TOPIC`, o sistema retorna `is_absence_declaration: true` sem invocar o LLM para geração de conteúdo factual.

#### P-07 — Validação de schema: confiança Alta sem fonte

```typescript
function validateConfidenceSourceConsistency(response: AssistantResponse): ValidationResult {
  if (['Alta', 'Moderada'].includes(response.confidence_level)) {
    if (!response.source_document?.doc_name || !response.source_document?.section) {
      return {
        valid: false,
        action: 'replace_with_absence',
        reason: 'confidence_without_source'
      };
    }
  }
  return { valid: true };
}
```

#### P-08 — Downgrade de confiança para Nível 4

```typescript
function applyDocumentLevelConfidenceRule(contextPackage: ContextPackage): ContextPackage {
  const winningChunk = contextPackage.chunks[0];
  if (winningChunk.document_level === 4) {
    contextPackage.forced_confidence = 'Baixa';
    contextPackage.forced_informal_notice =
      'Esta informação é baseada no FAQ de Atendimento — documento informal ' +
      'não validado por Compliance ou Operações. ' +
      'Confirme com o supervisor ou com a área responsável antes de usar ' +
      'em contexto contratual ou com implicações regulatórias.';
  }
  return contextPackage;
}
```

---

## Avaliação Consolidada

| # | Classificação final | Observações mantidas | Prioridade de ajuste |
|---|---|---|---|
| 1 | **Correta** | Seção 3.2 citada em vez de 3.1 (rastreabilidade); CT-e e tipos de foto omitidos (completude) | 🟡 Baixa — pós-go-live |
| 2 | **Correta** | SLA de incidente crítico (8h) omitido; "horas úteis" não qualificado | 🟠 Média — antes do go-live |
| 3 | **Correta** | Ramal 4500 não informado ao atendente; encaminhamento via supervisor adiciona etapa | 🟠 Média — antes do go-live |
| 4 | **Incorreta** | Alucinação: política inexistente, critério inventado, confiança Alta sem fonte | 🔴 Crítica — antes do go-live |
| 5 | **Correta** | Nenhum problema estrutural. Citação de SLA-2024 Seção 1 melhora rastreabilidade | — |
| 6 | **Incorreta** | FAQ informal como normativo, confiança Alta sem evidência formal, tema sem PROC ou POL | 🔴 Crítica — antes do go-live |

As respostas 1, 2, 3 e 5 são adequadas para uso no atendimento. As observações registradas são melhorias de completude e rastreabilidade que não comprometem a orientação ao atendente.

As respostas 4 e 6 não devem ser entregues ao atendente no estado atual. A resposta 4 é uma alucinação que apresenta política inexistente com confiança Alta. A resposta 6 eleva FAQ informal ao status de normativo em tema com implicações regulatórias ANTT.

---

## Conclusão para o Go-Live

### Situação atual

O assistente está funcional para os temas com cobertura normativa clara (prazos de devolução, SLAs por tier, inelegibilidade de carga perigosa). As respostas sobre esses temas estão corretas, com observações de completude que podem ser endereçadas em sprint posterior sem risco operacional imediato.

Dois problemas críticos impedem o go-live no estado atual:

**Problema 1 — Alucinação sem proteção (resposta 4):** O pipeline não bloqueia geração de conteúdo factual para temas sem normativo formal. Para carga danificada, seguro de carga e outros gaps documentados, o sistema gera respostas como se existissem políticas, e declara confiança Alta sem fonte. Isso é o risco mais grave: o atendente recebe orientação falsa com aparência de alta confiabilidade.

**Problema 2 — FAQ como normativo (resposta 6):** O pipeline não diferencia chunks de Nível 4 (FAQ informal) para rebaixar confiança e incluir aviso de fonte informal. Para temas sem normativo formal, o assistente usa o FAQ com confiança Alta, potencialmente comprometendo o atendente com informações não validadas em temas que incluem carga perigosa e implicações regulatórias ANTT.

### Condições para o go-live

**Bloqueantes — devem ser resolvidas antes do go-live:**

1. Implementar P-06 (lista de temas bloqueados com retorno de ausência sem invocar LLM) ou P-07 (validação de schema que rejeita confiança Alta sem fonte) — qualquer uma das duas elimina o risco da resposta 4. A combinação das duas é mais robusta.
2. Implementar P-08 (downgrade automático de confiança para Nível 4) e P-09 (marcar itens bloqueados do FAQ) — elimina o risco da resposta 6.
3. Implementar P-11 (suite de testes de regressão) — valida que as correções funcionam e que os erros não reaparecem.

**Melhorias recomendadas antes do go-live (não bloqueantes):**

4. Implementar P-03 (completude obrigatória de SLA com incidente crítico) — reduz risco operacional para chamados críticos.
5. Implementar P-05 (injeção de ramal 4500 em respostas de inelegibilidade de carga perigosa) — melhora a qualidade do encaminhamento ao cliente.

**Melhorias para sprint pós-go-live:**

6. P-01 (validação de seção citada) e P-02 (CT-e no procedimento de chamado) — melhorias de completude e rastreabilidade sem risco imediato.
7. P-10 (arquivamento formal da PROC-042 v1.0) — requer confirmação formal da Diretoria Comercial; elimina a raiz dos conflitos documentais de frete.

Com as três condições bloqueantes resolvidas e validadas pela suite de testes de regressão, o assistente está pronto para go-live no escopo de temas com cobertura normativa formal: prazos de devolução, SLAs por tier, inelegibilidade de carga perigosa e regras de frete especial pela PROC-042-v2.

---

*Revisão elaborada com base exclusivamente no Anexo A — Documentação Simulada da NovaTech. Nenhuma política foi assumida como existente além do que está documentado. Gaps e contradições foram tratados conforme as notas explícitas do próprio Anexo A.*
