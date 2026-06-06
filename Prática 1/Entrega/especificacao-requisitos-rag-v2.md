# Especificação de Requisitos de Produto — Pipeline de RAG
## Assistente de IA para Atendimento NovaTech — v2.0

**Elaborado por:** Product Specialist  
**Data:** 05/06/2026  
**Versão:** 2.0 (substitui v1.0 de 05/06/2026)  
**Contexto:** Projeto DB1 × NovaTech — Assistente de IA integrado ao Microsoft Teams + SharePoint  
**Fontes de entrada:** Cenário operacional, mapa de temas e gaps, análise de inconsistências, cruzamento FAQ × normativos, jornada do atendente, diagrama visual de fluxo, Anexo A (documentação simulada), Anexo B (chunks de referência RAG), Especificação v1.0

> Este documento especifica os requisitos de produto do pipeline de RAG em linguagem de negócio — sem código ou arquitetura técnica. Destina-se a alinhar time de produto, stakeholders da NovaTech e equipe de desenvolvimento sobre o que o sistema deve e não deve fazer antes de qualquer decisão de implementação.

---

## Registro de Alterações em Relação à v1.0

| Seção | Alteração |
|---|---|
| 1 — Fontes | Adicionados critérios objetivos de elegibilidade por nível; formato de arquivo; critério de idioma; procedimento de triagem para Confluence com responsável e prazo definidos |
| 2 — Exclusões | Adicionada regra dinâmica para documentos futuros com metadados incompletos; definido processo de quarentena; incluído critério de revisão periódica da lista |
| 3 — Contradições | Adicionado tratamento para conflito entre documentos do mesmo nível; definido protocolo de escalada quando NovaTech não resolve contradição antes do go-live; adicionada cláusula de desempate por data quando hierarquia for igual |
| 4 — Sem resposta | Adicionados cenários de pergunta ambígua e pergunta multi-partes; distinção entre ausência por falta de documento vs. baixa similaridade; comportamento para perguntas sensíveis |
| 5 — Atualização | Adicionado responsável pelo push de reindexação; SLA de contingência; processo de rollback; comunicação ao atendente durante janelas de atualização |
| 6 — Rastreabilidade | Definido formato de apresentação ao atendente; retenção de dados de feedback; conformidade LGPD; critério de anonimização; adicionada rastreabilidade de versão do assistente |
| 7 — Pré-condições | Adicionadas critérios de aceite verificáveis para cada pré-condição |
| 8 — Fora do escopo | Adicionado protocolo de comunicação ao atendente sobre escopo e processo de expansão de escopo por fase |
| 9 — Qualidade de resposta | Seção nova: critérios mínimos de qualidade, métricas de sucesso e processo de avaliação contínua |
| 10 — Acessibilidade e idioma | Seção nova |

---

## 1. Fontes de Dados que Devem ser Indexadas

### 1.1 Critério geral de elegibilidade

Para ser indexado em qualquer nível, um documento deve atender a todos os critérios abaixo:

- Estar redigido em português brasileiro.
- Estar em formato legível pelo pipeline: `.pdf`, `.docx`, `.md`, `.xlsx`, ou página HTML do Confluence/SharePoint. Arquivos de imagem, `.pptx` ou formatos proprietários não suportados devem ser convertidos antes da indexação.
- Ter pelo menos um dos seguintes metadados preenchidos: responsável formal, data de criação ou data de última atualização. Documentos sem nenhum metadado identificável vão para quarentena (ver Seção 2.4).
- Tratar de processos, políticas, procedimentos ou referências operacionais relacionados ao atendimento ao cliente da NovaTech. Documentos de RH, TI interna, projetos de infraestrutura ou comunicação institucional não são elegíveis.

### 1.2 Hierarquia de fontes e prioridade de resposta

O pipeline organiza as fontes em quatro níveis. Quando dois documentos de níveis diferentes cobrem o mesmo tema com informações divergentes, o documento de nível mais baixo (número menor) sempre prevalece. Quando dois documentos do mesmo nível divergem, aplica-se o critério de desempate da Seção 3.4.

---

#### Nível 1 — Documentos normativos formais (máxima prioridade)

Documentos com responsável formal designado, versão numerada e data de atualização registrada, emitidos por uma Diretoria ou área de Compliance. São a fonte de verdade do assistente.

**Documentos elegíveis no momento do go-live:**

| Documento | Versão | Localização | Responsável | Última atualização |
|---|---|---|---|---|
| POL-001 — Política de Devolução de Mercadorias | 3.1 | SharePoint — Pasta Operações/Normativos | Diretoria de Operações | 15/01/2024 |
| PROC-042-v2 — Frete Especial Revisado | 2.0 | SharePoint — Pasta Comercial/Procedimentos | Diretoria Comercial | 10/11/2023 |
| SLA-2024 — Tabela de SLA por Tipo de Cliente | 2024.1 | SharePoint — Pasta Comercial/Contratos | Diretoria Comercial + Operações | 02/01/2024 |

**Critério de entrada para novos documentos no Nível 1:**  
Todo novo documento que pretenda entrar no Nível 1 deve ser acompanhado de um formulário de cadastro preenchido pela área responsável, contendo: nome do documento, número de versão, responsável formal (nome e cargo), data de emissão, e declaração de que o documento substitui ou complementa qual documento anterior (se aplicável). Documentos enviados sem esse formulário vão para o Nível 3 até regularização.

**Critério de saída do Nível 1:**  
Um documento sai do Nível 1 quando: (a) é formalmente substituído por nova versão, (b) é revogado pela área responsável, ou (c) permanece sem atualização por mais de 18 meses e não recebe confirmação de vigência pela área responsável após notificação do time de produto.

---

#### Nível 2 — Planilhas e tabelas de referência operacional

Documentos tabulares atualizados em ciclo mensal ou trimestral, com data de vigência explícita. Complementam os documentos do Nível 1 com valores numéricos variáveis (tarifas, tabelas de preço, escalas regionais).

**Documentos elegíveis no momento do go-live:**

| Documento | Formato | Localização | Ciclo de atualização | Responsável |
|---|---|---|---|---|
| Tabela base de fretes (`frete-base-AAAAMM.xlsx`) | `.xlsx` | Pasta de rede `\\novatech-fs\comercial\tabelas\` | Mensal | Diretoria Comercial |

**Critério de entrada para novos documentos no Nível 2:**  
Planilha ou tabela com: responsável identificado, data de vigência explícita no nome do arquivo ou na primeira aba, e aprovação da área responsável por e-mail ou registro no sistema de chamados interno.

**Comportamento do assistente para documentos do Nível 2:**  
O assistente deve sempre indicar o mês de referência da planilha utilizada. Se a planilha for do mês anterior ao corrente, o assistente inclui automaticamente o aviso: "Atenção: esta tabela é de referência de [mês/ano]. Confirme se há versão mais recente com o responsável da área antes de usar o valor em cotação."

---

#### Nível 3 — Wiki interna (Confluence)

Páginas do Confluence com conteúdo normativo ou procedural validado. Exigem triagem prévia antes da indexação, pois o Confluence não tem controle de versão obrigatório e mistura conteúdo de diferentes estados de maturidade.

**Critério de triagem para o Nível 3:**  
Uma página do Confluence é elegível se: (a) tiver autor identificado, (b) tiver sido editada nos últimos 12 meses, (c) não estiver marcada como rascunho ou em elaboração, e (d) for aprovada pela área de Operações ou Compliance como representativa de processo vigente. A triagem é feita pelo responsável de base designado por cada área, com prazo de 5 dias úteis por lote de até 20 páginas.

**Páginas não elegíveis para o Nível 3** (independentemente de conteúdo):  
Páginas de projetos internos, atas de reunião, apresentações convertidas em wiki, comparativos de ferramentas, decisões arquitetônicas de TI, e qualquer página com o prefixo "RASCUNHO", "WIP", "DRAFT" ou equivalente no título.

---

#### Nível 4 — FAQ-Atendimento (fonte auxiliar com restrições severas)

O FAQ-Atendimento é um documento informal mantido sem responsável designado, sem versão e sem validação por Compliance ou Operações. Deve ser tratado como fonte de último recurso, disponível apenas para temas sem cobertura nos Níveis 1, 2 ou 3.

**Itens elegíveis do FAQ no go-live** (após correção das contradições identificadas):

| Item do FAQ | Tema | Status de elegibilidade | Condição |
|---|---|---|---|
| Item 3 | Devolução de carga perigosa | Elegível com ressalva | Alinhado com POL-001 Seção 3.2; apresentar sempre com marcação de baixa confiança |
| Item 15 | Tier Platinum inexistente | Elegível | Alinhado com SLA-2024 Seção 1 |
| Item 27 | Rastreamento com atraso | Elegível com ressalva | Único documento sobre o tema; sinalizar divergência de limiar com SLA-2024 (R$50k vs R$100k) |
| Item 38 | Carga danificada em trânsito | Elegível com ressalva | Único documento sobre o tema; sinalizar que é fonte informal sem respaldo normativo |
| Item 41 | Diferença entre SLA de resposta e resolução | Elegível | Alinhado com SLA-2024 Seção 2 |

**Itens bloqueados do FAQ no go-live:**

| Item do FAQ | Tema | Motivo do bloqueio | Condição de desbloqueio |
|---|---|---|---|
| Item 8 | Frete especial — prazo e versão da PROC | Omite prazo de +3 dias (v2) e não resolve ambiguidade de versão | Reescrita alinhada à PROC-042-v2 |
| Item 22 | Seguro de carga | Percentuais sem respaldo normativo | Publicação de normativo formal de seguro |
| Item 32 | Carga perigosa com frete expresso | Processo sem respaldo em PROC ou POL vigente | Publicação da PROC-043 revisada |
| Item 45 | Desconto por volume de frete | Limiar errado (>10 fretes em vez de ≥8) e percentuais ausentes | Reescrita alinhada à PROC-042-v2 Seção 4 |

**Comportamento do assistente para qualquer item do FAQ:**  
Toda resposta baseada no FAQ deve conter o seguinte bloco de aviso, sem exceção: "Esta informação é baseada no FAQ de Atendimento, documento informal não validado por Compliance ou Operações. Use com cautela e confirme com o supervisor em casos de impacto financeiro ou contratual."

---

### 1.3 Critério de idioma e formato

O assistente opera exclusivamente em português brasileiro. Documentos em outros idiomas não são indexados na fase 1. Documentos bilíngues são indexados apenas nas seções em português, com marcação explícita de que a versão em outro idioma existe mas não foi indexada.

Formatos aceitos para indexação: `.pdf` com texto selecionável (PDFs escaneados sem OCR não são aceitos), `.docx`, `.md`, `.txt`, `.xlsx` (conteúdo das células, não fórmulas), e páginas HTML do SharePoint e Confluence acessíveis via API.

---

## 2. Documentos que Devem ser Excluídos da Indexação

### 2.1 Lista de exclusões confirmadas no go-live

| Documento | Localização conhecida | Motivo de exclusão | Condição de inclusão futura |
|---|---|---|---|
| PROC-042 v1.0 (mar/2023) | SharePoint — Pasta Comercial/Procedimentos | Substituída pela v2.0; coexistência gera contradições em multiplicadores, fatores de peso, prazo e descontos. Deve ser arquivada com status "revogado" no SharePoint antes da indexação. | Nunca — deve permanecer arquivada após declaração formal de revogação |
| FAQ — Item 8 (frete especial) | FAQ-Atendimento | Omite prazo de +3 dias úteis (v2); não resolve ambiguidade entre versões | Reescrita validada pela Diretoria Comercial |
| FAQ — Item 22 (seguro de carga) | FAQ-Atendimento | Percentuais sem respaldo normativo; risco de cotação incorreta com impacto contratual | Publicação de normativo formal de seguro de carga |
| FAQ — Item 32 (carga perigosa + frete expresso) | FAQ-Atendimento | Processo sem respaldo em nenhum PROC ou POL vigente; PROC-043 em revisão | Publicação da PROC-043 revisada pelo Compliance |
| FAQ — Item 45 (desconto por volume) | FAQ-Atendimento | Limiar errado (>10 fretes); percentuais e base de cálculo ausentes; contradiz PROC-042-v2 | Reescrita validada pela Diretoria Comercial |
| PROC-043 — Frete de Cargas Perigosas (versão em revisão) | Compliance | Documento em elaboração ativa; indexar versão instável introduz informação que pode mudar a qualquer momento | Publicação oficial pelo Compliance com número de versão e data |
| PROC-088 — Interceptação de Carga | Não localizado | Referenciado pela POL-001 mas não disponível para indexação | Localização e validação pela Diretoria de Operações |
| Rascunhos e documentos com prefixo DRAFT/WIP/RASCUNHO | SharePoint e Confluence | Representam intenção de mudança, não regra vigente | Aprovação formal e remoção do prefixo pela área responsável |
| Documentos de projetos internos de TI | Confluence | Fora do escopo operacional do atendimento | Não elegíveis na fase 1 |
| Atas de reunião | SharePoint e Confluence | Registram discussão, não decisão formal | Não elegíveis — decisões devem ser formalizadas em PROC ou POL |

### 2.2 Critério dinâmico para documentos futuros desconhecidos

Para documentos que chegarem após o go-live sem enquadramento óbvio nas categorias acima, o critério de exclusão automática se aplica quando qualquer uma das condições abaixo for verdadeira:

- O documento não tem responsável formal identificável.
- O documento não tem data de criação ou última atualização.
- O documento contradiz um documento de Nível 1 existente sem declaração explícita de revisão ou substituição.
- O documento está em formato não suportado pelo pipeline (ver Seção 1.3).
- O documento foi enviado por um usuário sem permissão de publicação na fonte de dados correspondente.

Documentos que ativam qualquer um desses critérios vão para quarentena (ver Seção 2.4) em vez de serem indexados ou descartados imediatamente.

### 2.3 Revisão periódica da lista de exclusões

A lista de exclusões deve ser revisada trimestralmente pelo responsável de produto em conjunto com os responsáveis de base das três áreas (Operações, Compliance, Comercial). O objetivo é: (a) verificar se documentos bloqueados foram regularizados e podem ser desbloqueados, e (b) identificar novos documentos que devem ser excluídos.

### 2.4 Processo de quarentena

Documentos que não atendem aos critérios de elegibilidade mas não estão explicitamente listados como excluídos entram em quarentena: ficam registrados no sistema com status "aguardando triagem", não são indexados, e o responsável de base da área correspondente recebe notificação em até 24 horas para decidir: indexar, ajustar metadados e indexar, ou descartar. Sem decisão em 5 dias úteis, o documento é descartado da fila e o solicitante é notificado.

---

## 3. Como Lidar com Documentos Contraditórios

### 3.1 Contradições identificadas antes do go-live — resolução obrigatória

Para cada contradição confirmada abaixo, a NovaTech deve registrar formalmente a resolução antes que o pipeline inicie a indexação. Resolução não concluída bloqueia o go-live para os temas afetados, não para o produto inteiro.

| Contradição | Documentos envolvidos | Decisão esperada | Responsável pela decisão | Critério de aceite |
|---|---|---|---|---|
| Multiplicadores regionais, fatores de peso, prazo de manuseio | PROC-042 v1 × PROC-042-v2 | Declarar v2 como vigente; arquivar v1 com status "revogado" no SharePoint | Diretoria Comercial | E-mail ou documento assinado confirmando a revogação da v1, com data |
| Limiar de desconto por volume (>10 vs ≥8 fretes/mês) | FAQ item 45 × PROC-042-v2 | Bloquear item 45 do FAQ; PROC-042-v2 prevalece | Diretoria Comercial | Confirmação de que o item 45 foi removido do escopo de indexação ou reescrito |
| Prazo adicional de manuseio (+2 vs +3 dias úteis) | FAQ item 8 × PROC-042-v2 | Bloquear item 8 do FAQ; PROC-042-v2 prevalece | Diretoria Comercial | Idem acima |
| Limiar de escalada por valor de carga (R$50k vs R$100k) | FAQ item 27 × SLA-2024 | Definir qual limiar é o critério oficial para escalada interna | Diretoria Comercial + Operações | Decisão registrada em PROC ou adendo ao SLA-2024 |

**O que acontece se a NovaTech não resolver uma contradição antes do go-live:**  
Os temas cobertos pelos documentos em conflito ficam fora do escopo do assistente na fase 1. O assistente declara ao atendente que não há resposta disponível para aquele tema neste momento e orienta a consultar o supervisor. Isso é preferível a servir uma resposta com base em documento de hierarquia não resolvida.

### 3.2 Marcação de status por documento indexado

Cada documento na base carrega um rótulo de status que determina como o assistente o utiliza. O rótulo é atribuído no momento da indexação e pode ser alterado pelo responsável de base da área correspondente.

| Status | Critério de atribuição | Comportamento do assistente | Quem atribui |
|---|---|---|---|
| Vigente | Documento ativo com responsável confirmado e sem versão posterior declarada | Fonte primária; apresentado sem ressalva adicional | Responsável de base da área |
| Versão atual (substitui anterior) | Declarado formalmente como substituto de versão anterior | Fonte primária; se houver versão anterior arquivada, o assistente menciona isso apenas se o atendente perguntar | Responsável de base + confirmação da área |
| Arquivado | Versão revogada formalmente | Nunca incluído em respostas; mantido na base apenas para log de auditoria | Responsável de base + confirmação da área |
| Em revisão | Documento em atualização declarada; versão atual ainda em vigor | Usado apenas se não houver outro documento sobre o tema; resposta inclui aviso de instabilidade | Responsável de base |
| Informal — uso restrito | Documento sem validação normativa (ex: FAQ) | Último recurso; apresentado sempre com aviso de fonte informal; nunca usado quando há documento de nível superior sobre o tema | Atribuído automaticamente para documentos do Nível 4 |

### 3.3 Comportamento do assistente ao detectar conflito em tempo real

Se o pipeline recuperar chunks de documentos com informações contraditórias para a mesma pergunta, o assistente deve seguir esta sequência de decisão:

**Passo 1 — Identificar a hierarquia:**  
Verificar o nível (1 a 4) de cada documento. O documento de nível mais baixo prevalece automaticamente. Se os documentos forem do mesmo nível, passar ao passo 2.

**Passo 2 — Desempate por data dentro do mesmo nível:**  
Se dois documentos do mesmo nível divergirem, o mais recente (pela data de última atualização registrada nos metadados) prevalece. Isso se aplica ao Nível 3 (Confluence), onde podem existir múltiplas páginas sobre o mesmo processo com datas diferentes.

**Passo 3 — Conflito irresolvível:**  
Se os dois documentos têm o mesmo nível e a mesma data (ou data não identificável), o assistente não apresenta nenhuma das versões. Declara que há conflito não resolvido entre dois documentos sobre o tema e orienta o atendente a confirmar com o supervisor qual versão deve ser usada. O conflito é registrado automaticamente como item de feedback para resolução pela equipe de produto.

**Em qualquer caso de conflito detectado (independente de qual versão vence):**  
O assistente sempre inclui o seguinte alerta na resposta: "Atenção: existe outro documento na base com informação diferente sobre este tema — [nome do documento, versão]. A resposta acima é baseada em [nome do documento vencedor, versão], que tem [hierarquia superior / data mais recente]. Confirme com seu supervisor se houver dúvida."

**O assistente nunca deve:**  
Combinar valores de documentos contraditórios na mesma resposta. Apresentar os dois valores como igualmente válidos sem indicar qual prevalece. Omitir a existência do conflito.

### 3.4 Conflito entre documentos do mesmo nível — exemplos práticos

Para evitar ambiguidade na aplicação das regras acima, os seguintes exemplos ilustram o comportamento esperado:

- Duas páginas do Confluence (ambas Nível 3) descrevem processos de rastreamento com prazos diferentes: o assistente usa a mais recente e alerta sobre a existência da outra.
- Um documento de Nível 1 (POL-001) e uma página do Confluence de Nível 3 divergem sobre prazo de triagem: o assistente usa o Nível 1 sem ressalva adicional além do alerta padrão de conflito.
- PROC-042-v2 (Nível 1) e FAQ item 41 (Nível 4) apresentam os mesmos valores de SLA: o assistente responde com base na PROC-042-v2 sem mencionar o FAQ, pois não há conflito.

---

## 4. Comportamento Quando uma Pergunta Não Tiver Resposta na Base

### 4.1 Princípio fundamental

O assistente nunca deve inventar, inferir, estimar ou deduzir informação não encontrada na base. A ausência de resposta é sempre preferível a uma resposta incorreta. Frases como "provavelmente", "normalmente", "costuma ser", "deve ser" ou "em geral" são proibidas em respostas sobre prazos, valores numéricos, condições contratuais e procedimentos operacionais.

### 4.2 Cenários de ausência de resposta e comportamento esperado

**Cenário A — Tema não coberto por nenhum documento indexado**

Exemplos: seguro de carga, frete padrão abaixo de 500 kg, carga danificada em trânsito.

Comportamento: O assistente declara que o tema não está coberto pela base documental disponível. Indica a área responsável pelo tema quando identificável a partir de documentos formais. Oferece ao atendente a opção de registrar a pergunta como gap para análise posterior.

Formato da resposta: "Não encontrei informação sobre [tema] na base documental disponível para o assistente. Este tema não está coberto por nenhum documento indexado no momento. Para obter a resposta correta, consulte [área responsável, se identificável]. Deseja registrar esta pergunta como lacuna da base para que a equipe de produto avalie incluir esse conteúdo em atualizações futuras?"

---

**Cenário B — Documento referenciado existe mas não está indexado**

Exemplos: PROC-088 (Interceptação de Carga) referenciada pela POL-001; PROC-043 (Cargas Perigosas) referenciada pela PROC-042-v2.

Comportamento: O assistente informa que sabe da existência do documento mas que ele não está disponível na base do assistente, e identifica o documento pelo nome.

Formato da resposta: "A documentação da NovaTech menciona [nome do documento] como referência para este tema, mas esse documento não está disponível na base do assistente neste momento. Consulte diretamente [área responsável indicada no documento que faz a referência]."

---

**Cenário C — Documento em revisão é a única fonte disponível**

Exemplo: PROC-043 enquanto em revisão pelo Compliance.

Comportamento: O assistente informa que o único documento disponível sobre o tema está em revisão e que seu conteúdo pode mudar. Não apresenta o conteúdo do documento em revisão como definitivo.

Formato da resposta: "O documento que trata deste tema — [nome] — está atualmente em revisão pelo Compliance e pode sofrer alterações. Não é possível fornecer uma resposta definitiva com base nele. Consulte diretamente a área de Compliance para obter a orientação vigente."

---

**Cenário D — Pergunta ambígua que pode ser interpretada de formas distintas**

Exemplo: "Qual o prazo?" (sem especificar se é prazo de devolução, de entrega, de SLA ou de reembolso).

Comportamento: O assistente não escolhe uma interpretação arbitrária. Solicita esclarecimento ao atendente com as opções possíveis identificadas a partir da base.

Formato da resposta: "Sua pergunta pode se referir a diferentes prazos presentes na documentação. Qual dos seguintes você está consultando? (a) Prazo para solicitação de devolução após entrega — coberto pela POL-001; (b) Prazo de entrega para frete especial — coberto pela PROC-042-v2; (c) SLA de resposta e resolução por tier de cliente — coberto pelo SLA-2024; (d) Outro — descreva o contexto."

---

**Cenário E — Pergunta com múltiplas partes, algumas respondíveis e outras não**

Exemplo: "Qual o prazo de devolução, o seguro de carga e o frete para carga perigosa?"

Comportamento: O assistente responde as partes que têm cobertura documental e declara explicitamente quais partes não têm, sem omitir nenhuma.

Formato da resposta: O assistente estrutura a resposta em blocos numerados por parte da pergunta, com o status de cada uma: respondida (com fonte), parcialmente respondida, ou não respondida (com motivo e próximo passo).

---

**Cenário F — Pergunta fora do escopo da documentação interna**

Exemplos: legislação trabalhista, dados de concorrentes, questões jurídicas externas, dados pessoais de clientes.

Comportamento: O assistente informa que responde exclusivamente com base na documentação interna da NovaTech e que o tema está fora desse escopo. Não tenta responder com conhecimento geral.

Formato da resposta: "O assistente responde exclusivamente com base na documentação interna da NovaTech. [Tema da pergunta] está fora desse escopo. Para essa questão, consulte [área responsável, se identificável, ou 'o canal adequado para o tema']."

---

**Cenário G — Baixa similaridade semântica sem ausência confirmada**

Situação em que o pipeline encontra documentos relacionados ao tema mas com baixa correspondência com a pergunta específica — por exemplo, a pergunta usa terminologia incomum ou é muito técnica.

Comportamento: O assistente informa que encontrou documentos potencialmente relacionados mas que não há correspondência suficientemente clara para fornecer uma resposta confiável. Indica os documentos encontrados para que o atendente os consulte diretamente.

Formato da resposta: "Não encontrei uma resposta direta para sua pergunta, mas os documentos abaixo podem conter a informação que você busca: [lista de documentos com nome, versão e seção sugerida]. Consulte-os diretamente ou reformule sua pergunta com outros termos."

---

### 4.3 O que o assistente nunca faz ao não encontrar resposta

- Inventar ou estimar valores numéricos (prazos, multiplicadores, percentuais, limites financeiros).
- Sugerir contatos (e-mail, ramal, nome de pessoa) que não estejam formalizados em documento de Nível 1 ou 2.
- Apresentar como resposta definitiva o conteúdo de documento marcado como "em revisão" ou "informal".
- Silenciar sobre a ausência de informação — toda não-resposta deve ser explícita e estruturada.
- Repetir a pergunta do atendente de volta sem nenhuma informação adicional.

---

## 5. Tempo para Novos Documentos Estarem Disponíveis no Assistente

### 5.1 Definições

Para efeito desta seção:
- **Publicação oficial** = documento salvo na localização correta (SharePoint ou Confluence) com metadados completos, pelo responsável com permissão de publicação.
- **Disponibilidade no assistente** = documento indexado, chunks gerados, resposta do assistente baseada no novo conteúdo verificável.
- **Push de reindexação** = ação do responsável de base (humano) de notificar o pipeline que um novo documento está pronto para ser indexado.

### 5.2 SLAs de atualização por tipo de documento

| Tipo de atualização | Prazo-alvo | Prazo máximo (SLA de contingência) | Responsável pelo push | O que acontece se o prazo for descumprido |
|---|---|---|---|---|
| Novo documento normativo (POL, PROC, SLA) — Nível 1 | 4 horas após publicação oficial | 24 horas | Responsável de base da área publicadora | Alerta automático ao gestor de produto e ao responsável da área; documento entra na fila de prioridade máxima |
| Revisão de documento existente (nova versão) — Nível 1 | 4 horas após publicação, com arquivamento simultâneo da versão anterior | 24 horas | Responsável de base da área publicadora | Idem acima; versão anterior permanece ativa até a nova ser indexada |
| Planilha de referência mensal — Nível 2 | Último dia útil do mês anterior ao de vigência | Primeiro dia útil do mês de vigência | Responsável de base — Comercial | Assistente continua usando a planilha do mês anterior com aviso explícito de desatualização até a nova ser indexada |
| Correção de item do FAQ — Nível 4 | 48 horas após validação pela área responsável | 72 horas | Responsável de base — Atendimento | O item permanece bloqueado até a indexação da versão corrigida |
| Novo documento identificado na quarentena | 5 dias úteis após notificação ao responsável de base | 7 dias úteis | Responsável de base da área correspondente | Documento é descartado da fila e solicitante é notificado para reenviar |
| Página do Confluence após triagem — Nível 3 | 48 horas após aprovação na triagem | 72 horas | Responsável de base da área responsável pela página | Página permanece fora da base até indexação |

### 5.3 Responsáveis de base por área

Para que os prazos acima sejam operacionalizáveis, a NovaTech deve designar formalmente um responsável de base por área antes do go-live. Cada responsável tem as seguintes obrigações:

- Notificar o time de produto (via canal definido no projeto) a cada publicação, revisão ou revogação de documento em sua área, no prazo indicado na tabela acima.
- Responder às solicitações de triagem do Confluence dentro do prazo.
- Revisar e aprovar itens do FAQ que cobrem temas de sua área antes de serem indexados ou corrigidos.
- Participar da revisão trimestral da lista de exclusões.

| Área | Responsável de base | Escopo de documentos |
|---|---|---|
| Operações | A designar antes do go-live | POL-001 e derivados; PROC-088; normas de segurança de carga; rastreamento |
| Compliance | A designar antes do go-live | PROC-043 e derivados; normas ANTT; documentação de cargas perigosas |
| Comercial | A designar antes do go-live | PROC-042-v2 e derivados; SLA-2024; tabelas de frete; políticas de desconto |
| Atendimento | A designar antes do go-live | FAQ-Atendimento (triagem, correção e validação de itens) |

### 5.4 Processo de rollback

Se um documento for indexado e gerar respostas incorretas identificadas via feedback dos atendentes, o processo de rollback é:

1. O time de produto recebe o feedback e avalia em até 2 horas úteis se o problema é do documento ou do pipeline.
2. Se o problema for do documento (conteúdo incorreto ou desatualizado): o documento é imediatamente marcado como "suspenso — em verificação" e o assistente para de usá-lo como fonte. O responsável de base da área é notificado para providenciar correção.
3. Se o problema for do pipeline (indexação incorreta ou chunk malformado): o documento é reindexado em até 4 horas úteis.
4. O atendente que reportou o problema recebe notificação do encerramento do ciclo.

### 5.5 Comunicação ao atendente durante janelas de atualização

Durante o período em que um documento está sendo reindexado (entre a publicação e a disponibilidade), o assistente deve exibir um banner informativo se o tema consultado for o de um documento em atualização: "Atenção: a documentação sobre [tema] foi atualizada recentemente. As respostas sobre esse tema podem não refletir as mudanças mais recentes. Confirme informações críticas com o supervisor até que a base esteja atualizada."

---

## 6. Requisitos de Rastreabilidade

### 6.1 Rastreabilidade em cada resposta do assistente

Toda resposta fornecida pelo assistente deve apresentar os seguintes campos de rastreabilidade, independentemente do tipo de resposta (completa, parcial ou ausente).

| Campo | Obrigatório? | Descrição | Exemplo |
|---|---|---|---|
| Nome do documento | Sim | Identificador formal completo | POL-001 — Política de Devolução de Mercadorias |
| Versão do documento | Sim | Número de versão registrado nos metadados | v3.1 |
| Seção de referência | Sim | Número e título da seção utilizada | Seção 3.1 — Prazo geral |
| Data da última atualização do documento | Sim | Conforme metadados do documento | 15/01/2024 |
| Nível de confiança | Sim | Alta / Moderada / Baixa (ver critérios na Seção 6.2) | Alta confiança |
| Classificação do documento | Sim | Normativo / Contratual / Procedural / Informal | Normativo |
| Alerta de conflito | Sim (quando aplicável) | Indicação de documento divergente, com nome e versão | ⚠ Existe versão anterior arquivada: PROC-042 v1.0 |
| Alerta de desatualização | Sim (quando aplicável) | Quando documento tem mais de 6 meses sem atualização | ⚠ Este documento não é atualizado há mais de 6 meses. Confirme vigência com a área responsável. |
| Versão do assistente | Sim | Identificador da versão do assistente que gerou a resposta | Assistente NovaTech v1.2 |

**Formato de apresentação ao atendente:**  
Os campos de rastreabilidade devem ser apresentados em bloco colapsável abaixo da resposta principal, visível por padrão mas com opção de ocultar. O bloco deve usar linguagem clara: "Baseado em: [nome do documento], [versão], [seção], atualizado em [data]." Campos de alerta devem aparecer em destaque visual (cor diferente ou ícone) e nunca podem ser ocultados junto ao bloco de rastreabilidade.

### 6.2 Critérios objetivos para o nível de confiança

| Nível | Critério |
|---|---|
| Alta confiança | Resposta baseada exclusivamente em documento(s) de Nível 1; sem conflito detectado; documento atualizado há menos de 6 meses |
| Confiança moderada | Resposta baseada em documento de Nível 1 com data superior a 6 meses sem atualização; OU resposta baseada em documento de Nível 2 ou 3 validado; OU resposta baseada em Nível 1 com conflito resolvido (versão mais recente prevalece mas há versão arquivada) |
| Baixa confiança | Resposta baseada em documento do Nível 4 (FAQ); OU resposta baseada em documento marcado como "em revisão"; OU única fonte disponível tem data de atualização desconhecida |
| Sem confiança (não responde) | Nenhum documento na base cobre o tema com confiança suficiente; assistente declara ausência |

### 6.3 Rastreabilidade de perguntas sem resposta

Perguntas que resultarem em ausência de resposta (cenários A a G da Seção 4.2) devem ser registradas automaticamente com os seguintes campos:

- Texto da pergunta, anonimizado conforme regra de privacidade da Seção 6.6.
- Cenário de ausência identificado (A, B, C, D, E, F ou G).
- Tema inferido da pergunta (para agrupamento e análise de gaps recorrentes).
- Timestamp da consulta.
- Identificador do chamado em atendimento, quando disponível via integração com o sistema de chamados.
- Identificador do atendente (anonimizado conforme Seção 6.6).

Esses registros alimentam um painel de gaps recorrentes, revisado mensalmente pelo gestor de produto com os responsáveis de base das áreas. O painel deve exibir: volume de perguntas sem resposta por tema, tendência mensal, e status de cada gap (aberto, em cobertura, resolvido).

### 6.4 Rastreabilidade do ciclo de feedback

Cada feedback registrado pelo atendente gera um registro rastreável com os seguintes campos obrigatórios:

| Campo | Descrição |
|---|---|
| Identificador único do feedback | Gerado automaticamente pelo sistema |
| Resposta original do assistente | Texto completo e fontes citadas na resposta avaliada |
| Tipo de problema sinalizado | Seleção pelo atendente: Informação desatualizada / Documento errado / Resposta conflita com outro documento / Resposta incompleta / Outro |
| Descrição livre | Texto opcional fornecido pelo atendente |
| Timestamp | Data e hora do registro |
| Versão do assistente | Versão que gerou a resposta avaliada |
| Área para qual foi roteado | Determinada automaticamente pelo tema |
| Status de resolução | Aberto / Em análise / Resolvido / Descartado (com justificativa) |
| Data de encerramento | Preenchida quando status muda para Resolvido ou Descartado |
| Ação tomada | Descrição da correção realizada ou motivo do descarte |

**SLA de resolução de feedbacks:**  
Feedbacks críticos (marcados como ❌ Resposta incorreta em tema de risco alto: cargas perigosas, divergência de versão, prazo contratual): resolução em até 5 dias úteis.  
Feedbacks moderados (⚠ Resposta incompleta): resolução em até 15 dias úteis.  
Feedbacks abertos além do prazo geram alerta automático ao gestor de produto e ao responsável de base da área correspondente.

**Notificação ao atendente:**  
O atendente que registrou o feedback recebe notificação quando o ciclo for encerrado, com descrição resumida da ação tomada ou do motivo do descarte.

### 6.5 Auditoria mínima do sistema

O sistema deve manter os seguintes logs de auditoria, sem possibilidade de edição retroativa:

| Evento auditável | Campos registrados | Retenção mínima |
|---|---|---|
| Indexação de documento | Documento, versão, responsável que fez o push, timestamp, status atribuído | 24 meses |
| Remoção ou arquivamento de documento | Documento, versão, responsável, timestamp, motivo | 24 meses |
| Alteração de status de documento | De / para, responsável, timestamp, justificativa | 24 meses |
| Consultas realizadas pelos atendentes | Pergunta (anonimizada), resposta gerada, fontes utilizadas, versão do assistente, timestamp | 90 dias |
| Feedbacks registrados e ciclo completo de resolução | Todos os campos da Seção 6.4 | 24 meses |
| Rollbacks de documentos | Documento, versão, motivo, responsável, timestamp | 24 meses |

Os logs de consultas são retidos por 90 dias para fins de auditoria de qualidade de resposta. Logs relacionados a feedbacks e gestão de documentos são retidos por 24 meses para fins de governança documental e eventual contestação contratual.

### 6.6 Privacidade e conformidade com LGPD

O assistente opera em contexto de atendimento ao cliente, onde podem transitar dados pessoais de clientes (nome, número de contrato, CNPJ, dados de carga). Os seguintes requisitos de privacidade se aplicam:

- O texto das perguntas dos atendentes registrado nos logs deve ser anonimizado: dados de clientes (nome, CNPJ, número de contrato, endereço) são substituídos por tokens genéricos antes do armazenamento.
- Nenhum dado de cliente é armazenado pelo assistente. O assistente responde com base na documentação interna; dados de clientes específicos (como histórico de fretes ou contratos individuais) não devem ser indexados na base.
- O identificador do atendente nos logs de consulta é substituído por um hash não reversível para fins de análise agregada, preservando a possibilidade de auditoria sem exposição de dados pessoais do funcionário.
- A política de retenção de logs é comunicada aos atendentes no onboarding da ferramenta.

---

## 7. Pré-condições para Lançamento

As pré-condições abaixo devem ser verificadas e documentadas antes do go-live. Cada item tem um critério de aceite mensurável — "está pronto" não é critério de aceite.

| Pré-condição | Responsável NovaTech | Critério de aceite verificável | Prazo |
|---|---|---|---|
| PROC-042-v2 declarada como vigente; v1 arquivada no SharePoint | Diretoria Comercial | E-mail ou documento assinado pelo Diretor Comercial confirmando a revogação da v1, com data; status "revogado" visível no SharePoint para qualquer usuário com acesso | Semana 1 do projeto |
| Itens 8 e 45 do FAQ corrigidos ou bloqueados | Coordenação de Atendimento + Diretoria Comercial | Versões corrigidas revisadas e aprovadas pelo Diretor Comercial por e-mail; ou confirmação de que os itens foram removidos do escopo de indexação | Semana 2 |
| Itens 22 e 32 do FAQ bloqueados da base | Compliance + Coordenação de Atendimento | Confirmação por e-mail do Compliance de que os itens não devem ser indexados até publicação de normativos correspondentes | Semana 2 |
| Responsáveis de base designados por área | Diretoria (Operações, Compliance, Comercial, Atendimento) | Lista com nome, cargo e e-mail dos responsáveis, aprovada pela Diretoria, enviada ao time de produto | Semana 1 |
| Resolução da contradição de limiar de escalada (R$50k vs R$100k) | Diretoria Comercial + Operações | Decisão registrada em documento formal (adendo ao SLA-2024 ou nova PROC) ou comunicado interno assinado | Semana 3 |
| PROC-043 publicada OU decisão de manter tema fora do escopo da fase 1 | Compliance | E-mail do Compliance confirmando uma das duas opções, com data | Semana 3 |
| PROC-088 localizada OU decisão de manter tema fora do escopo da fase 1 | Diretoria de Operações | Idem acima | Semana 3 |
| Canal de notificação de novos documentos operacional | Responsáveis de base + time de produto | Teste de notificação realizado com sucesso: responsável publica documento de teste, time de produto recebe alerta, documento é indexado dentro do SLA | Semana 4 |
| Atendentes informados sobre escopo e limitações do assistente | Coordenação de Atendimento | Sessão de onboarding realizada; atendentes sabem quais temas o assistente cobre e quais não cobre na fase 1 | Antes do go-live |

---

## 8. Temas Fora do Escopo da Fase 1

### 8.1 Lista de temas não cobertos

Os seguintes temas não estarão cobertos pelo assistente na fase 1 e o sistema deve declará-los explicitamente como fora de escopo quando consultados.

| Tema | Motivo | Próximo passo sugerido ao atendente |
|---|---|---|
| Frete para cargas perigosas (cálculo e tabela) | PROC-043 pendente de publicação pelo Compliance | Consultar área de Compliance |
| Seguro de carga (percentuais e condições) | Sem normativo formal publicado | Consultar área Comercial |
| Carga danificada em trânsito (processo de sinistro) | Sem normativo formal; FAQ não validado | Consultar supervisor; e-mail sinistros@novatech.com.br (não validado formalmente — confirmar com supervisor) |
| Frete padrão para cargas abaixo de 500 kg | Sem documento na base | Consultar área Comercial |
| Interceptação de carga em trânsito | PROC-088 não localizada | Consultar Diretoria de Operações |
| Critérios de exceção para devolução de carga perigosa | Processo não formalizado; exceções são discricionárias da Gestão de Riscos | Ligar para o ramal 4500 — Gestão de Riscos (referência da POL-001) |
| Contratos individuais de clientes (termos específicos) | Dados contratuais específicos não são indexados na base | Consultar o Comercial ou a conta do cliente no sistema CRM |
| Fretes internacionais ou exportação | Fora do escopo operacional da base documental disponível | Consultar área responsável por operações internacionais |

### 8.2 Como o assistente comunica fora de escopo

Quando um tema fora do escopo for consultado, o assistente deve:

1. Reconhecer explicitamente que o tema é do interesse do atendente.
2. Informar que esse tema não está coberto pela base documental disponível no assistente neste momento.
3. Indicar o próximo passo sugerido (conforme coluna acima), com a ressalva de que contatos informais (como o e-mail sinistros@) não estão formalizados em normativo e devem ser confirmados com o supervisor.
4. Oferecer ao atendente a opção de registrar o tema como gap para que a equipe de produto avalie a inclusão em fases futuras.

### 8.3 Processo de expansão de escopo por fase

Temas fora do escopo da fase 1 podem ser incluídos em fases subsequentes mediante as seguintes condições:

- Publicação do normativo formal correspondente (para temas como seguro de carga, PROC-043, PROC-088).
- Validação do conteúdo do FAQ pelo responsável formal da área (para temas como carga danificada em trânsito).
- Decisão de produto documentada, com aprovação do gestor de produto e do responsável de base da área correspondente.

O painel de gaps recorrentes (Seção 6.3) é o insumo principal para priorização de expansão de escopo.

---

## 9. Critérios de Qualidade de Resposta

### 9.1 O que define uma boa resposta

Uma resposta do assistente é considerada de qualidade quando atende a todos os critérios abaixo:

| Critério | Descrição | Como verificar |
|---|---|---|
| Precisão | A informação apresentada corresponde exatamente ao que está no documento citado, sem adição, omissão ou distorção | Revisão humana por amostragem; comparação com chunk original |
| Rastreabilidade completa | Todos os campos da Seção 6.1 estão presentes e corretos | Verificação automática por checklist no pipeline |
| Ausência de inferência proibida | Nenhuma frase de estimativa em informações numéricas ou procedimentais | Revisão de amostra por supervisor de atendimento |
| Declaração de limitação quando aplicável | Respostas parciais ou ausentes usam os formatos da Seção 4.2 | Revisão por amostragem |
| Hierarquia respeitada | Quando há conflito, a versão vencedora é a de maior hierarquia | Verificação por auditoria de casos de conflito |
| Concisão | A resposta responde diretamente à pergunta sem repetir o contexto da base nem adicionar informação não solicitada | Avaliação qualitativa por supervisor |

### 9.2 Métricas de sucesso do produto

As métricas abaixo devem ser acompanhadas mensalmente pelo gestor de produto e apresentadas à NovaTech em reunião de revisão.

| Métrica | Meta fase 1 | Forma de medição |
|---|---|---|
| Tempo médio de busca por chamado | Reduzir de 12 para menos de 2 minutos | Cronometragem amostral por supervisor de atendimento |
| Taxa de chamados que exigem consulta a documentação sem resposta do assistente | Abaixo de 10% dos chamados que consultam o assistente | Log de consultas sem resposta / total de consultas |
| Taxa de feedbacks negativos (⚠ ou ❌) sobre total de consultas | Abaixo de 5% | Log de feedbacks |
| Taxa de resolução de feedbacks dentro do SLA | Acima de 90% | Log de feedbacks com timestamp de abertura e fechamento |
| Taxa de escalonamento para supervisor por falta de resposta | Reduzir de 15% para abaixo de 8% dos chamados | Comparação com baseline pré-produto |
| Satisfação do atendente com o assistente | NPS interno ≥ 30 | Pesquisa mensal com time de atendimento |

### 9.3 Processo de avaliação contínua

Mensalmente, o gestor de produto deve revisar uma amostra aleatória de 50 consultas com resposta completa (alta confiança) para verificar precisão. Adicionalmente, todos os casos em que um feedback negativo foi registrado devem ser revisados integralmente. Os resultados dessa revisão alimentam o backlog de melhorias do produto e o ciclo de atualização dos responsáveis de base.

---

## 10. Acessibilidade e Idioma

O assistente deve:

- Operar exclusivamente em português brasileiro tanto para receber perguntas quanto para formatar respostas.
- Responder a perguntas com erros ortográficos, gírias de atendimento ou siglas internas sem solicitar reformulação (desde que a intenção seja clara). Exemplos de siglas internas reconhecidas: CT-e, PROC, POL, SLA, FAQ, CDs (centros de distribuição).
- Não ter requisito de formatação especial da pergunta: perguntas podem ser feitas em linguagem coloquial, fragmentada ou como cópia do que o cliente disse.
- Quando a pergunta estiver em outro idioma (espanhol, inglês), o assistente informa que opera exclusivamente em português e solicita reformulação, sem tentar responder em outro idioma.
- Respostas devem ser legíveis na interface do Microsoft Teams sem necessidade de rolar horizontalmente: tabelas de mais de 4 colunas devem ser apresentadas em formato de lista estruturada em vez de tabela.

---

*Documento elaborado com base na Especificação v1.0 e no conjunto completo de análises do projeto. Incorpora correções de gaps e ambiguidades identificados na versão anterior. Todos os requisitos devem ser validados com as áreas de Operações, Comercial, Compliance e TI da NovaTech antes do início do desenvolvimento.*
