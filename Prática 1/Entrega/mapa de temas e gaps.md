# Análise Documental — NovaTech Logística
**Elaborado por:** Product Specialist  
**Data:** 05/06/2026  
**Finalidade:** Mapeamento de cobertura temática e identificação de gaps na base documental

---

## 1. Inventário de Documentos

### 1.1 FAQ-Atendimento — Perguntas Frequentes do Time de Suporte

| Campo | Valor |
|---|---|
| **Versão** | Não controlada |
| **Tema** | Suporte operacional / conhecimento tácito do atendimento |
| **Responsável** | Nenhum responsável formal (mantido informalmente pelo time) |
| **Classificação** | Documento informal — NÃO validado por Compliance ou Operações |
| **Escopo** | 47 perguntas práticas do cotidiano de atendimento ao cliente |

**Resumo:**  
Documento colaborativo criado organicamente ao longo de 2 anos pelo time de atendimento, sem versionamento formal nem processo de revisão. Cobre situações recorrentes como devolução de cargas perigosas, cálculo de frete especial, tiers de clientes, seguro de carga, rastreamento e autonomia de desconto. As respostas refletem experiência prática, mas apresentam risco de desatualização em relação às normas vigentes (POL, PROC, SLA). Não há indicação de data de criação por item nem rastreabilidade de quem contribuiu com cada resposta. A ausência de validação formal cria risco de orientações conflitantes com a documentação normativa, especialmente em relação à PROC-042 (versões coexistentes) e à POL-001. O documento reconhece explicitamente sua limitação, mas não estabelece mecanismo de atualização.

---

### 1.2 POL-001 — Política de Devolução de Mercadorias

| Campo | Valor |
|---|---|
| **Versão** | 3.1 |
| **Tema** | Devolução de mercadorias pós-entrega |
| **Responsável** | Diretoria de Operações |
| **Classificação** | Documento normativo — uso obrigatório pelo time de atendimento |
| **Escopo** | Todas as devoluções após entrega; exclui mercadorias em trânsito (ver PROC-088) |

**Resumo:**  
Política normativa estruturada, com versão e data de atualização definidas (15/01/2024). Define prazo geral de 7 dias úteis para solicitação de devolução, categorias de exceção (cargas perigosas classes 1–6, cargas refrigeradas com ruptura de cadeia de frio, lacre violado), procedimento de abertura de chamado via portal, prazos internos de triagem e reembolso, regras para devoluções parciais e distribuição de custos conforme responsabilidade. Referencia a PROC-088 para cargas em trânsito, mas esse documento não está presente no conjunto analisado. Não aborda o processo de carga danificada em trânsito (mencionado no FAQ), que parece seguir fluxo paralelo via Jurídico. Não há menção a SLAs específicos para o fluxo de devolução nem integração explícita com a tabela SLA-2024.

---

### 1.3 PROC-042 — Procedimento de Cálculo de Frete Especial (v1.0)

| Campo | Valor |
|---|---|
| **Versão** | 1.0 |
| **Tema** | Cálculo de frete especial para cargas acima de 500 kg |
| **Responsável** | Diretoria Comercial |
| **Classificação** | Procedimento operacional (status de vigência indefinido) |
| **Escopo** | Cargas acima de 500 kg; exclui cargas perigosas (remete à PROC-043) |

**Resumo:**  
Primeira versão do procedimento de frete especial, emitida em março/2023. Define fórmula de cálculo baseada em valor base, multiplicador regional e fator de peso. Os multiplicadores regionais são inferiores aos da versão v2 em todas as regiões. O fator de peso para a faixa intermediária (1.001–3.000 kg) é 1,2 e para acima de 3.000 kg é 1,5. O prazo adicional para manuseio é de +2 dias úteis. A condição de desconto por volume prevê apenas negociação via Comercial, sem percentuais automáticos. O documento não possui indicação formal de vigência ou obsolescência no sistema da NovaTech, coexistindo com a versão v2 sem hierarquia clara — situação de risco operacional e comercial. A PROC-043 referenciada não está no conjunto analisado.

---

### 1.4 PROC-042-v2 — Procedimento de Cálculo de Frete Especial Revisado (v2.0)

| Campo | Valor |
|---|---|
| **Versão** | 2.0 |
| **Tema** | Cálculo de frete especial para cargas acima de 500 kg (revisado) |
| **Responsável** | Diretoria Comercial |
| **Classificação** | Procedimento operacional revisado (status de vigência indefinido) |
| **Escopo** | Cargas acima de 500 kg; exclui cargas perigosas (PROC-043 em revisão pelo Compliance) |

**Resumo:**  
Versão revisada da PROC-042, emitida em novembro/2023 com multiplicadores regionais atualizados para refletir custos operacionais mais recentes — todos superiores à v1. O fator de peso para a faixa intermediária cai para 1,15 e para acima de 3.000 kg para 1,4. O prazo adicional de manuseio sobe para +3 dias úteis. Introduz descontos automáticos por volume (5% a partir de 8 fretes/mês; 10% acima de 15 fretes/mês), divergindo da v1 que exigia negociação manual a partir de 10 fretes/mês. Inclui disposição transitória para chamados abertos antes de 01/12/2023. Assim como a v1, não possui indicação formal de que substitui o documento anterior no sistema. A nota sobre PROC-043 "em processo de revisão pelo Compliance" é um sinal adicional de instabilidade normativa na área de cargas perigosas.

---

### 1.5 SLA-2024 — Tabela de SLA por Tipo de Cliente

| Campo | Valor |
|---|---|
| **Versão** | 2024.1 |
| **Tema** | Acordos de nível de serviço por tier de cliente |
| **Responsável** | Diretoria Comercial + Diretoria de Operações |
| **Classificação** | Documento contratual — compromissos formais com o cliente |
| **Escopo** | Todos os clientes NovaTech (tiers Gold, Silver e Standard) |

**Resumo:**  
Documento contratual bem estruturado, com responsabilidade dual (Comercial + Operações) e atualização em janeiro/2024. Define três tiers de clientes (Gold, Silver, Standard) com critérios objetivos de elegibilidade por valor de contrato ou volume de operações. Estabelece SLAs distintos para chamados gerais e incidentes críticos, com critérios claros de classificação de criticidade. Inclui regras de penalidade por descumprimento escalonadas por recorrência mensal e especifica a ferramenta de medição (Azure DevOps). Confirma explicitamente a inexistência do tier "Platinum", alinhado com o FAQ. Não aborda SLAs específicos para o processo de devolução (POL-001) nem para o fluxo de cargas danificadas. A disponibilidade de portal não diferencia entre horário comercial e fora dele para clientes Standard.

---

## 2. Mapa de Temas Cobertos

| Tema | Documento(s) | Status de Cobertura |
|---|---|---|
| Devolução pós-entrega (prazo, procedimento, custos) | POL-001 | ✅ Coberto — documento normativo vigente |
| Categorias inelegíveis para devolução padrão | POL-001 | ✅ Coberto |
| Carga em trânsito (interceptação) | POL-001 (remete PROC-088) | ⚠️ Parcial — PROC-088 ausente no conjunto |
| Carga danificada em trânsito | FAQ (item 38) | ⚠️ Parcial — apenas no FAQ informal; sem normativo |
| Cálculo de frete especial | PROC-042 v1 + v2 | ⚠️ Conflitante — duas versões sem hierarquia formal |
| Desconto por volume em frete especial | PROC-042 v1 e v2 | ⚠️ Conflitante — regras divergentes entre versões |
| Frete para cargas perigosas | PROC-042 v1 e v2 (remete PROC-043) | ❌ Não coberto — PROC-043 ausente e em revisão |
| Tiers de clientes e elegibilidade | SLA-2024 | ✅ Coberto — documento contratual vigente |
| SLAs de resposta e resolução | SLA-2024 + FAQ (item 41) | ✅ Coberto — alinhados entre si |
| Penalidades por descumprimento de SLA | SLA-2024 | ✅ Coberto |
| Incidentes críticos — definição e SLA | SLA-2024 | ✅ Coberto |
| Seguro de carga | FAQ (item 22) | ❌ Não coberto — sem normativo formal |
| Autonomia do atendente para desconto | FAQ (item 45) | ⚠️ Parcial — apenas no FAQ; sem normativo |
| Carga perigosa + frete expresso | FAQ (item 32) | ⚠️ Parcial — apenas no FAQ informal |
| Rastreamento e atrasos em rota | FAQ (item 27) | ⚠️ Parcial — apenas no FAQ; sem normativo |

---

## 3. Hipóteses de Gaps

### Gap 1 — Documentos referenciados ausentes no conjunto
Os documentos **PROC-088** (Interceptação de Carga) e **PROC-043** (Frete de Cargas Perigosas) são referenciados por POL-001 e PROC-042 respectivamente, mas não estão disponíveis para análise. A PROC-043 encontra-se adicionalmente em estado de revisão pelo Compliance, gerando zona cinzenta operacional para um dos segmentos de maior risco regulatório da NovaTech.

### Gap 2 — Coexistência de versões conflitantes da PROC-042 sem hierarquia formal
As versões v1 e v2 da PROC-042 divergem em multiplicadores regionais, fatores de peso, prazo de manuseio e regras de desconto por volume. Nenhuma das duas possui indicação formal de vigência ou obsolescência no sistema (SharePoint). Isso cria risco direto de erro de cotação, litígios contratuais com clientes e inconsistência entre atendentes. A disposição transitória da v2 (chamados antes de 01/12/2023 usam v1) já deveria ter expirado operacionalmente, mas não há confirmação de encerramento do período de transição.

### Gap 3 — Ausência de normativo para carga danificada em trânsito
O FAQ descreve um processo para carga danificada (registro em 48h, fotos, encaminhamento para `sinistros@novatech.com.br`), mas não existe um documento normativo que formalize esse fluxo, defina prazos de resposta do Jurídico, critérios de apuração de responsabilidade ou condições de reembolso integral. Esse tema possui alto impacto financeiro e reputacional.

### Gap 4 — Seguro de carga sem cobertura normativa
O FAQ (item 22) menciona percentuais de seguro (0,3% para carga padrão; 0,8% para perigosa) e diferencia contratos pré e pós-2023, mas não existe nenhum documento normativo formal sobre a oferta, condições ou processo de acionamento do seguro de carga da NovaTech.

### Gap 5 — FAQ sem processo de atualização e validação
O FAQ cobre temas críticos ausentes nos normativos (seguro, carga danificada, carga perigosa expressa), mas é mantido informalmente sem responsável designado, sem versionamento e sem validação por Compliance ou Operações. Isso cria dependência de conhecimento tácito e risco de respostas divergentes ao cliente conforme o atendente.

### Gap 6 — SLA do processo de devolução não está mapeado
A POL-001 define prazos internos para triagem (4h), agendamento de coleta (2 dias úteis) e processamento de reembolso (5 dias úteis), mas esses prazos não estão incorporados à tabela SLA-2024 como compromissos diferenciados por tier de cliente. Um cliente Gold poderia razoavelmente esperar SLA de devolução diferenciado sem ter isso formalizado.

### Gap 7 — Autonomia operacional do atendente não está documentada formalmente
Tanto a questão de descontos (FAQ item 45) quanto autorizações de exceção (ex.: devolução de carga perigosa via Gestão de Riscos) são descritas apenas no FAQ. Não existe um normativo de matriz de alçada ou delegação de autoridade que defina formalmente o que o atendente pode ou não fazer sem escalar.

---

## 4. Recomendações de Priorização

| Prioridade | Ação Recomendada |
|---|---|
| 🔴 Alta | Formalizar hierarquia entre PROC-042 v1 e v2; arquivar versão obsoleta com registro formal |
| 🔴 Alta | Criar normativo para carga danificada em trânsito (substitui/complementa FAQ item 38) |
| 🔴 Alta | Disponibilizar e revisar PROC-043 (Frete de Cargas Perigosas) — área de risco regulatório |
| 🟡 Média | Criar política formal de seguro de carga (substitui FAQ item 22) |
| 🟡 Média | Incorporar SLAs de devolução à tabela SLA-2024 por tier de cliente |
| 🟡 Média | Estabelecer responsável formal, ciclo de revisão e processo de validação para o FAQ |
| 🟢 Baixa | Documentar PROC-088 (Interceptação de Carga) se ainda não existir formalmente |
| 🟢 Baixa | Criar normativo de matriz de alçada do atendimento ao cliente |

---

*Documento gerado com base exclusivamente nos metadados, títulos e conteúdo dos 5 arquivos fornecidos. Análise de gaps é baseada em hipóteses derivadas das lacunas observadas — recomenda-se validação com as áreas de Operações, Comercial e Compliance.*
