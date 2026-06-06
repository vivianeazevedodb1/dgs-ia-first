# Análise Cruzada — Inconsistências Normativas × Práticas Informais do FAQ
**Elaborado por:** Product Specialist  
**Data:** 05/06/2026  
**Fontes de entrada:**
- `FAQ-Atendimento` — documento informal do time de atendimento
- `mapa_de_temas_e_gaps.md` — temas cobertos e hipóteses de gaps (05/06/2026)
- `análise_de_inconsistências.md` — inconsistências PROC-042 v1 × v2 e FAQ × PROC-042-v2 (05/06/2026)

---

## 1. Objetivo

Este documento consolida em uma visão única o que já foi mapeado separadamente: de um lado, as inconsistências entre documentos normativos (PROC-042 v1 × v2); de outro, os gaps da base documental (mapa de temas). O cruzamento com as práticas informais registradas no FAQ-Atendimento permite identificar quais inconsistências já estão **sendo operadas incorretamente no dia a dia**, quais gaps estão sendo **cobertos pelo FAQ de forma arriscada**, e onde há **dupla exposição** — inconsistência normativa + prática informal divergente simultâneas.

---

## 2. Modelo de Cruzamento

Cada item do cruzamento é classificado em um dos quatro quadrantes abaixo:

| Quadrante | Definição |
|---|---|
| 🔴 **Dupla Exposição** | Há inconsistência normativa E o FAQ pratica a versão incorreta ou adiciona outra camada de erro |
| 🟠 **Gap Operacionalizado pelo FAQ** | Não existe normativo formal, mas o FAQ preenche o vácuo com orientação informal e potencialmente incorreta |
| 🟡 **Inconsistência Normativa sem Reflexo no FAQ** | Há conflito entre documentos normativos, mas o FAQ não cobre o tema — risco latente sem propagação atual |
| 🟢 **Alinhamento** | FAQ alinhado com o normativo vigente ou tema coerente entre todas as fontes |

---

## 3. Cruzamento Item a Item

---

### CRZ-01 — Desconto por volume de frete especial
**Quadrante: 🔴 Dupla Exposição**

| Dimensão | Conteúdo |
|---|---|
| **PROC-042 v1** | Desconto apenas negociado via Comercial com aditivo; limiar >10 fretes/mês; sem percentual definido |
| **PROC-042 v2** | Desconto automático: 5% a partir de 8 fretes/mês; 10% acima de 15 fretes/mês; sobre o multiplicador regional |
| **FAQ (item 45)** | "Para clientes com mais de 10 fretes especiais por mês, existe desconto automático na tabela (veja PROC-042). Para outros casos, encaminhe ao Comercial." |
| **Inconsistência normativa (IC-04)** | v1 e v2 divergem em limiar (10 vs 8), percentuais (inexistente vs 5%/10%), natureza (manual vs automático) e base de cálculo |
| **Gap mapeado (Gap 2)** | Coexistência sem hierarquia formal entre versões |

**Análise do cruzamento:** O FAQ cria uma terceira versão do problema: afirma que existe desconto automático (correto para v2), mas usa o limiar da v1 (>10 fretes), omite o limiar de 15 fretes/mês e os percentuais de 5% e 10%, e não informa a base de cálculo (sobre o multiplicador regional). O resultado é que um atendente que segue o FAQ aplica uma regra híbrida que não corresponde a nenhum documento formal: reconhece automaticidade (v2) mas com parâmetros errados (v1). Clientes entre 8 e 10 fretes/mês estão sendo sistematicamente orientados a negociar algo a que já têm direito automático.

**Risco consolidado:** 🔴 Crítico — erro de processo ativo, impacto financeiro recorrente, potencial passivo contratual.

---

### CRZ-02 — Multiplicadores regionais para cotação de frete especial
**Quadrante: 🟡 Inconsistência Normativa sem Reflexo Direto no FAQ**

| Dimensão | Conteúdo |
|---|---|
| **PROC-042 v1** | Multiplicadores: Sul 1,2 / Sudeste 1,0 / Centro-Oeste 1,3 / Nordeste 1,4 / Norte 1,6 |
| **PROC-042 v2** | Multiplicadores: Sul 1,3 / Sudeste 1,1 / Centro-Oeste 1,4 / Nordeste 1,5 / Norte 1,8 |
| **FAQ (item 8)** | "Acima de 500kg, aplica a tabela de multiplicadores por região. Use a mais recente (v2), mas se o cliente reclamar do valor, pode ser que o contrato dele ainda esteja na tabela antiga." |
| **Inconsistência normativa (IC-01)** | Divergência em todas as regiões, de +7,1% (Nordeste) a +12,5% (Norte) |
| **Gap mapeado (Gap 2)** | Ausência de hierarquia formal entre versões |

**Análise do cruzamento:** O FAQ recomenda corretamente o uso da v2 como padrão. No entanto, ao sugerir que reclamações do cliente podem indicar que "o contrato dele ainda está na tabela antiga", o FAQ introduz uma válvula de escape informal sem critério objetivo para determinar qual contrato usa qual versão. Não existe no conjunto documental nenhum mecanismo formal para essa verificação (Gap 2). O atendente fica sem respaldo para confirmar ou negar a versão contratual do cliente, dependendo de consulta ao Comercial sem processo estruturado.

**Risco consolidado:** 🟡 Médio — prática do FAQ é parcialmente correta, mas introduz ambiguidade não resolvida pelos normativos.

---

### CRZ-03 — Prazo de entrega do frete especial
**Quadrante: 🔴 Dupla Exposição**

| Dimensão | Conteúdo |
|---|---|
| **PROC-042 v1** | Prazo padrão da rota + **2 dias úteis** |
| **PROC-042 v2** | Prazo padrão da rota + **3 dias úteis** (manuseio e roteirização) |
| **FAQ (item 8)** | Não menciona o prazo adicional — orienta apenas a usar a v2, sem detalhar seus parâmetros |
| **Inconsistência normativa (IC-03)** | Divergência de 1 dia útil entre versões |

**Análise do cruzamento:** O FAQ recomenda a v2 mas não reproduz seu conteúdo de prazo. Um atendente que consulte o FAQ como guia rápido e não leia integralmente a v2 desconhece a mudança de +2 para +3 dias úteis. Na prática, o risco é que o atendimento continue comunicando ao cliente o prazo da v1 (+2 dias), criando expectativa incorreta. A omissão no FAQ amplia o alcance da inconsistência normativa ao não corrigi-la na camada operacional.

**Risco consolidado:** 🟡 Médio — omissão do FAQ propaga o erro da v1 na comunicação com o cliente.

---

### CRZ-04 — Aprovação para cargas acima de 5.000 kg
**Quadrante: 🟡 Inconsistência Normativa sem Reflexo no FAQ**

| Dimensão | Conteúdo |
|---|---|
| **PROC-042 v1** | Cargas acima de 5.000 kg requerem aprovação prévia do gerente de operações regional |
| **PROC-042 v2** | Idem — regra mantida sem alteração |
| **FAQ** | Não menciona essa condição em nenhum dos 8 itens analisados |
| **Gap mapeado (IC-04 da análise FAQ × v2)** | Omissão operacional — atendente pode confirmar frete sem acionar aprovação |

**Análise do cruzamento:** Neste ponto, v1 e v2 estão alinhadas — a regra de aprovação é idêntica nas duas versões. O problema não é inconsistência entre normativos, mas ausência total de cobertura no FAQ. Um atendente que usa o FAQ como guia primário nunca saberá que cargas acima de 5.000 kg têm fluxo diferenciado. O gap é unilateral: está no FAQ, não entre os normativos.

**Risco consolidado:** 🟡 Médio — comprometimento operacional não autorizado em cargas de grande porte.

---

### CRZ-05 — Frete de cargas perigosas (acima de 500 kg)
**Quadrante: 🟠 Gap Operacionalizado pelo FAQ**

| Dimensão | Conteúdo |
|---|---|
| **PROC-042 v1 e v2** | Remete à PROC-043 (ausente no conjunto; v2 acrescenta que está em revisão pelo Compliance) |
| **FAQ (item 8)** | Menciona superficialmente que cargas perigosas têm tabela específica, sem detalhar |
| **FAQ (item 32)** | "Pode enviar carga perigosa com frete expresso? Sim, mas precisa de autorização do Compliance e documentação ANTT atualizada. Na prática, demora uns 2 dias." |
| **Gap mapeado (Gap 1)** | PROC-043 ausente e em revisão; frete de cargas perigosas não coberto formalmente |

**Análise do cruzamento:** O FAQ preenche com conhecimento tácito um vácuo normativo real. A orientação do item 32 — autorização do Compliance + ANTT atualizada + ~2 dias — não tem respaldo em nenhum documento formal do conjunto analisado. Com a PROC-043 em revisão, esse processo informal pode estar desalinhado com o que o Compliance está reformulando. Qualquer erro operacional em carga perigosa tem consequências regulatórias (ANTT) e de segurança que extrapolam o impacto financeiro.

**Risco consolidado:** 🔴 Crítico — área de alta sensibilidade regulatória operada exclusivamente por conhecimento tácito não validado.

---

### CRZ-06 — Devolução de carga perigosa
**Quadrante: 🟠 Gap Operacionalizado pelo FAQ**

| Dimensão | Conteúdo |
|---|---|
| **POL-001 (Seção 3.2)** | Cargas perigosas classes 1–6 NÃO são elegíveis para devolução pelo processo padrão; devem ser encaminhadas ao ramal 4500 (Gestão de Riscos) |
| **FAQ (item 3)** | "Oficialmente não pode pelo processo padrão, mas já tiveram casos em que o pessoal de Riscos autorizou exceção. Então não diga que é impossível — diga que precisa de tratamento especial." |
| **Gap mapeado** | Ausência de normativo para exceções; o FAQ operacionaliza informalmente o que deveria ser um processo formal |

**Análise do cruzamento:** O FAQ está alinhado com a POL-001 no ponto central (não é pelo processo padrão), mas acrescenta uma camada informal ao sugerir que exceções já ocorreram e orientar o atendente a não fechar a porta. Essa orientação, embora pragmática, não tem respaldo formal e pode criar expectativa no cliente de uma solução que depende de discricionariedade individual da Gestão de Riscos. Não existe normativo que defina critérios para essas exceções.

**Risco consolidado:** 🟠 Moderado — alinhamento parcial com a POL-001, mas a camada informal de "exceções possíveis" cria exposição sem processo.

---

### CRZ-07 — Carga danificada em trânsito
**Quadrante: 🟠 Gap Operacionalizado pelo FAQ**

| Dimensão | Conteúdo |
|---|---|
| **POL-001** | Não cobre carga danificada em trânsito — escopo é devolução pós-entrega |
| **Normativos PROC-042** | Não cobrem o tema |
| **FAQ (item 38)** | "Registrar ocorrência em até 48h com fotos e laudo. NovaTech investiga; se comprovada responsabilidade, reembolso integral. Encaminhar para sinistros@novatech.com.br." |
| **Gap mapeado (Gap 3)** | Ausência de normativo formal para esse fluxo |

**Análise do cruzamento:** O FAQ é a única fonte de orientação para um processo de alto impacto financeiro e reputacional. Os prazos (48h), o canal (sinistros@novatech.com.br), a condição de reembolso integral e o envolvimento do Jurídico são informações exclusivas do FAQ sem validação normativa. Se o prazo real for diferente, se o e-mail mudar, ou se as condições de reembolso tiverem restrições não documentadas, o atendimento continuará orientando incorretamente sem mecanismo de detecção.

**Risco consolidado:** 🟠 Moderado-Alto — gap crítico com única fonte de cobertura sendo documento informal não validado.

---

### CRZ-08 — Seguro de carga
**Quadrante: 🟠 Gap Operacionalizado pelo FAQ**

| Dimensão | Conteúdo |
|---|---|
| **Normativos (todos)** | Nenhum documento do conjunto cobre seguro de carga |
| **FAQ (item 22)** | "Seguro adicional: 0,3% do valor declarado para padrão; 0,8% para perigosas. Para contratos a partir de 2023. Contratos mais antigos: confirme com o Comercial." |
| **Gap mapeado (Gap 4)** | Ausência total de normativo formal |

**Análise do cruzamento:** Os percentuais de seguro informados no FAQ (0,3% e 0,8%) são a única referência disponível no conjunto documental e não têm respaldo normativo verificável. A distinção entre contratos pré e pós-2023 introduz uma variável adicional sem critério formal de verificação. Um cliente que receba cotação de seguro baseada nesses percentuais e depois descubra valores diferentes em contrato tem base para contestação.

**Risco consolidado:** 🟠 Moderado — informação comercialmente sensível operada sem normativo, com risco de cotação incorreta.

---

### CRZ-09 — Tiers de clientes (inexistência do tier Platinum)
**Quadrante: 🟢 Alinhamento**

| Dimensão | Conteúdo |
|---|---|
| **SLA-2024 (Seção 1)** | "Não existem outros tiers além dos três listados acima (Gold, Silver, Standard)." |
| **FAQ (item 15)** | "Não existe tier Platinum na NovaTech. Orienta que os tiers são Gold, Silver e Standard." |

**Análise do cruzamento:** Alinhamento total entre o documento contratual e o FAQ. O FAQ inclusive fornece contexto histórico útil (confusão com programa descontinuado em 2022) que não consta no SLA-2024, complementando sem contradizer.

**Risco consolidado:** 🟢 Baixo — ponto de coerência entre todas as fontes.

---

### CRZ-10 — SLAs de resposta e resolução por tier
**Quadrante: 🟢 Alinhamento**

| Dimensão | Conteúdo |
|---|---|
| **SLA-2024 (Seção 2)** | Gold: 2h/24h | Silver: 4h/48h | Standard: 8h/72h |
| **FAQ (item 41)** | Reproduz os mesmos valores: "Gold tem 2h de resposta e 24h de resolução. Silver é 4h e 48h. Standard é 8h e 72h." |

**Análise do cruzamento:** Os valores estão alinhados. O FAQ menciona corretamente que incidentes críticos têm prazos menores e remete à tabela SLA-2024 para detalhes, o que é um comportamento adequado de um documento de suporte.

**Risco consolidado:** 🟢 Baixo — coerência entre FAQ e normativo contratual.

---

### CRZ-11 — Rastreamento com status "em trânsito" por mais de 5 dias
**Quadrante: 🟠 Gap Operacionalizado pelo FAQ**

| Dimensão | Conteúdo |
|---|---|
| **Normativos** | Nenhum documento cobre procedimento de rastreamento ou critérios de escalada por atraso |
| **SLA-2024 (Seção 3)** | Define incidente crítico quando carga >R$100k está com status desconhecido há +6h, mas não cobre atrasos em rotas normais |
| **FAQ (item 27)** | "Rotas para o Norte: até 10 dias úteis é normal. Sul/Sudeste: mais de 3 dias parado é estranho. Abra chamado de rastreamento como prioridade alta se Gold ou valor >R$50.000." |
| **Gap mapeado** | Ausência de normativo para procedimento de rastreamento e escalada por atraso |

**Análise do cruzamento:** O FAQ define critérios operacionais relevantes (diferenciação por região, limiar de valor para escalada) que não existem em nenhum normativo. O critério de R$50.000 do FAQ difere do critério de incidente crítico do SLA-2024 (R$100.000) — o FAQ usa um limiar mais conservador para escalada interna, o que pode ser uma prática saudável ou uma inconsistência não documentada. Sem normativo, não é possível validar qual valor é o correto.

**Risco consolidado:** 🟠 Moderado — processo operacional crítico coberto apenas por conhecimento tácito, com possível divergência em relação ao limiar do SLA-2024.

---

## 4. Mapa de Calor Consolidado

| ID | Tema | Quadrante | Risco |
|---|---|---|---|
| CRZ-01 | Desconto por volume de frete especial | 🔴 Dupla Exposição | 🔴 Crítico |
| CRZ-05 | Frete de cargas perigosas | 🟠 Gap operacionalizado pelo FAQ | 🔴 Crítico |
| CRZ-03 | Prazo de entrega do frete especial | 🔴 Dupla Exposição | 🟡 Médio |
| CRZ-06 | Devolução de carga perigosa | 🟠 Gap operacionalizado pelo FAQ | 🟠 Moderado |
| CRZ-07 | Carga danificada em trânsito | 🟠 Gap operacionalizado pelo FAQ | 🟠 Moderado-Alto |
| CRZ-08 | Seguro de carga | 🟠 Gap operacionalizado pelo FAQ | 🟠 Moderado |
| CRZ-11 | Rastreamento e escalada por atraso | 🟠 Gap operacionalizado pelo FAQ | 🟠 Moderado |
| CRZ-02 | Multiplicadores regionais | 🟡 Inconsistência sem reflexo no FAQ | 🟡 Médio |
| CRZ-04 | Aprovação para cargas >5.000 kg | 🟡 Inconsistência sem reflexo no FAQ | 🟡 Médio |
| CRZ-09 | Tier Platinum inexistente | 🟢 Alinhamento | 🟢 Baixo |
| CRZ-10 | SLAs de resposta e resolução | 🟢 Alinhamento | 🟢 Baixo |

---

## 5. Padrões Identificados

### Padrão 1 — O FAQ como "normativo de fato" em áreas sem cobertura formal
Cinco dos onze temas cruzados (CRZ-05, CRZ-06, CRZ-07, CRZ-08, CRZ-11) não possuem normativo formal e são operados exclusivamente pelo FAQ. Em todos esses casos, a orientação informal cobre temas de alto impacto operacional, financeiro ou regulatório. O FAQ não foi concebido para esse papel, não passa por validação e não tem mecanismo de atualização — mas, na ausência de normativos, torna-se a única referência disponível para o atendimento.

### Padrão 2 — Hibridização de versões pelo FAQ
No caso mais crítico (CRZ-01), o FAQ não adotou integralmente a v1 nem a v2 da PROC-042: construiu uma regra híbrida que mistura a natureza automática do desconto (v2) com o limiar numérico incorreto (v1, que usa >10 fretes) e omite os percentuais e a base de cálculo de ambas. Esse tipo de hibridização é o resultado esperado quando documentos coexistem sem hierarquia formal e o FAQ é atualizado informalmente por memória coletiva.

### Padrão 3 — O FAQ acerta nos temas com normativo estável
Os dois pontos de alinhamento (CRZ-09 e CRZ-10) correspondem a temas cobertos por um único documento normativo claro e sem conflito de versões (SLA-2024). Onde há um normativo único, estável e bem estruturado, o FAQ tende a reproduzir corretamente as informações.

### Padrão 4 — Limiares divergentes entre FAQ e SLA-2024
O FAQ usa R$50.000 como critério de escalada de rastreamento (item 27), enquanto o SLA-2024 define R$100.000 como limiar para incidente crítico (Seção 3). Essa divergência pode tanto indicar uma camada conservadora de triagem interna (saudável) quanto uma inconsistência não gerenciada. Sem normativo específico para rastreamento, não é possível determinar qual critério prevalece.

---

## 6. Recomendações Priorizadas pelo Cruzamento

| Prioridade | Ação | Origem da Evidência |
|---|---|---|
| 🔴 Imediata | Corrigir FAQ (item 45) com limiar correto (8 fretes/mês), percentuais (5%/10%) e base de cálculo (sobre multiplicador regional) da PROC-042-v2 | CRZ-01 |
| 🔴 Imediata | Formalizar e publicar PROC-043 revisada; enquanto isso, estabelecer orientação provisória validada pelo Compliance para cargas perigosas | CRZ-05 |
| 🔴 Imediata | Declarar formalmente a PROC-042-v2 como vigente e arquivar a v1 — elimina a raiz dos erros de CRZ-01, CRZ-02 e CRZ-03 | IC-01 a IC-04 |
| 🟠 Curto prazo | Criar normativo para carga danificada em trânsito (formaliza e valida o que o FAQ item 38 já orienta) | CRZ-07 |
| 🟠 Curto prazo | Criar normativo para seguro de carga (valida ou corrige os percentuais do FAQ item 22) | CRZ-08 |
| 🟠 Curto prazo | Atualizar FAQ (item 8) com prazo de +3 dias úteis e incluir regra de aprovação para cargas >5.000 kg | CRZ-03, CRZ-04 |
| 🟡 Médio prazo | Definir normativo de rastreamento e escalada, harmonizando o critério de R$50k do FAQ com o limiar de R$100k do SLA-2024 | CRZ-11 |
| 🟡 Médio prazo | Formalizar critérios para exceções de devolução de carga perigosa via Gestão de Riscos | CRZ-06 |
| 🟢 Estrutural | Implementar ciclo formal de revisão do FAQ após cada publicação ou alteração de PROC/POL, com responsável designado e validação por Compliance | Padrões 1, 2 e 3 |

---

*Análise produzida por cruzamento entre o conteúdo integral do FAQ-Atendimento, o mapa de temas e gaps e as análises de inconsistências previamente elaboradas. Todas as conclusões são inferências analíticas que devem ser validadas com as áreas de Operações, Comercial, Compliance e Jurídico antes de qualquer ação corretiva.*
