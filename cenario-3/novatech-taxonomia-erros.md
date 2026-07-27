# Taxonomia de Erros — Respostas Problemáticas
## NovaTech Assistant · Análise Pré-Go-Live

**Data:** 05/06/2026  
**Elaborado por:** Product Specialist Sênior  
**Fonte de verdade:** Anexo A — Documentação Simulada da NovaTech  
**Base:** Avaliação principal + avaliação independente + análise comparativa

> As respostas 4, 5 e 6 foram identificadas como as mais críticas. As respostas 1, 2 e 3 foram classificadas como parcialmente corretas — seus erros são incluídos aqui porque, apesar de menos graves, indicam padrões de falha sistêmicos que se repetirão em produção se não forem tratados.
> A resposta 5 foi classificada como correta e não é analisada neste documento.

---

## Tabela Consolidada

| ID | Tipo de erro | Risco para o usuário | Evidência |
|---|---|---|---|
| 1 | Citação incorreta | Baixo-médio — rastreabilidade comprometida; auditoria posterior encontrará inconsistência | POL-001 Seção 3.2 trata de exceções ao prazo; o prazo de 7 dias úteis está na Seção 3.1 |
| 1 | Informação incompleta | Médio — atendente pode abrir chamado sem CT-e e ser rejeitado na triagem | POL-001 Seção 3.3 exige CT-e, 3 fotos (embalagem externa, etiqueta, conteúdo) e motivo; resposta omite CT-e e especificação das fotos |
| 2 | Informação incompleta | Alto — atendente pode informar 48h a cliente em incidente crítico, cujo SLA real é 8h | SLA-2024 Seção 2 distingue chamados gerais (48h) de incidentes críticos (8h) para o tier Silver; resposta cobre apenas chamados gerais |
| 2 | Informação incompleta | Médio — expectativa de prazo incorreta para atendimentos fora do horário comercial | SLA-2024 Seção 5: "o relógio de SLA pausa fora do horário comercial (08h–18h, dias úteis) para chamados gerais"; resposta omite a natureza das horas |
| 2 | Confiança mal calibrada | Médio — confiança Alta sinaliza resposta completa quando está incompleta em dimensão crítica | Confiança Alta implica cobertura total da pergunta; omissão do SLA de incidente crítico torna a resposta insuficiente para o universo da pergunta |
| 3 | Informação incompleta | Alto — cliente com carga perigosa fica sem contato correto para encaminhamento | POL-001 Seção 3.2: "o cliente deve entrar em contato com o setor de Gestão de Riscos (ramal 4500) para tratamento individual"; resposta substitui isso por escalada interna genérica |
| 4 | Alucinação | Crítico — política inexistente apresentada como oficial ao atendente | Notas do Anexo A, seção de gaps: "não existe documento formal (POL ou PROC) sobre tratamento de carga danificada em trânsito"; critério "negligência da transportadora" não consta em nenhum documento do Anexo A |
| 4 | Fonte não confiável | Crítico — conteúdo diverge até da única fonte informal disponível | FAQ item 38 descreve "responsabilidade comprovada", prazo de 48h, envolvimento do Jurídico e e-mail sinistros@; resposta omite prazo, canal e usa critério diferente ("negligência") |
| 4 | Confiança mal calibrada | Crítico — Alta confiança sem nenhuma fonte é a violação de guardrail mais grave da amostra | Guardrail DEVE: "toda afirmação factual deve estar sustentada por documentação indexada"; fonte citada = Nenhuma; confiança = Alta |
| 6 | Fonte não confiável | Crítico — procedimento de carga perigosa apresentado como oficial sem normativo de respaldo | Contradição 4 do Anexo A: "não existe documento formal (PROC ou POL) que defina esse processo. A informação pode ser prática informal não documentada"; FAQ-Atendimento classificado como "NÃO validado por Compliance ou Operações" |
| 6 | Confiança mal calibrada | Crítico — Alta confiança em fonte informal sobre tema regulatório ANTT | Guardrail NÃO DEVE: "tratar FAQ informal como documento normativo"; confiança deveria ser Baixa com aviso de fonte informal obrigatório |

---

## Análise Detalhada por Resposta

---

### Resposta 1

#### Erro 1.1 — Citação incorreta

**Descrição:** A resposta cita POL-001 Seção 3.2 para sustentar o prazo de 7 dias úteis. A Seção 3.2 trata das exceções ao prazo geral — cargas perigosas, refrigeradas e com lacre violado. O prazo de 7 dias úteis está na Seção 3.1 ("Prazo geral").

**Por que isso importa além da rastreabilidade formal:** Um atendente que acesse a Seção 3.2 para verificar a fonte encontrará apenas as exceções, não o prazo. Um sistema de auditoria que valide citações encontrará inconsistência. Em revisão humana de uma reclamação de cliente, a seção errada pode enfraquecer a defesa da empresa.

**Risco para o usuário:** Baixo-médio — o conteúdo factual não causa orientação errada imediata, mas compromete auditabilidade e pode causar confusão se o atendente verificar a fonte.

**Evidência:**
- POL-001 Seção 3.1: "O cliente pode solicitar a devolução de mercadorias em até 7 (sete) dias úteis após a data de recebimento confirmada no sistema de tracking."
- POL-001 Seção 3.2: Lista exclusivamente cargas perigosas, refrigeradas e com lacre violado como exceções. Não menciona o prazo de 7 dias.

---

#### Erro 1.2 — Informação incompleta

**Descrição:** A resposta orienta o atendente a "abrir chamado no portal e anexar fotos." A Seção 3.3 da POL-001 define que o chamado deve incluir: número do CT-e, fotos (mínimo 3, com especificação: embalagem externa, etiqueta de identificação e conteúdo) e motivo da devolução. A resposta omite o CT-e e não especifica os tipos de foto.

**Por que isso importa além da omissão:** O CT-e é um requisito da triagem (Seção 3.3, item 3). Um chamado aberto sem CT-e será rejeitado ou ficará pendente nas 4 horas de triagem — atrasando o processo e gerando retrabalho para o atendente e frustração para o cliente.

**Risco para o usuário:** Médio — o atendente orienta o cliente a abrir chamado incompleto, gerando rejeição na triagem com SLA comprometido.

**Evidência:**
- POL-001 Seção 3.3, item 2: "O chamado deve incluir: número do CT-e (Conhecimento de Transporte Eletrônico), fotos da mercadoria no estado atual (mínimo 3 fotos: embalagem externa, etiqueta de identificação, e conteúdo), e motivo da devolução."

---

### Resposta 2

#### Erro 2.1 — Informação incompleta (dimensão crítica)

**Descrição:** A resposta informa 48h como prazo de resolução para o tier Silver. Isso está correto para chamados gerais. Omite completamente que, em incidentes críticos, o SLA de resolução do tier Silver é de 8 horas — não 48h.

**Por que isso importa além da omissão:** A pergunta "qual o prazo de resolução" não especificou o tipo de chamado. Em produção, a maioria das perguntas virá nesse formato genérico. O assistente que responde apenas com 48h está cobrindo menos da metade do espaço de respostas válidas para a pergunta — e omite exatamente o dado mais crítico do ponto de vista operacional.

**Risco para o usuário:** Alto — em um incidente crítico (carga acima de R$100k, carga perigosa irregular, risco à segurança), o atendente informa 48h ao cliente quando o compromisso contratual é 8h. Isso pode gerar penalidades contratuais e comprometimento com o cliente por prazo incorreto.

**Evidência:**
- SLA-2024 Seção 2, linha "Tempo de resolução (chamados gerais)" — Silver: "Até 48h úteis"
- SLA-2024 Seção 2, linha "Tempo de resolução (incidentes críticos)" — Silver: "Até 8h"
- SLA-2024 Seção 3: define os critérios de incidente crítico que ativam o SLA de 8h

---

#### Erro 2.2 — Informação incompleta (dimensão técnica)

**Descrição:** A resposta diz "48h" sem especificar que são horas úteis. O SLA-2024 Seção 5 define que o relógio pausa fora do horário comercial (08h–18h, dias úteis) para chamados gerais.

**Risco para o usuário:** Médio — cliente pode calcular 48h corridas e esperar uma resolução que, pelo contrato, não está prometida para esse prazo. Em chamado aberto na sexta às 17h, o cliente pode esperar resposta até domingo, quando o SLA real reiniciará na segunda-feira.

**Evidência:**
- SLA-2024 Seção 5: "O relógio de SLA pausa fora do horário comercial (08h-18h, dias úteis) para chamados gerais."

---

#### Erro 2.3 — Confiança mal calibrada

**Descrição:** A confiança Alta sinaliza ao sistema (e ao atendente) que a resposta é completa e confiável. Uma resposta que omite o SLA de incidente crítico — que é o SLA mais relevante em situações urgentes — não deveria ser sinalizada como Alta.

**Risco para o usuário:** Médio — confiança Alta reduz a probabilidade de o atendente verificar a resposta antes de usá-la. Em um sistema com HITL (human-in-the-loop), confiança Alta pode suprimir revisão humana justamente em um caso que a exigiria.

**Evidência:** Critério de confiança Alta do produto requer cobertura adequada da pergunta com fonte de Nível 1 sem conflito. A pergunta sobre "prazo de resolução" de um tier envolve dois cenários distintos; cobrir apenas um não satisfaz o critério de cobertura adequada.

---

### Resposta 3

#### Erro 3.1 — Informação incompleta (encaminhamento formal)

**Descrição:** A resposta orienta o atendente a "escalar para o supervisor." A POL-001 Seção 3.2 define explicitamente que o cliente — não o atendente — deve contatar o setor de Gestão de Riscos pelo ramal 4500. São dois encaminhamentos diferentes: um é interno (supervisor), o outro é o procedimento formal que o cliente deve seguir (Gestão de Riscos, ramal 4500).

**Por que isso importa além da omissão:** Escalar para o supervisor é uma ação do atendente. Contatar Gestão de Riscos é a ação que o cliente deve executar. A confusão entre os dois deixa o cliente sem saber o que fazer, dependendo do supervisor para repassar a informação que o assistente deveria ter fornecido diretamente.

**Risco para o usuário:** Alto — o cliente com carga perigosa fica sem o canal de resolução correto. O supervisor precisará ser consultado para repassar o ramal 4500, adicionando etapas desnecessárias em um processo que envolve cargo com alto risco regulatório.

**Evidência:**
- POL-001 Seção 3.2: "Para essas categorias, o cliente deve entrar em contato com o setor de Gestão de Riscos (ramal 4500) para tratamento individual."

---

### Resposta 4

#### Erro 4.1 — Alucinação

**Descrição:** A resposta afirma que "a política de danos prevê reembolso integral quando comprovada negligência da transportadora, mediante laudo técnico e fotos." O critério "negligência da transportadora" não existe em nenhum documento do Anexo A — nem na POL-001, nem na PROC-042, nem no SLA-2024. O FAQ item 38 usa "responsabilidade comprovada" como critério, que é semanticamente distinto de "negligência" (responsabilidade é mais ampla e inclui culpa sem negligência intencional). A resposta apresentou como política oficial uma regra que não existe formalmente.

**Risco para o usuário:** Crítico — um atendente que use essa resposta pode comprometer a NovaTech com um padrão de reembolso inexistente, ou pode negar um reembolso legítimo por aplicar critério diferente do real. Em contexto de contestação jurídica, "política de danos" como documento citado não existe e não pode ser apresentado.

**Evidência:**
- Notas do Anexo A, Gaps identificados: "Política de carga danificada: o FAQ Item 38 descreve um processo, mas não existe documento formal (POL ou PROC) sobre tratamento de carga danificada em trânsito."
- Busca em todos os documentos do Anexo A: o termo "negligência" não aparece em nenhum documento.
- FAQ item 38: usa "se comprovada responsabilidade nossa, reembolsa integralmente" — não "negligência."

---

#### Erro 4.2 — Fonte não confiável

**Descrição:** Além de não citar nenhuma fonte, o conteúdo da resposta é inconsistente com a única fonte disponível — o FAQ item 38. O FAQ menciona: prazo de 48h para registrar a ocorrência, laudo "se possível", encaminhamento ao Jurídico, e e-mail sinistros@novatech.com.br. A resposta omite prazo, canal de encaminhamento e altera o critério de reembolso. Ou seja: a resposta não é sustentada nem pelo documento informal que poderia ter servido de base.

**Risco para o usuário:** Crítico — o atendente recebe uma versão do processo que não corresponde nem à prática informal descrita no FAQ. Isso aumenta o risco de orientação incorreta ao cliente e de inconsistência entre atendentes que usem o assistente em chamados similares.

**Evidência:**
- FAQ-Atendimento item 38: "O cliente precisa registrar a ocorrência em até 48h após o recebimento, com fotos e laudo se possível. A NovaTech investiga e, se comprovada responsabilidade nossa, reembolsa integralmente. Mas isso passa pelo Jurídico, não pelo atendimento normal — encaminhe para o e-mail sinistros@novatech.com.br."
- Resposta avaliada não menciona 48h, laudo, Jurídico, nem e-mail. Usa "negligência" em vez de "responsabilidade."

---

#### Erro 4.3 — Confiança mal calibrada

**Descrição:** Confiança Alta declarada para resposta sem nenhuma fonte citada, sobre tema sem normativo formal, com conteúdo divergente até da fonte informal disponível. Esta é a combinação de fatores mais severa da amostra do ponto de vista de violação de guardrails.

**Risco para o usuário:** Crítico — confiança Alta suprime revisão humana (HITL). Um sistema que deixa passar essa resposta com Alta confiança está falhando na camada de controle mais fundamental do produto.

**Evidência:** Guardrail DEVE: "A confiança deve refletir a qualidade e autoridade da fonte." Guardrail NÃO DEVE: "Ocultar ausência, conflito ou fragilidade da evidência." Ambos violados simultaneamente.

---

### Resposta 6

#### Erro 6.1 — Fonte não confiável

**Descrição:** A resposta usa o FAQ-Atendimento item 32 como única fonte para afirmar que cargas perigosas podem ser enviadas com frete expresso mediante autorização do Compliance. O FAQ-Atendimento é classificado no próprio Anexo A como "documento informal — NÃO validado por Compliance ou Operações." O Anexo A documenta explicitamente na contradição número 4 que não existe documento formal definindo esse processo.

**Por que isso é mais grave do que usar FAQ em outros contextos:** O tema envolve carga perigosa sujeita à regulação da ANTT. Procedimentos de carga perigosa têm requisitos legais externos à NovaTech. Apresentar como procedimento estabelecido uma informação informal sobre carga perigosa — especialmente envolvendo "autorização do Compliance" como critério — pode levar o atendente a comprometer operação de carga perigosa sem que o processo de autorização real tenha ocorrido.

**Risco para o usuário:** Crítico — impacto regulatório externo (ANTT), além do risco operacional interno. Um atendente que confie nessa resposta pode confirmar ao cliente o envio de carga perigosa por frete expresso sem que a autorização formal exista ou sem que a documentação ANTT esteja de fato em ordem.

**Evidência:**
- Notas do Anexo A, Contradições: "FAQ Item 32 vs documentação formal: O FAQ diz que carga perigosa pode ser enviada com frete expresso 'com autorização', mas não existe documento formal (PROC ou POL) que defina esse processo. A informação pode ser prática informal não documentada."
- FAQ-Atendimento, classificação: "Documento informal — NÃO validado por Compliance ou Operações."

---

#### Erro 6.2 — Confiança mal calibrada

**Descrição:** A resposta declara confiança Alta para informação baseada exclusivamente em FAQ informal sobre tema sem normativo formal. Pela taxonomia de confiança do produto, qualquer resposta baseada em Nível 4 (FAQ) deve ter confiança Baixa com aviso de fonte informal obrigatório.

**Risco para o usuário:** Crítico — confiança Alta em fonte informal de tema regulatório ANTT é a combinação de risco mais alta desta amostra. O sistema não sinaliza ao atendente que ele está operando sem respaldo normativo em uma decisão com implicações regulatórias externas.

**Evidência:** Guardrail DEVE: "A confiança deve refletir a qualidade e autoridade da fonte." Guardrail NÃO DEVE: "Tratar FAQ informal como documento normativo." Critério de confiança Baixa do produto: qualquer resposta baseada em Nível 4 (FAQ) deve ter confiança Baixa.

---

## Padrões Sistêmicos Identificados

A análise dos erros revela três padrões que se repetiriam em produção além das seis respostas avaliadas:

### Padrão 1 — Respostas incompletas com confiança Alta

As respostas 2 e 3 têm informação factual correta mas omitem condições críticas — e declaram confiança Alta. Isso indica que o sistema não está penalizando incompletude na calibração de confiança. Em produção, um assistente que declara Alta confiança para respostas incompletas suprime a revisão humana exatamente quando ela seria necessária.

**Impacto esperado em escala:** Para 320 chamados/dia com 60% de consulta documental, ~192 chamados/dia. Se o padrão de incompletude se mantiver, um percentual desses chamados resultará em atendentes comprometendo SLAs incorretos com clientes ou orientando clientes sem o contato formal correto.

### Padrão 2 — Ausência de bloqueio para respostas sem fonte com confiança Alta

A resposta 4 passou pelo sistema com confiança Alta e sem nenhuma fonte. Isso indica que o pipeline não está validando a presença de `source_document` como pré-condição para confiança Alta. Este é o cenário exato que o structured output e a validação de schema deveriam prevenir — e aparentemente não estão prevenindo.

**Impacto esperado em escala:** Qualquer tema sem cobertura normativa formal na base (carga danificada, seguro, frete padrão) pode gerar respostas com conteúdo inventado e confiança Alta se o bloqueio não estiver implementado.

### Padrão 3 — FAQ informal como normativo em temas sensíveis

A resposta 6 usa FAQ como normativo em tema de carga perigosa. O FAQ-Atendimento cobre múltiplos temas sem normativo (seguro de carga, carga danificada, frete expresso para carga perigosa). Se o pipeline não está diferenciando Nível 4 de Nível 1–3, todos esses temas produzirão respostas com confiança inadequada.

**Impacto esperado em escala:** Os itens bloqueados do FAQ (8, 22, 32, 45) cobrem temas que aparecem nas categorias mais consultadas (regras de frete: 25%, outros: 20%). Sem bloqueio explícito, esses itens serão usados como fonte sempre que o retrieval não encontrar normativo de nível superior.

---

## Tabela de Prioridade de Correção

| Prioridade | Resposta(s) | Tipo(s) de erro | Ação recomendada |
|---|---|---|---|
| 🔴 Crítica — antes do go-live | 4, 6 | Alucinação, Fonte não confiável, Confiança mal calibrada | Bloquear respostas sem `source_document` quando confiança = Alta; implementar filtro de Nível 4 com downgrade obrigatório de confiança para Baixa |
| 🟠 Alta — antes do go-live | 2 | Informação incompleta (SLA crítico), Confiança mal calibrada | Ajustar prompt para exigir distinção entre chamados gerais e incidentes críticos em qualquer resposta de SLA |
| 🟡 Média — sprint pós-go-live | 1, 3 | Citação incorreta, Informação incompleta | Implementar validação de seção citada contra chunk recuperado; incluir Gestão de Riscos como campo obrigatório em respostas sobre categorias inelegíveis |

---

*Taxonomia elaborada com base no Anexo A — Documentação Simulada da NovaTech, nos guardrails formalizados do produto e nas avaliações registradas em novatech-avaliacao-respostas.md e novatech-comparacao-avaliacoes.md.*
