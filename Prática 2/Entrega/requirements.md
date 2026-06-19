# requirements.md — Query Endpoint
## NovaTech Assistant

**Módulo:** Query Endpoint  
**Versão:** 1.0  
**Data:** 05/06/2026  
**Contextos cobertos:** Atendimento ao Cliente · Logística de Frete · Políticas de Devolução · SLAs e Contratos · Gestão Documental e Governança do Conhecimento

---

## Outcomes

### Para o atendente

- Obter uma resposta fundamentada em documentação oficial em menos de 30 segundos, sem precisar abrir nenhuma fonte externa manualmente.
- Saber exatamente de onde veio cada informação recebida — documento, versão e seção — para poder confiar na resposta e usá-la diretamente no atendimento ao cliente.
- Receber orientação clara e imediata quando o assistente não tem informação confiável disponível, em vez de receber silêncio ou uma resposta inventada.
- Identificar sem ambiguidade quando duas fontes divergem sobre o mesmo tema, de forma a escalar para o supervisor com contexto suficiente.
- Reduzir o tempo de busca por informação de aproximadamente 12 minutos para menos de 2 minutos por chamado.

### Para o negócio

- Aumentar a consistência das respostas fornecidas ao cliente final ao eliminar a variação causada por atendentes consultando fontes diferentes ou versões distintas de um mesmo documento.
- Reduzir o volume de escaladas desnecessárias ao supervisor originadas por incapacidade de encontrar a resposta — meta: de 15% para abaixo de 8% dos chamados que consultam o assistente.
- Criar rastreabilidade auditável de quais informações foram fornecidas a quais atendentes em quais momentos, suportando eventuais contestações contratuais.
- Tornar visível o gap documental da empresa ao registrar sistematicamente as perguntas que o assistente não consegue responder, viabilizando priorização de cobertura documental.

---

## Scope Boundaries

### In Scope

O query endpoint cobre as seguintes responsabilidades, derivadas dos bounded contexts definidos:

**Atendimento ao Cliente (BC-05 / BC-03)**
- Receber uma pergunta em linguagem natural de um atendente autenticado via Microsoft Teams ou painel web interno.
- Processar perguntas nas categorias mais frequentes de consulta: prazos de entrega, regras de frete especial, política de devolução de mercadorias e SLAs por tier de cliente.
- Processar perguntas que cruzam mais de uma categoria simultaneamente (estimado em ~15% dos casos).
- Retornar uma resposta estruturada com: conteúdo da resposta, bloco de rastreabilidade (fonte, versão, seção, data, nível de confiança) e alertas contextuais quando aplicáveis.
- Declarar ausência de resposta de forma estruturada e com motivo identificável quando o tema não tiver cobertura confiável na base.
- Solicitar esclarecimento ao atendente quando a pergunta for ambígua e admitir mais de uma interpretação com respostas distintas.
- Processar perguntas com múltiplas partes respondendo cada parte individualmente e declarando ausência nas partes sem cobertura.
- Manter histórico de até 3 turnos anteriores da conversa para contextualizar a resposta da pergunta atual.

**Logística de Frete (BC-02 / BC-03)**
- Recuperar e apresentar regras de cálculo de frete especial (acima de 500 kg), incluindo multiplicadores regionais, fatores de peso e condições de desconto por volume, conforme a versão vigente da PROC-042.
- Alertar o atendente quando multiplicadores ou condições forem divergentes entre versões de documentos recuperados, indicando qual versão prevalece e por quê.

**Políticas de Devolução (BC-02 / BC-03)**
- Recuperar e apresentar regras da POL-001: prazo de 7 dias úteis, categorias inelegíveis (cargas perigosas classes 1–6, cargas refrigeradas com ruptura de cadeia de frio, cargas com lacre violado), procedimento de abertura de chamado, prazos de triagem, coleta reversa e reembolso.
- Distinguir corretamente a devolução padrão da carga danificada em trânsito, orientando para o canal correto em cada caso.

**SLAs e Contratos (BC-02 / BC-03)**
- Recuperar e apresentar SLAs por tier (Gold, Silver, Standard): tempos de resposta e resolução para chamados gerais e incidentes críticos, critérios de incidente crítico, penalidades por descumprimento e regras de medição de SLA.
- Confirmar que não existe tier Platinum ou qualquer outro tier além dos três oficiais.

**Gestão Documental e Governança do Conhecimento (BC-01 / BC-02 / BC-04)**
- Aplicar a hierarquia de fontes ao compor a resposta: Nível 1 (normativo formal) prevalece sobre Nível 4 (FAQ informal); desempate por data quando documentos do mesmo nível divergem.
- Sinalizar ao atendente quando a resposta é baseada exclusivamente em documento informal (FAQ), indicando que não há respaldo normativo formal.
- Sinalizar quando o documento fonte tem mais de 6 meses sem atualização confirmada, recomendando verificação com o supervisor.

---

### Out of Scope

O query endpoint explicitamente **não faz** o seguinte:

**Contextos externos ao sistema**
- Calcular fretes de forma autônoma — o endpoint recupera e apresenta os parâmetros documentados; não executa a fórmula com valores reais de chamados.
- Consultar ou apresentar dados do contrato individual de clientes — contratos não estão indexados na base.
- Criar, editar, publicar ou arquivar documentos nas fontes externas (SharePoint, Confluence).
- Autenticar o atendente — delegado ao Azure AD / Microsoft 365.
- Acessar o Portal do Cliente (portal.novatech.com.br) ou qualquer sistema externo à NovaTech.

**Contextos externos ao escopo do produto**
- Responder perguntas sobre temas tratados pela Gestão de Riscos (ex: tratamento de exceções de devolução de carga perigosa) — o endpoint orienta o atendente a contatar o ramal 4500, conforme POL-001, mas não representa o processo desse setor.
- Responder perguntas sobre sinistros em trânsito conduzidos pelo Jurídico — o endpoint orienta para o canal de sinistros sem reproduzir o processo jurídico interno.
- Responder perguntas sobre negociações comerciais individuais (descontos fora das faixas automáticas, contratos especiais) — o endpoint orienta escalada para o Comercial.
- Fornecer qualquer informação sobre clientes em linguagem de decisão ("o cliente X tem direito a...") — o endpoint fornece as regras gerais; a aplicação ao caso específico é do atendente.

**Funcionalidades de outros módulos do sistema**
- Indexar, chunkizar ou reindexar documentos — responsabilidade do pipeline de ingestão (BC-01).
- Gerenciar o ciclo de vida ou status dos documentos — responsabilidade da Governança Documental (BC-04).
- Registrar feedback e auditar respostas — responsabilidade do módulo de Rastreabilidade e Feedback (BC-06).
- Exibir o painel de métricas ou o histórico administrativo — fora do escopo do endpoint de consulta.
- Enviar notificações proativas ao atendente sobre resolução de feedbacks.

---

## Constraints

### Fundamentação em evidências

Toda resposta deve ser exclusivamente derivada dos chunks recuperados da base documental indexada. O endpoint não pode acrescentar informação que não esteja presente nos chunks recebidos no pacote de contexto da consulta corrente.

### Exibição obrigatória de fontes

Cada resposta com conteúdo factual deve incluir, sem exceção: nome do documento, número de versão, seção de referência, data da última atualização do documento e nível de confiança (Alta / Moderada / Baixa). Esses campos não são opcionais nem podem ser ocultados pelo atendente.

### Ausência de alucinação

O endpoint não deve usar termos de estimativa ou inferência — "provavelmente", "normalmente", "costuma ser", "deve ser", "em geral", "aproximadamente" — em afirmações sobre prazos, valores numéricos, condições contratuais ou procedimentos operacionais. Quando a informação não estiver disponível na base, a resposta deve ser uma declaração de ausência estruturada, não uma estimativa.

### Tratamento explícito de contradições

Quando dois ou mais chunks recuperados cobrirem o mesmo tema com valores ou regras divergentes, o endpoint deve: (a) usar o chunk de maior hierarquia de nível como base da resposta, (b) incluir alerta explícito identificando o documento divergente pelo nome e versão, e (c) nunca combinar valores de fontes contraditórias em uma única afirmação.

### Atualização documental

O endpoint deve operar com a base documental atualizada em até 24 horas após a publicação oficial de um documento novo ou revisado nas fontes externas. Enquanto um documento está em processo de reindexação, o endpoint deve exibir banner informativo ao atendente indicando que a base sobre aquele tema pode não refletir as mudanças mais recentes.

### Tempo de resposta

O endpoint deve retornar a resposta completa ao atendente em menos de 30 segundos a partir do recebimento da pergunta, medido do lado do cliente (Teams ou painel web), para o percentil 95 das requisições em condições normais de operação.

### Idioma

O endpoint recebe perguntas e retorna respostas exclusivamente em português brasileiro. Perguntas em outros idiomas devem receber resposta informando a limitação de idioma e solicitando reformulação — sem tentativa de resposta no idioma da pergunta.

### Escopo de perguntas

O endpoint responde exclusivamente com base na documentação interna da NovaTech. Perguntas sobre legislação externa, dados de terceiros, concorrentes ou qualquer tema fora do escopo da base documental devem receber declaração de fora de escopo com indicação de próximo passo.

### Dados pessoais

O endpoint não persiste dados pessoais de clientes (nome, CNPJ, número de contrato, endereço) mencionados nas perguntas dos atendentes. Qualquer dado de cliente presente na pergunta é descartado após o processamento da consulta corrente.

### Contexto de conversa

O histórico de conversa incluído no contexto do LLM é limitado a 3 turnos anteriores, conforme ADR-0002. Turnos mais antigos não influenciam a resposta atual.

---

## Prior Decisions

### ADR-0001 — Azure OpenAI (GPT-4o) como modelo principal

O modelo de linguagem utilizado para geração de respostas é o GPT-4o via Azure OpenAI. Essa decisão foi tomada pela integração nativa com o ecossistema Microsoft da NovaTech (Teams, SharePoint, Azure AD) e pela janela de contexto de 128K tokens, que suporta o orçamento de contexto definido. A escolha do modelo não deve ser rediscutida neste módulo; eventuais revisões de modelo seguem o processo de ADR.

### ADR-0002 — Orçamento de contexto limitado e histórico restrito

O prompt enviado ao LLM em cada consulta é composto por: ~4K tokens para o system prompt (guardrails e instruções de formato), ~8K tokens para os chunks recuperados (5 chunks de ~1.500 tokens cada), histórico limitado a 3 turnos anteriores da conversa, e a pergunta atual. O total disponível na janela de 128K tokens do GPT-4o é respeitado com folga, mas o orçamento dos componentes acima é fixo e não deve ser expandido sem nova ADR. O limite de histórico de 3 turnos é intencional para controlar custo e latência.

### ADR-0003 — Preservação de documentos contraditórios com priorização por metadado de vigência

Documentos contraditórios não são excluídos da base; são preservados com status diferenciado (`vigente`, `arquivado`, `em-revisao`). O endpoint prioriza o documento de maior hierarquia de nível e, dentro do mesmo nível, o de data de atualização mais recente. A lógica de resolução de conflito é executada na camada de recuperação (BC-02) antes de montar o pacote de contexto para o LLM. O LLM não decide qual versão prevalece — recebe apenas o chunk vencedor e a flag de conflito para incluir o alerta na resposta.

### ADR-0004 — Arquitetura RAG com Azure AI Search + Azure OpenAI

A recuperação de chunks é feita por busca vetorial por similaridade semântica no Azure AI Search. Os embeddings dos chunks são gerados via Azure OpenAI no momento da ingestão. A busca retorna os top-5 chunks mais relevantes acima do threshold de confiança configurado. Essa arquitetura foi validada no protótipo open-source (ChromaDB + sentence-transformers) durante o discovery, que também identificou o problema de chunking em tabelas — resolvido no pipeline de ingestão com regra de não-corte de seções tabulares.

---

## Verification Criteria

### VC-01 — Resposta com fonte obrigatória

**Dado** que existe na base um documento vigente cobrindo o tema da pergunta com confiança alta,  
**quando** o atendente submete uma pergunta sobre esse tema,  
**então** a resposta deve conter: (a) o conteúdo factual correto conforme o documento, (b) o nome completo do documento, (c) o número de versão, (d) a seção de referência, (e) a data da última atualização do documento, e (f) o nível de confiança "Alta" — todos visíveis na resposta sem necessidade de ação adicional do atendente.

---

### VC-02 — Resposta baseada em documento informal sinalizada

**Dado** que o único documento disponível sobre o tema é o FAQ-Atendimento (Nível 4 — informal),  
**quando** o atendente submete uma pergunta sobre esse tema,  
**então** a resposta deve: (a) apresentar o conteúdo do FAQ, (b) incluir aviso explícito de que a fonte é um documento informal não validado por Compliance, (c) exibir nível de confiança "Baixa", e (d) recomendar confirmação com supervisor para casos de impacto financeiro ou contratual.

---

### VC-03 — Tratamento de documentos contraditórios

**Dado** que dois documentos indexados cobrem o mesmo tema com valores divergentes (ex: multiplicadores regionais da PROC-042 v1 e PROC-042-v2),  
**quando** o atendente faz uma pergunta sobre esse tema,  
**então** a resposta deve: (a) apresentar apenas o valor do documento de maior hierarquia ou mais recente, (b) incluir alerta explícito identificando o documento divergente pelo nome e versão, (c) não apresentar os dois valores como igualmente válidos, e (d) não combinar valores dos dois documentos em uma única afirmação.

---

### VC-04 — Ausência de resposta por tema não coberto

**Dado** que nenhum documento indexado cobre o tema da pergunta (ex: seguro de carga, frete padrão abaixo de 500 kg),  
**quando** o atendente submete uma pergunta sobre esse tema,  
**então** a resposta deve: (a) declarar explicitamente que o tema não está coberto na base documental disponível, (b) identificar o motivo (tema não indexado / documento não disponível / documento em revisão), (c) indicar a área responsável para consulta quando identificável a partir de documentos formais, e (d) oferecer opção de registrar a pergunta como gap — sem fornecer nenhum conteúdo factual sobre o tema.

---

### VC-05 — Ausência de alucinação em informações numéricas

**Dado** que a pergunta solicita um valor numérico (prazo, multiplicador, percentual, limite financeiro) não presente em nenhum chunk recuperado,  
**quando** o endpoint processa a pergunta,  
**então** a resposta não deve conter nenhum valor numérico sobre o tema consultado — apenas a declaração de ausência estruturada. Nenhuma frase de estimativa ("provavelmente", "normalmente", "deve ser") deve aparecer na resposta.

---

### VC-06 — Pergunta que cruza múltiplos domínios

**Dado** que a pergunta envolve simultaneamente mais de uma categoria (ex: prazo de devolução + regra de frete especial + SLA para cliente Gold),  
**quando** o atendente submete essa pergunta,  
**então** a resposta deve: (a) estruturar a resposta em blocos separados por parte da pergunta, (b) responder cada parte com a fonte correspondente de forma independente, (c) declarar ausência explicitamente nas partes sem cobertura documental, e (d) não misturar fontes de domínios distintos em uma única afirmação factual.

---

### VC-07 — Pergunta ambígua

**Dado** que a pergunta pode ser interpretada de formas distintas com respostas diferentes (ex: "Qual o prazo?" sem especificação do tipo),  
**quando** o atendente submete essa pergunta,  
**então** o endpoint não deve escolher uma interpretação arbitrária. A resposta deve apresentar as interpretações possíveis identificadas com base nos temas disponíveis na base e solicitar esclarecimento ao atendente antes de fornecer conteúdo factual.

---

### VC-08 — Tempo máximo de resposta

**Dado** que o assistente está em operação em condições normais (sem degradação de infraestrutura),  
**quando** o atendente submete qualquer pergunta,  
**então** a resposta completa deve ser entregue ao atendente em menos de 30 segundos, medidos do lado do cliente (Teams ou painel web), para no mínimo 95% das requisições em um período de observação de 24 horas.

---

### VC-09 — Documento com mais de 6 meses sem atualização

**Dado** que o documento fonte da resposta tem data de última atualização superior a 6 meses sem confirmação de vigência pelo responsável de base,  
**quando** o atendente recebe a resposta,  
**então** a resposta deve incluir aviso explícito indicando que o documento não é atualizado há mais de 6 meses e recomendando confirmação com a área responsável antes de usar a informação em contexto contratual.

---

### VC-10 — Pergunta fora do escopo

**Dado** que a pergunta envolve tema explicitamente fora do escopo do endpoint (ex: negociação comercial individual, processo jurídico de sinistro, dados de contrato de cliente específico),  
**quando** o atendente submete essa pergunta,  
**então** a resposta deve: (a) reconhecer o tema da pergunta, (b) informar que o assistente responde exclusivamente com base na documentação interna da NovaTech e que aquele tema está fora desse escopo, (c) indicar a área ou canal correto para a questão, e (d) não fornecer nenhum conteúdo factual sobre o tema.

---

### VC-11 — Integridade do bloco de rastreabilidade

**Dado** que o atendente recebe qualquer resposta com conteúdo factual,  
**quando** o atendente visualiza o bloco de rastreabilidade,  
**então** os alertas de conflito, de fonte informal e de desatualização devem estar sempre visíveis — não podem ser ocultados como parte do bloco colapsável. O conteúdo do bloco de rastreabilidade deve corresponder exatamente aos metadados do chunk utilizado para gerar aquela afirmação específica.

---

### VC-12 — Histórico de conversa limitado

**Dado** que o atendente está em uma conversa com mais de 3 turnos anteriores,  
**quando** o atendente submete uma nova pergunta,  
**então** o contexto enviado ao LLM deve conter no máximo os 3 turnos mais recentes. Turnos mais antigos não devem influenciar a resposta, e o atendente não deve receber respostas que referenciem informações de turnos anteriores ao limite de 3.
