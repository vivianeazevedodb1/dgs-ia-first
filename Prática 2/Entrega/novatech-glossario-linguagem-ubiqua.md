# Glossário de Linguagem Ubíqua — NovaTech
**Fonte:** Anexo A — Documentação Simulada da NovaTech  
**Elaborado por:** Product Specialist  
**Data:** 05/06/2026  
**Versão:** 1.0

> Os termos abaixo foram extraídos diretamente dos documentos normativos, procedimentais e contratuais da NovaTech (POL-001, PROC-042, PROC-042-v2, SLA-2024 e FAQ-Atendimento). O significado oficial é o definido nos documentos de origem — termos com mais de uma definição entre documentos têm as divergências explicitadas.

---

## A

**ANTT**
Agência Nacional de Transportes Terrestres. Órgão regulador cujas resoluções classificam as cargas perigosas e definem a documentação obrigatória para transporte. Resolução de referência: ANTT nº 5.947/2021.
*Fonte: POL-001 Seção 3.2; PROC-042-v2 Seção 4*

**Avaria em trânsito**
Dano ocorrido à carga durante o transporte, caracterizando erro ou defeito imputável à NovaTech. Quando comprovada, a devolução é sem custo para o cliente. Processo distinto da devolução padrão — encaminhado ao Jurídico via sinistros@novatech.com.br (conforme FAQ) e não pela POL-001.
*Fonte: POL-001 Seção 3.5; FAQ item 38*

---

## C

**Cadeia de frio**
Conjunto de condições de temperatura controlada exigidas para conservação de cargas refrigeradas, conforme faixa especificada na nota fiscal. A ruptura da cadeia de frio ocorre quando a temperatura fica fora da faixa por mais de 30 minutos contínuos, conforme registro do sensor IoT. Cargas com ruptura confirmada não são elegíveis para devolução pelo processo padrão.
*Fonte: POL-001 Seção 3.2*

**Carga danificada em trânsito**
Mercadoria que sofreu avaria durante o transporte, antes da entrega. Tratamento distinto da devolução pós-entrega: o cliente deve registrar a ocorrência em até 48 horas após o recebimento, com fotos e laudo. Processo conduzido pelo Jurídico, não pelo atendimento padrão.
*Fonte: FAQ item 38 — nota: não há POL ou PROC formal sobre este processo*

**Carga perigosa**
Mercadoria classificada nas classes 1 a 6 da ANTT, conforme Resolução ANTT nº 5.947/2021. Classificação por classe: Classe 1 — explosivos; Classe 2 — gases; Classe 3 — líquidos inflamáveis; Classe 4 — sólidos inflamáveis; Classe 5 — oxidantes e peróxidos orgânicos; Classe 6 — substâncias tóxicas e infectantes. Não elegível para devolução pelo processo padrão. Para frete, segue tabela específica da PROC-043.
*Fonte: POL-001 Seção 3.2; PROC-042-v2 Seção 4*

**Carga refrigerada**
Mercadoria que requer temperatura controlada durante o transporte, com faixa especificada na nota fiscal e monitorada por sensor IoT. Não elegível para devolução pelo processo padrão quando houver ruptura da cadeia de frio.
*Fonte: POL-001 Seção 3.2*

**Centro de distribuição (CD)**
Local da NovaTech para onde a mercadoria devolvida deve ser recebida para que o reembolso ou crédito seja processado. O prazo de reembolso começa a contar a partir do recebimento da mercadoria no CD.
*Fonte: POL-001 Seção 3.3*

**Chamado**
Registro formal de uma solicitação ou ocorrência aberta pelo cliente no Portal do Cliente ou pelo atendente no sistema interno. Ponto de partida para o processo de devolução, rastreamento e abertura de sinistros. O timestamp de abertura é o marco de início da medição de SLA.
*Fonte: POL-001 Seção 3.3; SLA-2024 Seção 5*

**Coleta reversa**
Operação logística de retirada da mercadoria no endereço do cliente para retorno ao centro de distribuição da NovaTech, após aprovação do chamado de devolução. Prazo: até 2 dias úteis após a aprovação.
*Fonte: POL-001 Seção 3.3*

**CT-e (Conhecimento de Transporte Eletrônico)**
Documento fiscal eletrônico que formaliza o contrato de transporte entre a NovaTech e o cliente. Número obrigatório para abertura de chamado de devolução. Base para o cálculo de reembolso proporcional em devoluções parciais.
*Fonte: POL-001 Seções 3.3 e 3.4*

---

## D

**Desconto de volume**
Benefício aplicado automaticamente a clientes que realizam alto volume de fretes especiais no mesmo mês. Definição diverge entre versões da PROC-042:
- PROC-042 v1: aplicável a clientes com mais de 10 fretes especiais/mês; percentual e base de cálculo não definidos; negociação manual via Comercial com aditivo contratual.
- PROC-042-v2: a partir de 8 fretes especiais/mês → 5% de desconto sobre o multiplicador regional; acima de 15 fretes/mês → 10% de desconto; descontos maiores requerem aprovação da Diretoria Comercial.
*Fonte: PROC-042 Seção 4; PROC-042-v2 Seção 4*

**Devolução parcial**
Solicitação de devolução de volumes individuais de uma entrega que continha múltiplos volumes. Cada volume devolvido segue o mesmo procedimento da devolução padrão. O reembolso é calculado proporcionalmente ao peso ou valor do volume devolvido, conforme o CT-e.
*Fonte: POL-001 Seção 3.4*

**Devolução padrão**
Processo formal de devolução de mercadoria iniciado pelo cliente via Portal do Cliente, dentro de 7 dias úteis após o recebimento confirmado no tracking. Não aplicável a cargas perigosas, cargas refrigeradas com ruptura de cadeia de frio e cargas com lacre violado.
*Fonte: POL-001 Seções 3.1 e 3.3*

**Dias úteis**
Contagem de tempo excluindo sábados, domingos e feriados nacionais. Aplicável ao prazo de devolução (7 dias), triagem (4 horas), coleta reversa (2 dias), reembolso (5 dias), prazo adicional de frete especial e SLAs de atendimento. Para incidentes críticos de clientes Gold, o relógio de SLA não pausa fora do horário comercial.
*Fonte: POL-001 Seções 3.1 e 3.3; SLA-2024 Seções 2 e 5*

---

## F

**Fator de peso**
Multiplicador aplicado ao cálculo do frete especial conforme a faixa de peso da carga. Definição diverge entre versões:
- PROC-042 v1: 1,0 para 500–1.000 kg; 1,2 para 1.001–3.000 kg; 1,5 para acima de 3.000 kg.
- PROC-042-v2: 1,0 para 500–1.000 kg; 1,15 para 1.001–3.000 kg; 1,4 para acima de 3.000 kg.
*Fonte: PROC-042 Seção 2; PROC-042-v2 Seção 2*

**Frete especial**
Modalidade de frete aplicável a cargas com peso acima de 500 kg. Calculado pela fórmula: `Valor base × Multiplicador regional × Fator de peso`. Cargas perigosas acima de 500 kg seguem tabela específica (PROC-043), não a PROC-042.
*Fonte: PROC-042 Seção 1; PROC-042-v2 Seção 1*

**Frete reverso**
Custo do transporte de retorno da mercadoria ao centro de distribuição em caso de devolução por desistência do cliente (carga correta, sem defeito). Calculado com os mesmos multiplicadores do frete original.
*Fonte: POL-001 Seção 3.5*

---

## G

**Gerente de conta dedicado**
Profissional da NovaTech designado exclusivamente para um cliente Gold. Benefício exclusivo do tier Gold; clientes Silver e Standard não têm gerente dedicado.
*Fonte: SLA-2024 Seção 2*

**Gestão de Riscos**
Setor interno da NovaTech responsável pelo tratamento individual de solicitações envolvendo cargas perigosas (que não são elegíveis para devolução padrão) e outras situações de risco. Contato: ramal 4500.
*Fonte: POL-001 Seção 3.2; FAQ item 3*

**Gold**
Tier de cliente de maior nível na NovaTech. Critério de elegibilidade: contrato anual acima de R$ 500.000 OU mais de 200 operações por mês. Revisão semestral. SLAs: resposta em até 2 horas úteis / resolução em até 24 horas úteis (chamados gerais); resposta em até 30 minutos / resolução em até 4 horas (incidentes críticos). Único tier com gerente de conta dedicado e relógio de SLA sem pausa para incidentes críticos.
*Fonte: SLA-2024 Seções 1 e 2*

---

## I

**Incidente crítico**
Chamado classificado como crítico quando atende a ao menos um dos critérios: (a) carga com valor declarado acima de R$ 100.000 com status desconhecido há mais de 6 horas; (b) carga perigosa com qualquer irregularidade de documentação ou rastreamento; (c) mais de 5 chamados do mesmo cliente nas últimas 24 horas sobre o mesmo problema; (d) qualquer situação que envolva risco à segurança de pessoas. SLAs de incidentes críticos são menores e, para clientes Gold, o relógio não pausa fora do horário comercial.
*Fonte: SLA-2024 Seção 3*

---

## L

**Lacre de segurança**
Dispositivo aplicado à carga para garantir integridade durante o transporte. Carga com lacre violado não é elegível para devolução padrão, salvo quando a violação foi documentada no ato da entrega com assinatura do motorista e do recebedor.
*Fonte: POL-001 Seção 3.2*

---

## M

**Multiplicador regional**
Fator aplicado ao cálculo do frete especial conforme a região de destino da carga. Definição diverge entre versões:
- PROC-042 v1: Sul 1,2 / Sudeste 1,0 / Centro-Oeste 1,3 / Nordeste 1,4 / Norte 1,6.
- PROC-042-v2: Sul 1,3 / Sudeste 1,1 / Centro-Oeste 1,4 / Nordeste 1,5 / Norte 1,8.
*Fonte: PROC-042 Seção 2.1; PROC-042-v2 Seção 2.1*

---

## O

**Operações/mês**
Unidade de medida do volume de atividade de um cliente, usada como critério alternativo de elegibilidade para classificação de tier. Gold: mais de 200 operações/mês; Silver: entre 50 e 200 operações/mês.
*Fonte: SLA-2024 Seção 1*

---

## P

**Penalidade por descumprimento de SLA**
Consequência contratual para violações de SLA. Escala mensal: primeira violação → registro interno sem impacto contratual; segunda violação → crédito de 5% sobre o valor do frete do chamado afetado; terceira violação ou mais → crédito de 10% + reunião obrigatória com gerente de conta (Gold) ou gerente de operações (Silver/Standard).
*Fonte: SLA-2024 Seção 4*

**Portal do Cliente**
Plataforma digital da NovaTech (portal.novatech.com.br) utilizada pelo cliente para abertura de chamados, incluindo solicitações de devolução de mercadoria.
*Fonte: POL-001 Seção 3.3*

**Prazo adicional de manuseio**
Dias úteis acrescidos ao prazo padrão de entrega para cargas que se enquadram no frete especial. Definição diverge entre versões:
- PROC-042 v1: +2 dias úteis para manuseio de carga pesada.
- PROC-042-v2: +3 dias úteis para manuseio e roteirização de carga pesada.
*Fonte: PROC-042 Seção 3; PROC-042-v2 Seção 3*

**PROC-043**
Procedimento de Frete de Cargas Perigosas. Documento referenciado pela PROC-042 e PROC-042-v2 como tabela específica para cálculo de frete de cargas perigosas acima de 500 kg. Status: em processo de revisão pelo Compliance (conforme PROC-042-v2 Seção 4). Documento não disponível na base atual.
*Fonte: PROC-042 Seção 4; PROC-042-v2 Seção 4*

**PROC-088**
Procedimento de Interceptação de Carga. Documento referenciado pela POL-001 como norma aplicável a mercadorias ainda em trânsito (fora do escopo da política de devolução). Documento não disponível na base atual.
*Fonte: POL-001 Seção 2*

---

## R

**Reembolso / Crédito**
Restituição do valor pago pelo cliente após devolução aprovada. Prazo: até 5 dias úteis após o recebimento da mercadoria devolvida no centro de distribuição. Pode ser em dinheiro (reembolso) ou abatimento em fatura futura (crédito).
*Fonte: POL-001 Seção 3.3*

**Relatório mensal de performance**
Documento periódico com métricas de atendimento e operação entregue pela NovaTech ao cliente. Gold: relatório detalhado, entregue mensalmente. Silver: relatório resumido, mensal. Standard: sob demanda.
*Fonte: SLA-2024 Seção 2*

---

## S

**Seguro de carga**
Serviço adicional oferecido pela NovaTech sobre o valor declarado da mercadoria. Percentuais indicados no FAQ (não formalizados em normativo): 0,3% para cargas padrão e 0,8% para cargas perigosas, válidos para contratos a partir de 2023. Contratos anteriores podem ter percentuais diferentes.
*Fonte: FAQ item 22 — nota: não há POL ou PROC formal sobre seguro de carga*

**Silver**
Tier intermediário de cliente na NovaTech. Critério de elegibilidade: contrato anual entre R$ 100.000 e R$ 500.000 OU entre 50 e 200 operações por mês. Revisão semestral. SLAs: resposta em até 4 horas úteis / resolução em até 48 horas úteis (chamados gerais); resposta em até 1 hora / resolução em até 8 horas (incidentes críticos).
*Fonte: SLA-2024 Seções 1 e 2*

**SLA de resolução**
Tempo máximo para resolução efetiva de um chamado. Distinto do SLA de resposta. Gold: 24 horas úteis (geral) / 4 horas (crítico); Silver: 48 horas úteis (geral) / 8 horas (crítico); Standard: 72 horas úteis (geral) / 24 horas (crítico).
*Fonte: SLA-2024 Seção 2; FAQ item 41*

**SLA de resposta**
Tempo máximo para o primeiro retorno ao cliente após a abertura do chamado — mesmo que seja apenas uma confirmação de recebimento. Gold: 2 horas úteis (geral) / 30 minutos (crítico); Silver: 4 horas úteis (geral) / 1 hora (crítico); Standard: 8 horas úteis (geral) / 2 horas (crítico).
*Fonte: SLA-2024 Seção 2; FAQ item 41*

**Standard**
Tier base de cliente na NovaTech. Critério de elegibilidade: todos os demais clientes que não se enquadram nos critérios de Gold ou Silver. Revisão anual. SLAs: resposta em até 8 horas úteis / resolução em até 72 horas úteis (chamados gerais); resposta em até 2 horas / resolução em até 24 horas (incidentes críticos).
*Fonte: SLA-2024 Seções 1 e 2*

---

## T

**Tabela mensal de fretes**
Planilha de referência com as tarifas base de frete, atualizada mensalmente pela área Comercial. Localização: `\\novatech-fs\comercial\tabelas\frete-base-AAAAMM.xlsx`. Utilizada como "Valor base" na fórmula de cálculo do frete especial.
*Fonte: PROC-042 Seção 2; PROC-042-v2 Seção 2*

**Tier**
Classificação do cliente na NovaTech com base em volume e valor contratual, determinando os SLAs, benefícios e obrigações contratuais aplicáveis. Existem exatamente três tiers: Gold, Silver e Standard. Não existe tier Platinum ou qualquer outra denominação.
*Fonte: SLA-2024 Seção 1; FAQ item 15*

**Tier Platinum**
Denominação inexistente na NovaTech. Pode gerar confusão por referência a programas de fidelidade de outras transportadoras ou ao programa de fidelidade da própria NovaTech descontinuado em 2022.
*Fonte: SLA-2024 Seção 1; FAQ item 15*

**Timestamp de abertura do chamado**
Marco de início da contagem de SLA, registrado automaticamente pelo sistema de chamados (Azure DevOps) no momento em que o chamado é criado.
*Fonte: SLA-2024 Seção 5*

**Triagem**
Etapa inicial do processo de devolução realizada pelo time de atendimento após abertura do chamado pelo cliente. Consiste em verificar a elegibilidade (tipo de carga, prazo e documentação). Prazo: 4 horas úteis a partir da abertura do chamado.
*Fonte: POL-001 Seção 3.3*

---

## V

**Valor base**
Tarifa de frete publicada mensalmente na tabela de fretes da NovaTech. Primeiro fator da fórmula de cálculo do frete especial: `Valor base × Multiplicador regional × Fator de peso`.
*Fonte: PROC-042 Seção 2; PROC-042-v2 Seção 2*

**Violação de SLA**
Ocorrência de atendimento em que os prazos de resposta ou resolução definidos contratualmente para o tier do cliente foram descumpridos. Contada mensalmente por cliente para fins de aplicação de penalidades.
*Fonte: SLA-2024 Seção 4*

**Volume**
Unidade individual de uma entrega quando a carga é composta por múltiplos itens ou embalagens. Em devoluções parciais, o cliente pode devolver volumes individuais, e o reembolso é proporcional ao peso ou valor do volume devolvido conforme o CT-e.
*Fonte: POL-001 Seção 3.4*

---

## Termos com Definição Divergente entre Documentos

A tabela abaixo consolida os termos cujo significado varia conforme o documento consultado. Atenção especial necessária no pipeline de RAG ao recuperar chunks sobre esses temas.

| Termo | PROC-042 v1 | PROC-042-v2 | Impacto |
|---|---|---|---|
| Multiplicador regional — Sul | 1,2 | 1,3 | +8,3% no cálculo do frete |
| Multiplicador regional — Sudeste | 1,0 | 1,1 | +10,0% no cálculo do frete |
| Multiplicador regional — Centro-Oeste | 1,3 | 1,4 | +7,7% no cálculo do frete |
| Multiplicador regional — Nordeste | 1,4 | 1,5 | +7,1% no cálculo do frete |
| Multiplicador regional — Norte | 1,6 | 1,8 | +12,5% no cálculo do frete |
| Fator de peso (1.001–3.000 kg) | 1,2 | 1,15 | −4,2% no cálculo do frete |
| Fator de peso (acima de 3.000 kg) | 1,5 | 1,4 | −6,7% no cálculo do frete |
| Prazo adicional de manuseio | +2 dias úteis | +3 dias úteis | 1 dia útil a mais no prazo de entrega |
| Desconto de volume — limiar faixa 1 | >10 fretes/mês (negociação manual) | ≥8 fretes/mês (5% automático) | Clientes entre 8 e 10 fretes podem ter direito não reconhecido |
| Desconto de volume — limiar faixa 2 | Não previsto | >15 fretes/mês (10% automático) | Benefício ausente na v1 |

---

*Glossário extraído exclusivamente do conteúdo literal do Anexo A. Termos marcados como "não há POL ou PROC formal" indicam conceitos presentes apenas no FAQ-Atendimento (documento informal não validado), sem respaldo em normativo oficial. Recomenda-se validação com as áreas responsáveis antes de uso em contextos contratuais ou operacionais.*
