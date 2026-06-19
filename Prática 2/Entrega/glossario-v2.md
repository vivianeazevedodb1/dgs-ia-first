# Glossário de Linguagem Ubíqua — NovaTech Assistant
**Versão:** 2.0 (substitui v1.0 de 05/06/2026)  
**Data:** 05/06/2026  
**Elaborado por:** Product Specialist  
**Alterações em relação à v1.0:** Adicionada Seção II — Termos do Sistema (resolve AMB-10); adicionado "Turno" (resolve AMB-07); padronizado uso de "tier" minúsculo (resolve AMB-15); adicionados tipos de ambiguidade (resolve AMB-02).

> Este glossário tem duas seções: **Termos de Negócio** (extraídos dos documentos da NovaTech) e **Termos do Sistema** (vocabulário interno do assistente de IA). Ambas as seções fazem parte da linguagem ubíqua do projeto — toda comunicação entre produto, desenvolvimento e QA deve usar esses termos com as definições exatas abaixo.

---

## SEÇÃO I — Termos de Negócio

*Extraídos dos documentos normativos, procedimentais e contratuais da NovaTech. O significado oficial é o definido nos documentos de origem.*

---

### A

**ANTT**  
Agência Nacional de Transportes Terrestres. Órgão regulador cujas resoluções classificam as cargas perigosas e definem a documentação obrigatória para transporte. Resolução de referência: ANTT nº 5.947/2021.  
*Fonte: POL-001 Seção 3.2; PROC-042-v2 Seção 4*

**Avaria em trânsito**  
Dano ocorrido à carga durante o transporte, caracterizando erro ou defeito imputável à NovaTech. Quando comprovada, a devolução é sem custo para o cliente. Processo distinto da devolução padrão — encaminhado ao Jurídico via sinistros@novatech.com.br (conforme FAQ) e não pela POL-001.  
*Fonte: POL-001 Seção 3.5; FAQ item 38 — nota: contato informal, não formalizado em normativo*

---

### C

**Cadeia de frio**  
Conjunto de condições de temperatura controlada exigidas para conservação de cargas refrigeradas, conforme faixa especificada na nota fiscal. A ruptura ocorre quando a temperatura fica fora da faixa por mais de 30 minutos contínuos, conforme registro do sensor IoT. Cargas com ruptura confirmada não são elegíveis para devolução pelo processo padrão.  
*Fonte: POL-001 Seção 3.2*

**Carga danificada em trânsito**  
Mercadoria que sofreu avaria durante o transporte, antes da entrega. Tratamento distinto da devolução pós-entrega: o cliente deve registrar a ocorrência em até 48 horas após o recebimento, com fotos e laudo. Processo conduzido pelo Jurídico, não pelo atendimento padrão.  
*Fonte: FAQ item 38 — nota: não há POL ou PROC formal sobre este processo (Gap 3 mapeado)*

**Carga perigosa**  
Mercadoria classificada nas classes 1 a 6 da ANTT, conforme Resolução ANTT nº 5.947/2021. Classe 1 — explosivos; Classe 2 — gases; Classe 3 — líquidos inflamáveis; Classe 4 — sólidos inflamáveis; Classe 5 — oxidantes e peróxidos orgânicos; Classe 6 — substâncias tóxicas e infectantes. Não elegível para devolução padrão. Para frete, segue PROC-043 (em revisão).  
*Fonte: POL-001 Seção 3.2; PROC-042-v2 Seção 4*

**Carga refrigerada**  
Mercadoria que requer temperatura controlada durante o transporte, com faixa especificada na nota fiscal e monitorada por sensor IoT. Não elegível para devolução padrão quando houver ruptura da cadeia de frio.  
*Fonte: POL-001 Seção 3.2*

**Centro de distribuição (CD)**  
Local da NovaTech para onde a mercadoria devolvida deve ser recebida para que o reembolso ou crédito seja processado. O prazo de reembolso começa a contar a partir do recebimento no CD.  
*Fonte: POL-001 Seção 3.3*

**Chamado**  
Registro formal de uma solicitação ou ocorrência aberta pelo cliente no Portal do Cliente ou pelo atendente no sistema interno. Ponto de partida para devolução, rastreamento e sinistros. O timestamp de abertura é o marco de início da medição de SLA.  
*Fonte: POL-001 Seção 3.3; SLA-2024 Seção 5*

**Coleta reversa**  
Operação logística de retirada da mercadoria no endereço do cliente para retorno ao CD da NovaTech, após aprovação do chamado de devolução. Prazo: até 2 dias úteis após aprovação.  
*Fonte: POL-001 Seção 3.3*

**CT-e (Conhecimento de Transporte Eletrônico)**  
Documento fiscal eletrônico que formaliza o contrato de transporte entre a NovaTech e o cliente. Número obrigatório para abertura de chamado de devolução. Base para o cálculo de reembolso proporcional em devoluções parciais.  
*Fonte: POL-001 Seções 3.3 e 3.4*

---

### D

**Desconto de volume**  
Benefício aplicado a clientes com alto volume de fretes especiais no mesmo mês. Definição vigente (PROC-042-v2): a partir de 8 fretes especiais/mês → 5% de desconto sobre o multiplicador regional; acima de 15 fretes/mês → 10% de desconto. Descontos maiores requerem aprovação da Diretoria Comercial.  
*Fonte vigente: PROC-042-v2 Seção 4. Atenção: PROC-042-v1 usa limiar diferente (>10 fretes/mês, sem percentual automático) — versão revogada.*

**Devolução parcial**  
Solicitação de devolução de volumes individuais de uma entrega com múltiplos volumes. Cada volume devolvido segue o procedimento padrão. O reembolso é proporcional ao peso ou valor do volume conforme o CT-e.  
*Fonte: POL-001 Seção 3.4*

**Devolução padrão**  
Processo formal de devolução de mercadoria iniciado pelo cliente via Portal do Cliente, dentro de 7 dias úteis após o recebimento confirmado no tracking. Não aplicável a cargas perigosas, cargas refrigeradas com ruptura de cadeia de frio e cargas com lacre violado.  
*Fonte: POL-001 Seções 3.1 e 3.3*

**Dias úteis**  
Contagem de tempo excluindo sábados, domingos e feriados nacionais. Aplicável ao prazo de devolução (7 dias), triagem (4 horas), coleta reversa (2 dias), reembolso (5 dias), prazo adicional de frete especial e SLAs de atendimento. Para incidentes críticos de clientes tier Gold, o relógio de SLA não pausa fora do horário comercial.  
*Fonte: POL-001 Seções 3.1 e 3.3; SLA-2024 Seções 2 e 5*

---

### F

**Fator de peso**  
Multiplicador aplicado ao cálculo do frete especial conforme a faixa de peso da carga. Valores vigentes (PROC-042-v2): 1,0 para 500–1.000 kg; 1,15 para 1.001–3.000 kg; 1,4 para acima de 3.000 kg.  
*Fonte vigente: PROC-042-v2 Seção 2. Atenção: PROC-042-v1 usa fatores diferentes (1,2 e 1,5) — versão revogada.*

**Frete especial**  
Modalidade de frete aplicável a cargas com peso acima de 500 kg. Fórmula: `Valor base × Multiplicador regional × Fator de peso`. Cargas perigosas acima de 500 kg seguem PROC-043 (não a PROC-042).  
*Fonte: PROC-042-v2 Seção 1*

**Frete reverso**  
Custo do transporte de retorno da mercadoria ao CD em caso de devolução por desistência do cliente (carga correta, sem defeito). Calculado com os mesmos multiplicadores do frete original.  
*Fonte: POL-001 Seção 3.5*

---

### G

**Gerente de conta dedicado**  
Profissional da NovaTech designado exclusivamente para um cliente tier Gold. Benefício exclusivo do tier Gold; tiers Silver e Standard não têm gerente dedicado.  
*Fonte: SLA-2024 Seção 2*

**Gestão de Riscos**  
Setor interno da NovaTech responsável pelo tratamento individual de solicitações envolvendo cargas perigosas e outras situações de risco. Contato: ramal 4500 (referenciado na POL-001 — contato formal).  
*Fonte: POL-001 Seção 3.2*

---

### I

**Incidente crítico**  
Chamado classificado como crítico quando atende a ao menos um dos critérios: (a) carga com valor declarado acima de R$ 100.000 com status desconhecido há mais de 6 horas; (b) carga perigosa com qualquer irregularidade de documentação ou rastreamento; (c) mais de 5 chamados do mesmo cliente nas últimas 24 horas sobre o mesmo problema; (d) qualquer situação que envolva risco à segurança de pessoas. SLAs de incidentes críticos são menores e, para clientes tier Gold, o relógio não pausa fora do horário comercial.  
*Fonte: SLA-2024 Seção 3*

---

### L

**Lacre de segurança**  
Dispositivo aplicado à carga para garantir integridade durante o transporte. Carga com lacre violado não é elegível para devolução padrão, salvo quando a violação foi documentada no ato da entrega com assinatura do motorista e do recebedor.  
*Fonte: POL-001 Seção 3.2*

---

### M

**Multiplicador regional**  
Fator aplicado ao cálculo do frete especial conforme a região de destino da carga. Valores vigentes (PROC-042-v2): Sul 1,3 / Sudeste 1,1 / Centro-Oeste 1,4 / Nordeste 1,5 / Norte 1,8.  
*Fonte vigente: PROC-042-v2 Seção 2.1. Atenção: PROC-042-v1 usa valores inferiores — versão revogada.*

---

### P

**Penalidade por descumprimento de SLA**  
Consequência contratual para violações de SLA, escalonada mensalmente por cliente: primeira violação → registro interno sem impacto contratual; segunda violação → crédito de 5% sobre o valor do frete do chamado afetado; terceira ou mais → crédito de 10% + reunião obrigatória com gerente de conta (tier Gold) ou gerente de operações (tiers Silver/Standard).  
*Fonte: SLA-2024 Seção 4*

**Portal do Cliente**  
Plataforma digital da NovaTech (portal.novatech.com.br) utilizada pelo cliente para abertura de chamados, incluindo solicitações de devolução.  
*Fonte: POL-001 Seção 3.3*

**Prazo adicional de manuseio**  
Dias úteis acrescidos ao prazo padrão de entrega para cargas que se enquadram no frete especial. Valor vigente (PROC-042-v2): +3 dias úteis para manuseio e roteirização de carga pesada.  
*Fonte vigente: PROC-042-v2 Seção 3. Atenção: PROC-042-v1 define +2 dias úteis — versão revogada.*

---

### R

**Reembolso / Crédito**  
Restituição do valor pago pelo cliente após devolução aprovada. Prazo: até 5 dias úteis após o recebimento da mercadoria devolvida no CD.  
*Fonte: POL-001 Seção 3.3*

**Relatório mensal de performance**  
Documento periódico com métricas de atendimento entregue pela NovaTech ao cliente. tier Gold: relatório detalhado mensal. tier Silver: relatório resumido mensal. tier Standard: sob demanda.  
*Fonte: SLA-2024 Seção 2*

---

### S

**Seguro de carga**  
Serviço adicional sobre o valor declarado da mercadoria. Percentuais indicados no FAQ (não formalizados em normativo): 0,3% para cargas padrão e 0,8% para cargas perigosas, para contratos a partir de 2023. **Tema fora do escopo do assistente na fase 1 — sem normativo formal indexado.**  
*Fonte: FAQ item 22 — não há POL ou PROC formal*

**SLA de resolução**  
Tempo máximo para resolução efetiva de um chamado, distinto do SLA de resposta. tier Gold: 24 horas úteis (geral) / 4 horas (crítico); tier Silver: 48 horas úteis (geral) / 8 horas (crítico); tier Standard: 72 horas úteis (geral) / 24 horas (crítico).  
*Fonte: SLA-2024 Seção 2*

**SLA de resposta**  
Tempo máximo para o primeiro retorno ao cliente após abertura do chamado. tier Gold: 2 horas úteis (geral) / 30 minutos (crítico); tier Silver: 4 horas úteis (geral) / 1 hora (crítico); tier Standard: 8 horas úteis (geral) / 2 horas (crítico).  
*Fonte: SLA-2024 Seção 2*

---

### T

**Tabela mensal de fretes**  
Planilha de referência com as tarifas base de frete, atualizada mensalmente pela área Comercial. Localização: `\\novatech-fs\comercial\tabelas\frete-base-AAAAMM.xlsx`. Constitui o "Valor base" na fórmula do frete especial.  
*Fonte: PROC-042-v2 Seção 2*

**tier** *(substantivo comum, valores sempre capitalizados)*  
Classificação do cliente na NovaTech com base em volume e valor contratual, determinando os SLAs, benefícios e obrigações contratuais aplicáveis. Existem exatamente três tiers: **tier Gold**, **tier Silver** e **tier Standard**. Não existe tier Platinum nem qualquer outra denominação. *Uso canônico: "tier" minúsculo + valor capitalizado: "tier Gold", "tier Silver", "tier Standard".*  
*Fonte: SLA-2024 Seção 1*

**tier Gold**  
Tier de maior nível. Critério: contrato anual acima de R$ 500.000 OU mais de 200 operações/mês. Revisão semestral.  
*Fonte: SLA-2024 Seção 1*

**tier Silver**  
Tier intermediário. Critério: contrato anual entre R$ 100.000 e R$ 500.000 OU entre 50 e 200 operações/mês. Revisão semestral.  
*Fonte: SLA-2024 Seção 1*

**tier Standard**  
Tier base. Critério: todos os demais clientes. Revisão anual.  
*Fonte: SLA-2024 Seção 1*

**tier Platinum**  
Denominação inexistente na NovaTech. Pode gerar confusão com programas de outras transportadoras ou com o programa de fidelidade NovaTech descontinuado em 2022.  
*Fonte: SLA-2024 Seção 1; FAQ item 15*

**Timestamp de abertura do chamado**  
Marco de início da contagem de SLA, registrado automaticamente pelo sistema de chamados (Azure DevOps) no momento em que o chamado é criado.  
*Fonte: SLA-2024 Seção 5*

**Triagem**  
Etapa inicial do processo de devolução realizada pelo time de atendimento após abertura do chamado. Verifica elegibilidade, prazo e documentação. Prazo: 4 horas úteis a partir da abertura do chamado.  
*Fonte: POL-001 Seção 3.3*

---

### V

**Valor base**  
Tarifa de frete publicada mensalmente na tabela de fretes da NovaTech. Primeiro fator da fórmula do frete especial: `Valor base × Multiplicador regional × Fator de peso`.  
*Fonte: PROC-042-v2 Seção 2*

**Violação de SLA**  
Ocorrência de atendimento em que os prazos de resposta ou resolução definidos contratualmente para o tier do cliente foram descumpridos. Contada mensalmente por cliente para fins de aplicação de penalidades.  
*Fonte: SLA-2024 Seção 4*

**Volume**  
Unidade individual de uma entrega quando a carga é composta por múltiplos itens ou embalagens. Em devoluções parciais, o cliente pode devolver volumes individuais, e o reembolso é proporcional ao peso ou valor do volume conforme o CT-e.  
*Fonte: POL-001 Seção 3.4*

---

## SEÇÃO II — Termos do Sistema

*Vocabulário interno do assistente de IA NovaTech. Não derivado dos documentos da NovaTech — definido pelo time de produto e engenharia. Todos os artefatos técnicos (bounded contexts, requirements, ADRs) devem usar estes termos com estas definições exatas.*

---

**Ambiguidade semântica**  
Tipo de ambiguidade em que a pergunta usa termos vagos ou polissêmicos que remetem a múltiplos domínios documentais, independentemente dos scores de similaridade retornados pelo retrieval. Exemplo: "Qual o prazo?" pode remeter a devolução, frete especial ou SLA. Detectada em BC-03 pela análise dos temas dos chunks recuperados. Resulta em resposta de esclarecimento (lista de opções ao atendente). *Ver também: Ambiguidade de score.*  
*(Resolve AMB-02)*

**Ambiguidade de score**  
Tipo de ambiguidade em que dois ou mais chunks de temas distintos retornam scores de similaridade semântica semelhantes (diferença < 0,05 em cosine similarity) para a mesma query, tornando indeterminado qual tema o atendente está consultando. Detectada em BC-02 durante o retrieval. Sinalizada via flag `is_ambiguous: true` no pacote de contexto. *Ver também: Ambiguidade semântica.*  
*(Resolve AMB-02)*

**Chunk**  
Fragmento de texto extraído de um documento durante a ingestão, com tamanho alvo de ~1.500 tokens, mantendo coerência semântica (não corta no meio de tabelas ou seções). Unidade básica de recuperação no Azure AI Search.

**Chunk vencedor**  
Chunk selecionado em caso de conflito de retrieval: aquele com menor número de nível hierárquico; em caso de empate de nível, o com data de atualização mais recente. Definido em BC-02 antes de montar o pacote de contexto.

**Confirmação de vigência**  
Ato formal do responsável de base de uma área confirmando que um documento continua válido, mesmo sem alteração de conteúdo. Gera atualização do metadado `last_confirmed_at` em BC-04. **Distinto da data de atualização do documento.** O aviso de desatualização (> 6 meses) é baseado em `last_confirmed_at`, não na data do documento.  
*(Resolve AMB-13)*

**Conflito de retrieval**  
Situação em que dois ou mais chunks recuperados para a mesma query cobrem o mesmo tema com valores ou regras divergentes. Detectado em BC-02 pela comparação de metadados de documento e seção. Resulta em flag `has_conflict: true` no pacote de contexto e alerta obrigatório na resposta.

**Context budget**  
Orçamento de tokens do prompt enviado ao LLM por requisição, conforme ADR-0002: ~4K tokens para system prompt + ~8K tokens para chunks (top-5 de ~1.500 tokens) + pergunta atual + histórico de 3 turnos. Total não deve exceder o orçamento fixo sem nova ADR.

**Declaração de ausência**  
Resposta estruturada gerada por BC-03 quando o sistema não tem informação confiável disponível. Deve sempre incluir: (a) afirmação explícita de que o tema não está coberto, (b) motivo identificável, (c) próximo passo sugerido. **Nunca contém valores estimados ou inferidos.**

**Embedding**  
Representação vetorial de um chunk, gerada via Azure OpenAI durante a ingestão (BC-01). Usada pelo Azure AI Search para cálculo de similaridade semântica durante o retrieval (BC-02).

**Nível** *(hierarquia de fontes)*  
Classificação de confiabilidade de um documento indexado, de 1 a 4: Nível 1 = normativo formal (POL, PROC, SLA com responsável e versão); Nível 2 = planilhas de referência operacional; Nível 3 = páginas do Confluence validadas; Nível 4 = FAQ-Atendimento (informal). Em conflito, nível menor prevalece. Em empate de nível, data mais recente prevalece.  
*(Resolve AMB-10)*

**Nível de confiança**  
Indicador qualitativo da confiabilidade de uma resposta, calculado em BC-03 com base nos critérios abaixo. Exibido obrigatoriamente ao atendente em toda resposta com conteúdo factual.

| Nível | Critérios |
|---|---|
| **Alta** | Resposta baseada exclusivamente em documento(s) de Nível 1; sem conflito detectado; `last_confirmed_at` ≤ 6 meses |
| **Moderada** | Nível 1 com `last_confirmed_at` > 6 meses; OU Nível 2 ou 3 validado; OU Nível 1 com conflito resolvido pelo chunk vencedor |
| **Baixa** | Nível 4 (FAQ); OU documento com status `em-revisao`; OU `last_confirmed_at` desconhecida |

*(Resolve AMB-04)*

**Pacote de contexto**  
Estrutura de dados montada por BC-02 e entregue a BC-03, contendo: lista de chunks com metadados, flag `has_conflict` (booleano), flag `is_ambiguous` (booleano), lista `ambiguous_topics` (temas candidatos quando `is_ambiguous: true`), flag `no_result` (booleano quando nenhum chunk supera o threshold).  
*(Resolve AMB-01)*

**Pergunta ambígua**  
Pergunta do atendente que admite mais de uma interpretação com respostas factuais distintas. Pode ser do tipo **ambiguidade de score** (detectada em BC-02) ou **ambiguidade semântica** (detectada em BC-03). Em ambos os casos, o sistema solicita esclarecimento ao atendente antes de fornecer conteúdo factual.  
*(Resolve AMB-02)*

**Pergunta multi-partes**  
Pergunta do atendente que contém dois ou mais subtemas independentes, cada um exigindo retrieval e resposta separados. Detectada em BC-02 no pré-processamento por heurística de conectores ("e", "+", vírgula entre entidades de domínios distintos). Limite: até 4 partes por pergunta. Se o sistema não detectar a multi-parte, responde pelo chunk de maior score e não gera blocos separados — comportamento registrado como fallback no log.  
*(Resolve AMB-06)*

**Push de reindexação**  
Evento disparado pelo responsável de base de uma área notificando o pipeline (BC-01) que um documento foi publicado ou atualizado e deve ser reprocessado. SLA: 4 horas para documentos de Nível 1.

**Score de relevância**  
Métrica de similaridade semântica (cosine similarity, escala 0–1) entre a query e cada chunk candidato, calculada pelo Azure AI Search durante o retrieval.

**Streaming**  
Modo de entrega de resposta em que os tokens chegam progressivamente ao cliente à medida que são gerados pelo LLM. O assistente usa streaming. Dois SLAs se aplicam: **TTFT** (Time to First Token, meta < 3s) e **tempo de resposta completa** (meta < 30s no percentil 95). O timer exibido ao atendente durante o carregamento mede o TTFT.  
*(Resolve AMB-09)*

**Threshold de confiança**  
Score mínimo de cosine similarity para que um chunk seja incluído no pacote de contexto. **Valor inicial:** 0,72 (derivado do protótipo com ChromaDB). Comportamento de borda: retorna todos os chunks que superam o threshold, mesmo que menos de 5; se nenhum supera, sinaliza `no_result: true`. O valor pode ser ajustado via configuração sem nova ADR, mas qualquer mudança deve ser registrada no changelog de configuração.  
*(Resolve AMB-03)*

**Turno**  
Par pergunta-resposta dentro de uma sessão de conversa. Um turno = 1 mensagem do atendente + 1 resposta do assistente (incluindo respostas de esclarecimento). Interações de clarificação (S5: opções de esclarecimento + escolha do atendente) contam como 1 turno do ponto de vista do contexto — a pergunta original e a escolha do atendente são tratadas como uma única query. O histórico enviado ao LLM inclui os últimos 3 turnos completos (6 mensagens).  
*(Resolve AMB-07)*

**Versão do assistente**  
Identificador semântico da versão do sistema que gerou uma resposta (ex: `v1.2.0`). Registrado obrigatoriamente em todos os logs de auditoria (BC-06). **Não exibido ao atendente** na interface — disponível apenas no painel administrativo e nos logs para fins de rastreabilidade e comparação de qualidade entre versões.  
*(Resolve AMB-14)*

---

*Glossário v2.0. Seção I: 35 termos de negócio. Seção II: 17 termos do sistema. Resolve AMB-02, AMB-03, AMB-04, AMB-06, AMB-07, AMB-09, AMB-10, AMB-13, AMB-14, AMB-15 da revisão técnica.*
