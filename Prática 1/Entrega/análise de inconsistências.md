# Análise de Inconsistências — PROC-042 v1 × PROC-042-v2
**Elaborado por:** Product Specialist  
**Data:** 05/06/2026  
**Documentos analisados:**
- `PROC-042` — Versão 1.0, emitida em 03/03/2023, Diretoria Comercial
- `PROC-042-v2` — Versão 2.0, emitida em 10/11/2023, Diretoria Comercial

---

## 1. Sumário Executivo

Ambos os documentos regulam o mesmo objeto — cálculo de frete especial para cargas acima de 500 kg — e foram emitidos pela mesma área responsável (Diretoria Comercial). No entanto, **nenhum dos dois possui indicação formal de vigência, obsolescência ou relação de substituição entre si**, coexistindo no SharePoint sem hierarquia clara. A análise comparativa identificou **5 inconsistências diretas** em parâmetros de cálculo, prazos e regras de desconto, além de **2 divergências estruturais** que afetam a governança documental. O uso não controlado de qualquer das duas versões pelo atendimento ou pelo Comercial resulta em cotações divergentes para o mesmo cliente, com diferenças que podem chegar a **30% no multiplicador regional** dependendo da rota.

---

## 2. Contexto: O Problema de Coexistência

Antes das inconsistências de conteúdo, é necessário registrar o problema estrutural que as amplifica:

| Atributo de Governança | PROC-042 v1 | PROC-042-v2 |
|---|---|---|
| Indica que substitui versão anterior | Não | Não |
| Indica que foi substituído por versão posterior | Não | Não |
| Status formal no sistema | Indefinido | Indefinido |
| Localização | SharePoint (sem hierarquia) | SharePoint (sem hierarquia) |
| Período de transição definido | Não se aplica | Sim — chamados até 01/12/2023 usam v1 |

A ausência de controle formal significa que qualquer colaborador pode estar usando a versão errada sem saber. O período de transição definido na v2 (Seção 5) implica reconhecimento tácito de que a v2 substitui a v1 — mas isso nunca é declarado explicitamente, gerando ambiguidade jurídica e operacional.

---

## 3. Inconsistências Diretas

### IC-01 — Multiplicadores regionais divergentes em todas as regiões

A v2 revisou os multiplicadores de todas as cinco regiões para cima, refletindo atualização de custos operacionais. As diferenças são as seguintes:

| Região | v1 (mar/2023) | v2 (nov/2023) | Variação absoluta | Variação % |
|---|---|---|---|---|
| Sul | 1,2 | 1,3 | +0,1 | +8,3% |
| Sudeste | 1,0 | 1,1 | +0,1 | +10,0% |
| Centro-Oeste | 1,3 | 1,4 | +0,1 | +7,7% |
| Nordeste | 1,4 | 1,5 | +0,1 | +7,1% |
| Norte | 1,6 | 1,8 | +0,2 | +12,5% |

**Impacto:** Uma carga destinada ao Norte calculada com a v1 resulta em um frete 12,5% mais barato do que o correto pela v2. Para uma carga de alto valor base, a diferença pode ser substancial. Se diferentes atendentes usam versões diferentes, o mesmo cliente pode receber cotações incompatíveis para rotas equivalentes.

**Classificação de risco:** 🔴 Alto — impacto direto em receita e consistência de cotações.

---

### IC-02 — Fator de peso para a faixa intermediária (1.001–3.000 kg) divergente

| Faixa de peso | v1 | v2 | Variação |
|---|---|---|---|
| 500 kg a 1.000 kg | 1,0 | 1,0 | Sem alteração |
| 1.001 kg a 3.000 kg | **1,2** | **1,15** | −0,05 (−4,2%) |
| Acima de 3.000 kg | **1,5** | **1,4** | −0,1 (−6,7%) |

**Impacto:** Ao contrário dos multiplicadores regionais (que subiram na v2), os fatores de peso para as faixas superiores **reduziram** na v2. Isso cria uma assimetria: a v2 é mais cara por região, mas mais barata por peso nas faixas acima de 1.000 kg. O efeito líquido sobre o frete final depende da combinação de rota e peso — tornando impossível afirmar que uma versão é sistematicamente mais cara que a outra sem calcular caso a caso.

**Classificação de risco:** 🔴 Alto — cotações incorretas e resultado financeiro imprevisível dependendo da combinação usada.

---

### IC-03 — Prazo adicional de manuseio divergente

| Atributo | v1 (Seção 3) | v2 (Seção 3) |
|---|---|---|
| Prazo adicional sobre o prazo padrão da rota | **+2 dias úteis** | **+3 dias úteis** |
| Justificativa | Manuseio de carga pesada | Manuseio e roteirização de carga pesada |

**Impacto:** A v2 acrescenta 1 dia útil adicional ao prazo, com nova justificativa ("roteirização"). Um cliente cotado com base na v1 receberá uma expectativa de entrega 1 dia útil mais curta do que a praticada pela operação. Para rotas do Norte e Nordeste, onde os prazos base já são longos, esse dia a mais tem maior visibilidade e é fonte frequente de chamados de rastreamento.

**Classificação de risco:** 🟡 Médio — impacto em satisfação do cliente e volume de chamados.

---

### IC-04 — Regras de desconto por volume completamente reestruturadas

| Aspecto | v1 (Seção 4) | v2 (Seção 4) |
|---|---|---|
| Limiar para desconto | Mais de 10 fretes/mês | A partir de **8 fretes/mês** |
| Percentual de desconto faixa 1 | Não definido — negociação via Comercial | **5%** sobre o multiplicador regional |
| Limiar para desconto adicional | Não previsto | Acima de **15 fretes/mês** |
| Percentual de desconto faixa 2 | Não previsto | **10%** sobre o multiplicador regional |
| Base de aplicação | Não informada | Sobre o multiplicador regional |
| Descontos maiores | Negociados pelo Comercial em aditivo contratual | Aprovação da Diretoria Comercial |
| Natureza do desconto | Negociado (não automático) | **Automático** nas faixas definidas |

**Impacto:** A mudança é estrutural. A v1 não prevê desconto automático algum — tudo é negociação manual via Comercial com aditivo contratual. A v2 cria descontos automáticos em duas faixas. Clientes que atingem 8 fretes/mês têm direito garantido pela v2 a 5% de desconto, mas não pela v1. Se a área comercial ou o atendimento operar com a v1, esse benefício nunca será aplicado. Adicionalmente, a base de cálculo (sobre o multiplicador regional, não sobre o valor final) é informação ausente na v1, impossibilitando cálculo correto mesmo para negociações manuais.

**Classificação de risco:** 🔴 Alto — impacto financeiro direto para clientes elegíveis e risco contratual.

---

### IC-05 — Nota sobre PROC-043 com status divergente

| Aspecto | v1 (Seção 4) | v2 (Seção 4) |
|---|---|---|
| Referência à PROC-043 | "Cargas perigosas com peso acima de 500kg seguem tabela específica (PROC-043)" | Idem, acrescido de: **"Nota: a PROC-043 está em processo de revisão pelo Compliance e pode sofrer alterações"** |

**Impacto:** A v1 trata a PROC-043 como documento estável. A v2 sinaliza instabilidade normativa nessa área. Qualquer decisão de frete para cargas perigosas acima de 500 kg feita com base na v1 ignora esse risco. Colaboradores que usem apenas a v1 não têm ciência de que a PROC-043 pode estar desatualizada.

**Classificação de risco:** 🟠 Latente — risco regulatório e operacional em segmento de alta sensibilidade.

---

## 4. Divergências Estruturais

### DE-01 — Disposição transitória presente apenas na v2

A v2 inclui uma Seção 5 (Disposições Transitórias) ausente na v1:

> *"Chamados abertos antes de 01/12/2023 que ainda estejam em processamento devem usar os multiplicadores da versão anterior (PROC-042 v1). Chamados novos a partir de 01/12/2023 devem usar os multiplicadores desta versão."*

**Análise:** A existência dessa cláusula é o único indicador — ainda que implícito — de que a v2 foi concebida como substituta da v1. Porém, por não declarar explicitamente a revogação da v1 nem registrar isso no sistema de gestão documental, a disposição transitória perdeu validade prática após 01/12/2023 sem criar segurança jurídica sobre qual versão governa chamados subsequentes. Hoje (06/2026), a cláusula está operacionalmente expirada, mas a v1 permanece ativa no sistema.

---

### DE-02 — Justificativa de revisão presente apenas na v2

A v2 declara em seu objetivo: *"Os multiplicadores foram revisados para refletir os custos operacionais atualizados de cada região."* A v1 não possui qualquer declaração equivalente de contexto ou motivação.

**Análise:** A ausência de rastreabilidade na v1 impede que um colaborador, ao encontrar os dois documentos, compreenda a relação cronológica e causal entre eles sem leitura comparativa completa. Um sistema de gestão documental adequado exigiria que a v1 fosse marcada como "substituída por PROC-042-v2" e que a v2 declarasse explicitamente "revoga e substitui PROC-042-v1.0".

---

## 5. Efeito Combinado das Inconsistências de Cálculo

Para ilustrar o impacto financeiro da coexistência das versões, considere um cenário hipotético de carga com peso de 2.000 kg destinada ao Norte:

| Componente | v1 | v2 |
|---|---|---|
| Multiplicador regional (Norte) | 1,6 | 1,8 |
| Fator de peso (1.001–3.000 kg) | 1,2 | 1,15 |
| Produto (multiplicador × fator de peso) | **1,92** | **2,07** |
| Diferença sobre o valor base | — | **+7,8%** |

Para uma carga de 2.000 kg para o Sudeste:

| Componente | v1 | v2 |
|---|---|---|
| Multiplicador regional (Sudeste) | 1,0 | 1,1 |
| Fator de peso (1.001–3.000 kg) | 1,2 | 1,15 |
| Produto | **1,20** | **1,265** |
| Diferença sobre o valor base | — | **+5,4%** |

**Conclusão do cenário:** A v2 resulta em fretes mais altos na combinação mais comum (regiões com multiplicadores maiores e faixas de peso superiores), mas a magnitude varia por rota. Não há cenário em que as duas versões produzam o mesmo resultado para cargas acima de 1.000 kg.

---

## 6. Matriz Consolidada

| ID | Tipo | Tema | Versão de referência | Impacto em v1 se usada | Risco |
|---|---|---|---|---|---|
| IC-01 | Valores divergentes | Multiplicadores regionais (todas as regiões) | v2 | Frete subestimado em 7,1% a 12,5% por região | 🔴 Alto |
| IC-02 | Valores divergentes | Fatores de peso faixas ≥1.001 kg | v2 | Frete superestimado em 4,2% a 6,7% por faixa | 🔴 Alto |
| IC-03 | Valor divergente | Prazo adicional de manuseio | v2 | Prazo informado 1 dia útil menor que o real | 🟡 Médio |
| IC-04 | Regra reestruturada | Descontos por volume | v2 | Desconto automático não aplicado; clientes prejudicados | 🔴 Alto |
| IC-05 | Informação ausente | Status da PROC-043 | v2 | Risco regulatório em cargas perigosas ignorado | 🟠 Latente |
| DE-01 | Estrutural | Disposição transitória / relação entre versões | v2 | Hierarquia documental indefinida | 🔴 Alto |
| DE-02 | Estrutural | Rastreabilidade e motivação da revisão | v2 | Impossibilidade de escolha informada da versão | 🟡 Médio |

---

## 7. Recomendações

| Prioridade | Ação | Responsável Sugerido |
|---|---|---|
| 🔴 Imediata | Declarar formalmente a v2 como versão vigente e arquivar a v1 com status "revogado" no SharePoint | Diretoria Comercial + Gestão de Qualidade |
| 🔴 Imediata | Comunicar ao time de atendimento e ao Comercial que apenas a v2 deve ser utilizada para chamados a partir de 01/12/2023 | Coordenação de Atendimento + Comercial |
| 🔴 Imediata | Auditar cotações emitidas com multiplicadores da v1 após 01/12/2023 e avaliar necessidade de correção | Diretoria Comercial + Financeiro |
| 🟡 Curto prazo | Atualizar o FAQ-Atendimento com os parâmetros corretos da v2 (multiplicadores, fatores de peso, prazo, descontos) | Coordenação de Atendimento |
| 🟡 Curto prazo | Publicar versão revisada da PROC-043 e remover nota de instabilidade da v2 | Compliance + Operações |
| 🟢 Estrutural | Implementar controle de versão formal para documentos PROC e POL: campo "substitui", "substituído por" e status obrigatórios | Gestão de Qualidade / Compliance |

---

*Análise baseada exclusivamente no conteúdo integral dos documentos `PROC-042-v1.0` e `PROC-042-v2.0`. Os cenários de impacto financeiro são ilustrativos e baseados na fórmula definida nos próprios documentos. Validação com as áreas de Operações, Comercial e Compliance é recomendada antes de qualquer ação corretiva.*
