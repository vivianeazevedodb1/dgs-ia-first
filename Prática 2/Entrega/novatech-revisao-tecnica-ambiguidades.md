# Revisão Técnica — Ambiguidades e Inconsistências na Documentação
## NovaTech Assistant · Query Endpoint

**Elaborado por:** Tech Lead  
**Data:** 05/06/2026  
**Versão:** 1.0  
**Documentos revisados:**
- `novatech-bounded-contexts.md` (BC-01 a BC-06)
- `novatech-glossario-linguagem-ubiqua.md`
- `requirements.md` (v1.0 — Query Endpoint)
- `novatech-mockups.html` (S1 a S7)

> Este documento lista ambiguidades, contradições internas e lacunas de especificação encontradas na revisão cruzada dos artefatos acima. Cada item tem identificador único, localização precisa nos documentos, o problema diagnosticado e uma proposta de resolução. O objetivo é que o time resolva esses pontos antes de iniciar implementação — não durante.

---

## Sumário por severidade

| Severidade | Quantidade | Impacto se não resolvido |
|---|---|---|
| 🔴 Bloqueante | 5 | Gera comportamento indefinido em runtime ou testes impossíveis de escrever |
| 🟡 Importante | 8 | Gera implementações divergentes entre devs ou falsos positivos no QA |
| 🟢 Menor | 4 | Inconsistência terminológica ou lacuna de detalhe que não bloqueia, mas polui |

---

## 🔴 Ambiguidades Bloqueantes

---

### AMB-01 — Quem detecta "pergunta ambígua": BC-02 ou BC-03?

**Severidade:** 🔴 Bloqueante  
**Localização:** `bounded-contexts.md` BC-02 (dentro) vs BC-03 (dentro) vs `requirements.md` In Scope  

**Problema:**  
BC-02 inclui em seu escopo: *"Sinalizar pergunta ambígua (múltiplos temas com scores semelhantes) para que BC-03 solicite esclarecimento."*  
BC-03 inclui em seu escopo: *"Tratar perguntas ambíguas: apresentar as interpretações possíveis com base nos temas identificados pelos chunks recuperados e solicitar esclarecimento."*  

As duas descrições são diferentes:
- BC-02 detecta a ambiguidade por score semelhante de múltiplos temas.
- BC-03 "trata" — mas também "identifica os temas" com base nos chunks.

Não está claro se BC-03 recebe uma flag binária de ambiguidade de BC-02 (e BC-02 já sabe quais temas conflitam) ou se BC-03 recebe os chunks normalmente e detecta a ambiguidade por conta própria ao compor o prompt. As duas versões implicam contrato de interface completamente diferente entre os dois BCs.

`requirements.md` In Scope diz apenas *"Solicitar esclarecimento ao atendente quando a pergunta for ambígua"* sem atribuir responsabilidade de detecção.

**Proposta de resolução:**  
Definir explicitamente: BC-02 detecta ambiguidade (tem o score), seta flag `is_ambiguous: true` no pacote de contexto, e inclui a lista de temas candidatos identificados. BC-03 recebe essa flag e formata a resposta de esclarecimento — não detecta novamente. Atualizar os dois BCs e o contrato do pacote de contexto.

---

### AMB-02 — Definição de "pergunta ambígua" difere entre BC-02 e o mockup S5

**Severidade:** 🔴 Bloqueante  
**Localização:** `bounded-contexts.md` BC-02 glossário vs `novatech-mockups.html` S5  

**Problema:**  
BC-02 define pergunta ambígua como: *"Query que pode ser respondida por chunks de temas distintos com scores similares."*  
O mockup S5 mostra a pergunta *"Qual o prazo?"* gerando 4 opções de esclarecimento — mas essa pergunta não é ambígua por score similar de chunks. É ambígua semânticamente (a palavra "prazo" remete a múltiplos contextos). Uma busca vetorial de "qual o prazo" provavelmente retornaria chunks de POL-001 com score mais alto do que os demais, sem empate de scores.

Há dois tipos de ambiguidade que a documentação trata como um só:
1. **Ambiguidade de score:** múltiplos chunks de temas distintos com similaridade semelhante (detectável em BC-02).
2. **Ambiguidade semântica:** pergunta vaga que pode remeter a múltiplos contextos, mas o score pode não empatar (detectável apenas em BC-03 ao analisar o conteúdo).

Se BC-02 só detecta o tipo 1, o cenário do mockup S5 nunca seria gerado pelo sistema real — o assistente responderia com base no chunk de maior score (POL-001) sem perguntar nada.

**Proposta de resolução:**  
Distinguir os dois tipos no glossário e nos BCs. Definir qual o mecanismo de detecção de cada tipo. Alinhar o mockup S5 com o tipo que o sistema realmente implementará — ou documentar os dois tipos com comportamentos separados.

---

### AMB-03 — "Threshold de confiança" é definido mas nunca especificado

**Severidade:** 🔴 Bloqueante  
**Localização:** `bounded-contexts.md` BC-02 glossário e escopo; `requirements.md` ADR-0004  

**Problema:**  
BC-02 define threshold de confiança como *"Score mínimo de similaridade abaixo do qual o chunk não é incluído no contexto (valor a calibrar em testes)"* e descreve o comportamento: *"Sinalizar pergunta sem resultado suficiente (nenhum chunk acima do threshold de confiança)."*

ADR-0004 diz: *"A busca retorna os top-5 chunks mais relevantes acima do threshold de confiança configurado."*

Nenhum artefato define:
- Qual é o valor do threshold (mesmo que inicial/provisório).
- Se é um valor absoluto (ex: cosine similarity > 0.75) ou relativo (ex: top-1 score < X%).
- Se o threshold se aplica individualmente a cada chunk ou ao conjunto.
- Quem é responsável por definir e ajustar o threshold (BC-02? BC-04? time de ML?).
- O que acontece se apenas 3 dos 5 chunks estão acima do threshold — retorna 3 ou retorna 0?

Isso torna VC-04 (ausência de resposta) não testável: o QA não sabe em que condição o sistema deve declarar ausência.

**Proposta de resolução:**  
Definir um valor inicial documentado (ex: cosine similarity ≥ 0.72, baseado no protótipo com ChromaDB). Especificar o comportamento de borda (retorna os chunks acima do threshold, mesmo que menos de 5). Registrar como ADR se a escolha do valor for considerada decisão arquitetural.

---

### AMB-04 — "Nível de confiança" tem dois critérios em conflito

**Severidade:** 🔴 Bloqueante  
**Localização:** `requirements.md` Constraints "Exibição obrigatória de fontes" vs BC-03 glossário vs `novatech-especificacao-requisitos-rag-v2.md` Seção 6.2  

**Problema:**  
Há três definições de nível de confiança nos artefatos, com critérios divergentes:

**Definição A** — `requirements.md` Constraints:  
*"nível de confiança (Alta / Moderada / Baixa)"* — sem critérios.

**Definição B** — `bounded-contexts.md` BC-03 glossário:  
*"alta = Nível 1 sem conflito e < 6 meses; baixa = FAQ ou documento em revisão"* — Moderada não é definida.

**Definição C** — `especificacao-requisitos-rag-v2.md` Seção 6.2:  
*"Alta: Nível 1, sem conflito, < 6 meses. Moderada: Nível 1 com > 6 meses OU Nível 2 ou 3 validado OU Nível 1 com conflito resolvido (versão mais recente prevalece). Baixa: Nível 4 (FAQ) OU documento em revisão OU data desconhecida."*

O mockup S3 exibe "Confiança moderada" para resposta com conflito — consistente com Definição C, mas BC-03 não define Moderada explicitamente.

O `requirements.md` é o documento de referência do módulo, mas não contém os critérios objetivos — apenas cita o conceito. Um dev implementando BC-03 a partir do `requirements.md` não tem como calcular o nível de confiança corretamente.

**Proposta de resolução:**  
Incorporar a Definição C completa ao `requirements.md` em uma tabela dentro de Constraints. Remover as definições parciais de BC-03 e referenciar o `requirements.md` como fonte de verdade.

---

### AMB-05 — Comportamento de rollback não está no escopo do query endpoint, mas é disparado por ele

**Severidade:** 🔴 Bloqueante  
**Localização:** `bounded-contexts.md` BC-01 (rollback) vs BC-06 vs `requirements.md` Out of Scope  

**Problema:**  
`requirements.md` Out of Scope declara explicitamente: *"Registrar feedback e auditar respostas — responsabilidade do módulo de Rastreabilidade e Feedback (BC-06)."*

`bounded-contexts.md` BC-01 descreve: *"Rollback: capacidade de reverter a indexação de um documento para a versão anterior em caso de problema identificado via feedback (BC-06)."*

BC-06 descreve: *"Feedbacks críticos resolvidos que implicam reindexação disparam ação em BC-01."*

O fluxo completo é: atendente clica ❌ no feedback (BC-05, parte do query endpoint) → BC-06 registra → BC-06 aciona BC-01 (rollback). O botão ❌ que inicia esse fluxo está no mockup S2 e faz parte da interface do query endpoint.

O problema: o `requirements.md` exclui o registro de feedback do escopo, mas o mockup inclui os botões de feedback em todas as telas de resposta. Se o QA vai testar o query endpoint, ele vai testar os botões de feedback — mas os critérios de verificação (VC-01 a VC-12) não cobrem o comportamento do feedback.

A fronteira entre o que pertence ao query endpoint e o que pertence ao módulo de feedback está indefinida para a camada de interface.

**Proposta de resolução:**  
Definir explicitamente que o query endpoint é responsável por *coletar* o feedback (exibir os botões e enviar o evento) mas *não por processar nem rotear*. Adicionar um VC para esse comportamento de coleta: "Dado que o atendente clica ❌, quando a ação é processada, então o evento de feedback é emitido com os campos X, Y, Z e confirmação visual é exibida ao atendente."

---

## 🟡 Ambiguidades Importantes

---

### AMB-06 — "Pergunta multi-partes" não tem critério de detecção

**Severidade:** 🟡 Importante  
**Localização:** `requirements.md` In Scope; `bounded-contexts.md` BC-03; mockup S6  

**Problema:**  
`requirements.md` define que o endpoint deve *"processar perguntas com múltiplas partes respondendo cada parte individualmente"*. BC-03 inclui o mesmo comportamento. O mockup S6 ilustra a resposta estruturada por blocos.

Nenhum artefato define:
- Como o sistema detecta que uma pergunta tem múltiplas partes (pontuação? conjunções? número de entidades distintas?).
- Se a detecção é feita em BC-02 (antes do retrieval) ou em BC-03 (ao compor o prompt).
- O que acontece se o sistema não detecta a multi-parte — responde apenas a primeira parte? A mais relevante por score?
- Existe um limite de partes por pergunta?

Sem isso, VC-06 (pergunta multi-domínio) não tem critério de aceite para a etapa de detecção — só para o formato da resposta.

**Proposta de resolução:**  
Definir o mecanismo de detecção (sugestão: heurística no pré-processamento de BC-02, baseada em conectores "e", "+", vírgulas com múltiplos substantivos-domínio). Documentar o comportamento de fallback se a detecção falhar. Adicionar esse critério ao VC-06.

---

### AMB-07 — Histórico de conversa: "turno" não está definido na linguagem ubíqua

**Severidade:** 🟡 Importante  
**Localização:** `requirements.md` ADR-0002 e Constraints; `bounded-contexts.md` BC-03 e BC-05; `glossario-linguagem-ubiqua.md`  

**Problema:**  
ADR-0002 diz *"histórico limitado a 3 turnos anteriores"*. BC-03 faz referência a *"histórico de turnos"*. BC-05 define "Turno" em seu glossário como *"Um par pergunta-resposta dentro de uma sessão de conversa."*

O glossário de linguagem ubíqua (`glossario-linguagem-ubiqua.md`) não contém o termo "turno". O termo aparece apenas no glossário de BC-05, tornando-o local a esse contexto.

Mais importante: se um turno = 1 pergunta + 1 resposta, então 3 turnos = 6 mensagens. Mas se o atendente fez uma pergunta de esclarecimento no turno de ambiguidade (S5), e a resposta foi apenas as opções de clarificação, isso conta como 1 turno? O contexto de 3 turnos pode estar incluindo interações "vazias" (sem conteúdo factual) no lugar de interações úteis.

**Proposta de resolução:**  
Adicionar "Turno" ao glossário de linguagem ubíqua com definição precisa. Definir se interações de esclarecimento (pergunta ambígua → opções → escolha) contam como 1 ou 2 turnos. Atualizar VC-12 com esse detalhe.

---

### AMB-08 — O mockup S3 exibe nome do documento no alerta de conflito, mas requirements não define qual formato

**Severidade:** 🟡 Importante  
**Localização:** `requirements.md` Constraints "Tratamento explícito de contradições" vs mockup S3  

**Problema:**  
O requirements exige *"incluir alerta explícito identificando o documento divergente pelo nome e versão."* O mockup S3 exibe o alerta assim: *"A PROC-042-v1 ainda consta ativa no SharePoint sem marcação de revogação."*

Dois problemas:
1. O alerta menciona o estado do documento no SharePoint (*"ainda consta ativa"*) — mas o query endpoint não tem visibilidade direta do estado do SharePoint. Essa informação viria de BC-04 (Governança Documental), que está fora do escopo do query endpoint. O mockup exibe informação que o endpoint não pode saber.
2. O requirements diz "identificar o documento divergente" — mas não define se o alerta deve aparecer na resposta como texto inline, num bloco separado, ou nos metadados. O mockup mostra como bloco separado com fundo amarelo, mas isso é uma decisão de UI sem especificação no requirements.

**Proposta de resolução:**  
(1) Remover do alerta qualquer referência ao estado do documento no SharePoint — o endpoint só sabe que dois chunks divergem, não por que a v1 ainda está ativa. Reformular para: *"Atenção: existe outro documento na base com valor diferente para este tema (PROC-042 v1.0, Seção 2.1). A resposta acima usa a versão mais recente (PROC-042-v2)."*  
(2) Especificar no requirements o formato do alerta de conflito (sugestão: campo obrigatório `conflict_notice` no bloco de rastreabilidade, sempre visível).

---

### AMB-09 — VC-08 (tempo de resposta) não define o que é medido quando a resposta é streaming

**Severidade:** 🟡 Importante  
**Localização:** `requirements.md` VC-08 e Constraints "Tempo de resposta"  

**Problema:**  
VC-08 define: *"a resposta completa deve ser entregue ao atendente em menos de 30 segundos, medidos do lado do cliente (Teams ou painel web)."*

O mockup S7 mostra indicador de *"3,2 s"* durante carregamento, sugerindo que a resposta é entregue completa de uma vez. Mas arquiteturas RAG com GPT-4o tipicamente usam streaming (tokens chegam progressivamente). Se houver streaming:
- "Resposta completa entregue" = último token recebido pelo cliente?
- Ou = primeiro token visível (Time to First Token — TTFT)?
- O timer no mockup S2 (*"2,4 s"*) representa qual dos dois?

Para o atendente em chamado ativo, o TTFT importa mais do que o tempo total — ele começa a ler enquanto a resposta chega. O SLA de 30s pode ser muito restritivo para tempo total em respostas longas, mas razoável para TTFT.

**Proposta de resolução:**  
Definir se a arquitetura usa streaming ou não. Se streaming: especificar SLA separado para TTFT (ex: < 3s) e para resposta completa (ex: < 30s). Atualizar VC-08 e o mockup S7 para refletir o modelo de entrega.

---

### AMB-10 — "Nível 1", "Nível 4", "Nível" aparecem nos BCs mas não estão no glossário de linguagem ubíqua

**Severidade:** 🟡 Importante  
**Localização:** `bounded-contexts.md` BC-01 glossário; `glossario-linguagem-ubiqua.md`; `requirements.md` vários  

**Problema:**  
BC-01 define "Nível" em seu glossário local como *"Hierarquia de confiabilidade: 1 (normativo formal) → 4 (FAQ informal)."* BC-03 usa o termo *"Nível 1"* como critério de cálculo do nível de confiança. O requirements usa *"Nível 1 (normativo formal)"* e *"Nível 4 (FAQ informal)"* sem definição.

O glossário de linguagem ubíqua do projeto (`glossario-linguagem-ubiqua.md`) foi extraído dos documentos da NovaTech (POL-001, PROC-042, SLA-2024, FAQ) e não contém os termos "Nível", "Nível 1", "Nível 4" nem a hierarquia de fontes — conceitos que são internos ao sistema de IA, não ao negócio da NovaTech.

Isso cria dois vocabulários paralelos: o vocabulário do domínio de negócio (glossário) e o vocabulário do sistema (espalhado nos BCs). Um dev novo consultando o glossário para entender "Nível 1" não o encontrará.

**Proposta de resolução:**  
Criar uma seção "Termos do Sistema" no glossário (separada dos termos de negócio) com: Nível, Chunk, Score de relevância, Threshold, Embedding, Status do documento, Pacote de contexto. Alternativamente, criar um glossário técnico separado em `glossario-sistema.md`.

---

### AMB-11 — BC-04 e BC-06 têm responsabilidades sobrepostas no painel de gaps

**Severidade:** 🟡 Importante  
**Localização:** `bounded-contexts.md` BC-04 (dentro) vs BC-06 (dentro)  

**Problema:**  
BC-04 inclui em seu escopo: *"Gerenciar o painel de gaps: consolidar perguntas sem resposta por tema (recebidas de BC-06), apresentar tendência mensal e status de cada gap (aberto, em cobertura, resolvido)."*

BC-06 inclui em seu escopo: *"Fornecer dados agregados para o painel de gaps de BC-04: volume de perguntas sem resposta por tema, tendência mensal, temas mais consultados."*

Problema: BC-04 *gerencia* o painel; BC-06 *fornece os dados* para o painel. Mas quem *exibe* o painel? Não está definido. É BC-05? É uma interface separada? O mockup não cobre o painel de gaps.

Além disso, BC-06 lista *"tendência mensal"* como dado que fornece ao BC-04 — mas BC-04 também lista *"apresentar tendência mensal"* como sua responsabilidade. A tendência é calculada em BC-06 ou em BC-04?

**Proposta de resolução:**  
Definir explicitamente: BC-06 armazena os eventos brutos e calcula as métricas agregadas (incluindo tendência); BC-04 consome as métricas e toma decisões editoriais com base nelas; a exibição do painel é responsabilidade de uma interface administrativa (BC-05 expandido, ou uma tela separada fora do escopo da fase 1).

---

### AMB-12 — O mockup S1 mostra "chips clicáveis" que preenchem o campo — comportamento não está especificado nos requirements

**Severidade:** 🟡 Importante  
**Localização:** `novatech-mockups.html` S1 anotação 1 vs `requirements.md`  

**Problema:**  
O mockup S1 descreve na anotação: *"Clicáveis: preenchem o campo de entrada com pergunta de exemplo para reduzir fricção de início."*

Esse comportamento (sugestões clicáveis de pergunta no onboarding) não aparece em nenhum lugar do `requirements.md` — nem como funcionalidade, nem como constraint, nem como critério de verificação. Um dev implementando a partir do requirements não saberia que deve construir esse comportamento.

Adicionalmente: as perguntas de exemplo nos chips do mockup são genéricas ("Prazos de entrega", "Regras de frete") — não são perguntas reais pré-definidas. O sistema deveria ter um banco de perguntas de exemplo? Quem mantém esse banco? Isso é conteúdo de produto ou configuração técnica?

**Proposta de resolução:**  
Se os chips de sugestão são desejados, adicionar ao `requirements.md` In Scope como funcionalidade de onboarding com especificação: lista de sugestões configurável, comportamento ao clicar (preenche o campo, não envia automaticamente), responsável pela manutenção das sugestões (produto ou configuração).

---

### AMB-13 — VC-09 (documento > 6 meses) conflita com a constraint de exibição de fontes

**Severidade:** 🟡 Importante  
**Localização:** `requirements.md` VC-09 vs Constraints "Exibição obrigatória de fontes"  

**Problema:**  
A constraint "Exibição obrigatória de fontes" lista os campos obrigatórios: *"nome do documento, número de versão, seção de referência, data da última atualização do documento e nível de confiança."*

VC-09 define que, quando um documento tem mais de 6 meses sem atualização, deve aparecer *"aviso explícito indicando que o documento não é atualizado há mais de 6 meses."*

Problema: a constraint já exige a data do documento em toda resposta. Se a data já está sempre visível, o atendente poderia calcular a antiguidade sozinho. O aviso de VC-09 é redundante com a data exibida — ou adiciona valor? Não está claro se o aviso é um campo adicional ou uma reformatação da data.

Mais importante: VC-09 define o critério como *"mais de 6 meses sem atualização confirmada pelo responsável de base"* — mas a data exibida ao atendente é a "data da última atualização do documento", não a "data da última confirmação de vigência pelo responsável de base". São datas diferentes. Um documento pode ter 8 meses de data de atualização mas ter sido confirmado como vigente há 2 meses. O critério do aviso deveria ser baseado em qual das duas datas?

**Proposta de resolução:**  
Definir que o aviso de 6 meses é baseado na *data da última confirmação de vigência* (gerenciada por BC-04), não na data do documento. Especificar que a data exibida ao atendente é a do documento (para rastreabilidade) e que o aviso é um campo adicional derivado do metadado de vigência de BC-04. Atualizar VC-09 com essa precisão.

---

## 🟢 Inconsistências Menores

---

### AMB-14 — "Versão do assistente" aparece nos requisitos de rastreabilidade mas não nos mockups

**Severidade:** 🟢 Menor  
**Localização:** `especificacao-requisitos-rag-v2.md` Seção 6.1; `requirements.md` (ausente); mockups (ausente)  

**Problema:**  
A especificação de requisitos RAG v2.0 define *"Versão do assistente"* como campo obrigatório de rastreabilidade em toda resposta. O `requirements.md` não menciona esse campo na constraint de exibição obrigatória de fontes. Os mockups S2, S3 e S6 mostram o bloco de rastreabilidade sem esse campo.

**Proposta de resolução:**  
Decidir se a versão do assistente é visível ao atendente ou apenas nos logs internos. Atualizar o `requirements.md` e os mockups de acordo.

---

### AMB-15 — O glossário usa "Tier" com T maiúsculo mas os mockups usam minúsculo inconsistentemente

**Severidade:** 🟢 Menor  
**Localização:** `glossario-linguagem-ubiqua.md` entrada "Tier"; mockups S1 chips ("tier de cliente"), S6 ("Cliente Gold")  

**Problema:**  
O glossário define "Tier" (maiúsculo) como termo oficial da linguagem ubíqua do negócio. Os mockups alternam entre "tier" (minúsculo) e "cliente Gold" (sem usar "tier"). As respostas do assistente em S2 e S6 usam "Gold" sem o prefixo "tier".

É um detalhe menor, mas o assistente vai gerar texto que os atendentes podem copiar para comunicação com clientes — consistência terminológica importa.

**Proposta de resolução:**  
Padronizar nos mockups e no `requirements.md` o uso de "tier" (minúsculo, por ser substantivo comum em português) com o valor sempre capitalizado: "tier Gold", "tier Silver", "tier Standard". Atualizar o glossário para refletir o uso canônico em português.

---

### AMB-16 — O mockup S7 mostra o campo de input desabilitado durante o carregamento, mas requirements não especifica esse comportamento

**Severidade:** 🟢 Menor  
**Localização:** `novatech-mockups.html` S7 vs `requirements.md`  

**Problema:**  
O mockup S7 desabilita o campo de input e o botão de envio durante o processamento de uma resposta. Esse comportamento de UX (bloquear nova pergunta enquanto processa) não está especificado nos requirements. Um dev pode ou não implementá-lo, e o QA não tem critério para testá-lo.

**Proposta de resolução:**  
Adicionar comportamento de estado de carregamento ao `requirements.md` em Scope Boundaries ou como critério de acessibilidade: "O campo de entrada deve ser desabilitado durante o processamento de uma resposta para evitar envio concorrente."

---

### AMB-17 — "Condições normais de operação" em VC-08 não está definido

**Severidade:** 🟢 Menor  
**Localização:** `requirements.md` VC-08  

**Problema:**  
VC-08 define: *"o assistente está em operação em condições normais (sem degradação de infraestrutura)."* O termo "condições normais" não está definido — carga máxima sustentada, número de requisições simultâneas, latência de rede esperada. Isso torna o VC parcialmente não testável: o QA não sabe qual carga aplicar para simular "condições normais."

**Proposta de resolução:**  
Adicionar ao VC-08 a definição operacional de "condições normais": ex. *"carga de até 200 requisições/hora (baseado em 320 chamados/dia × 60% com consulta documental), com latência de rede interna abaixo de 50ms para os serviços Azure."*

---

## Matriz Consolidada

| ID | Severidade | Artefatos afetados | Responsável sugerido |
|---|---|---|---|
| AMB-01 | 🔴 Bloqueante | bounded-contexts BC-02 + BC-03, requirements In Scope | Tech Lead + Dev Sênior |
| AMB-02 | 🔴 Bloqueante | bounded-contexts BC-02, mockups S5 | Tech Lead + Product Specialist |
| AMB-03 | 🔴 Bloqueante | bounded-contexts BC-02, requirements ADR-0004 | Tech Lead + Dev Sênior |
| AMB-04 | 🔴 Bloqueante | requirements Constraints, bounded-contexts BC-03, spec RAG v2 | Product Specialist + Tech Lead |
| AMB-05 | 🔴 Bloqueante | requirements Out of Scope, bounded-contexts BC-01 + BC-06, mockups S2 | Tech Lead + Product Specialist |
| AMB-06 | 🟡 Importante | requirements In Scope + VC-06, bounded-contexts BC-03 | Dev Pleno + QA |
| AMB-07 | 🟡 Importante | requirements ADR-0002 + VC-12, bounded-contexts BC-03 + BC-05, glossário | Tech Lead |
| AMB-08 | 🟡 Importante | requirements Constraints, mockups S3 | Tech Lead + UX |
| AMB-09 | 🟡 Importante | requirements VC-08 + Constraints, mockups S2 + S7 | Tech Lead + Dev Sênior |
| AMB-10 | 🟡 Importante | glossário, bounded-contexts BC-01 + BC-03, requirements | Product Specialist |
| AMB-11 | 🟡 Importante | bounded-contexts BC-04 + BC-06 | Tech Lead |
| AMB-12 | 🟡 Importante | mockups S1, requirements (ausente) | Product Specialist + UX |
| AMB-13 | 🟡 Importante | requirements VC-09 + Constraints | Product Specialist + Tech Lead |
| AMB-14 | 🟢 Menor | spec RAG v2, requirements, mockups S2 + S3 + S6 | Product Specialist |
| AMB-15 | 🟢 Menor | glossário, mockups | UX + Product Specialist |
| AMB-16 | 🟢 Menor | mockups S7, requirements | Product Specialist |
| AMB-17 | 🟢 Menor | requirements VC-08 | QA + Tech Lead |

---

## Ordem de resolução recomendada

Antes de qualquer linha de código de produção, resolver nesta sequência:

1. **AMB-01** — Contrato de interface BC-02 → BC-03 (pergunta ambígua): define o payload da API interna mais crítica do sistema.
2. **AMB-03** — Threshold de confiança: sem esse valor, o pipeline de testes não pode ser escrito.
3. **AMB-04** — Critérios de nível de confiança: sem eles, BC-03 não pode ser implementado nem testado.
4. **AMB-05** — Fronteira do feedback no query endpoint: define o escopo real do que o QA vai testar.
5. **AMB-09** — Streaming vs resposta completa: afeta a arquitetura de rede e o design do estado de carregamento (S7).
6. **AMB-02** — Tipos de ambiguidade: alinha mockup com comportamento real antes de construir a UI.
7. **AMB-08** — Formato do alerta de conflito: requisito de produto que afeta diretamente o sistema prompt de BC-03.
8. Demais itens podem ser resolvidos em paralelo durante o desenvolvimento.

---

*Revisão baseada nos artefatos listados no cabeçalho. Todos os itens identificados são derivados de leitura cruzada da documentação existente — não representam julgamento sobre o conteúdo de negócio, apenas sobre a precisão e consistência interna dos artefatos técnicos.*
