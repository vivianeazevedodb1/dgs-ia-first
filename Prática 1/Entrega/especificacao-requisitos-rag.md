# Especificação de Requisitos de Produto — Pipeline de RAG
## Assistente de IA para Atendimento NovaTech

**Elaborado por:** Product Specialist  
**Data:** 05/06/2026  
**Versão:** 1.0  
**Contexto:** Projeto DB1 × NovaTech — Assistente de IA integrado ao Microsoft Teams + SharePoint  
**Fontes de entrada:** Cenário operacional, mapa de temas e gaps, análise de inconsistências, cruzamento FAQ × normativos, jornada do atendente, diagrama visual de fluxo, Anexo A (documentação simulada), Anexo B (chunks de referência RAG)

> Este documento especifica os requisitos de produto do pipeline de RAG na linguagem de negócio — sem código ou arquitetura técnica. Destina-se a alinhar time de produto, stakeholders da NovaTech e equipe de desenvolvimento sobre o que o sistema deve e não deve fazer antes de qualquer decisão de implementação.

---

## 1. Fontes de Dados que Devem ser Indexadas

O pipeline deve indexar as fontes listadas abaixo. A ordem representa a hierarquia de confiabilidade que o assistente deve seguir ao apresentar respostas.

### 1.1 Documentos normativos — Nível 1 (máxima prioridade)

São os documentos com responsável formal, versão e data de atualização definidos. Constituem a fonte de verdade do assistente.

| Documento | Localização atual | Responsável | Ciclo de atualização |
|---|---|---|---|
| POL-001 — Política de Devolução de Mercadorias (v3.1) | SharePoint corporativo | Diretoria de Operações | Sob demanda (última: jan/2024) |
| PROC-042-v2 — Frete Especial Revisado (v2.0) | SharePoint corporativo | Diretoria Comercial | Sob demanda (última: nov/2023) |
| SLA-2024 — Tabela de SLA por Tipo de Cliente (v2024.1) | SharePoint corporativo | Diretoria Comercial + Operações | Anual (última: jan/2024) |

Quando a PROC-043 (Frete de Cargas Perigosas) for publicada pelo Compliance, deve ser incorporada imediatamente ao Nível 1.

### 1.2 Planilhas de referência operacional — Nível 2

São os documentos de suporte atualizados mensalmente, como a tabela de valor base de fretes (`frete-base-AAAAMM.xlsx`). Devem ser indexados com marcação de mês de referência, para que o assistente saiba qual versão está usando e informe o atendente quando a planilha for do mês anterior.

### 1.3 Wiki interna (Confluence) — Nível 3

Páginas do Confluence com caráter normativo ou procedural devem ser indexadas após triagem prévia pela área de Operações ou Compliance. Páginas de projeto, rascunhos e discussões internas não devem entrar na base.

### 1.4 FAQ-Atendimento — Nível 4 (fonte auxiliar, com restrições)

O FAQ deve ser indexado com classificação explícita de fonte informal. O assistente pode usá-lo apenas para temas não cobertos por documentos de Nível 1 ou 2, e sempre com marcação de baixa confiança visível ao atendente. Os itens do FAQ que estão em contradição comprovada com documentos normativos (itens 8 e 45, conforme análise de cruzamento) devem ser excluídos da indexação até que sejam corrigidos.

---

## 2. Documentos que Devem ser Excluídos da Indexação

Os documentos abaixo não devem entrar na base do assistente, pelos motivos indicados.

| Documento | Motivo de exclusão |
|---|---|
| PROC-042 v1.0 (versão original, mar/2023) | Substituída implicitamente pela v2.0. Coexistência na base geraria respostas contraditórias sobre multiplicadores, fatores de peso, prazo e descontos. A v1 deve ser arquivada com status "revogado" no SharePoint antes da indexação. |
| FAQ-Atendimento — itens 8 e 45 | Contradizem diretamente a PROC-042-v2: o item 8 omite o prazo de +3 dias úteis e o item 45 usa o limiar de desconto errado (>10 fretes em vez de ≥8). Devem ser corrigidos antes de serem indexados. |
| FAQ-Atendimento — item 22 (seguro de carga) | Percentuais sem respaldo normativo verificável. Risco de cotação incorreta com impacto contratual. Deve ser bloqueado até publicação de normativo formal de seguro. |
| FAQ-Atendimento — item 32 (carga perigosa com frete expresso) | Processo descrito sem respaldo em nenhum PROC ou POL vigente. PROC-043 está em revisão. Não deve ser usado como fonte até regularização normativa. |
| PROC-043 — Frete de Cargas Perigosas (versão em revisão) | Documento em revisão ativa pelo Compliance. Indexar versão em elaboração introduziria informação instável. Indexar apenas após publicação oficial. |
| PROC-088 — Interceptação de Carga | Referenciado pela POL-001, mas não localizado no conjunto documental. Não indexar até que o documento seja identificado e validado. |
| Documentos do SharePoint sem versão, responsável ou data de atualização | Documentos sem metadados mínimos não oferecem rastreabilidade. Devem passar por triagem antes de serem considerados. |
| Rascunhos, templates e documentos de projeto no Confluence | Não representam políticas vigentes. Risco de o assistente confundir intenção de mudança com regra em vigor. |

---

## 3. Como Lidar com Documentos Contraditórios

A contradição entre documentos é o principal risco operacional identificado na base documental da NovaTech. O pipeline deve tratar esse risco em três camadas.

### 3.1 Antes da indexação — resolução editorial

Para cada par de documentos identificados como contraditórios, a NovaTech deve definir formalmente qual versão é a vigente antes que o pipeline entre em produção. Esse trabalho é pré-requisito de lançamento, não opcional.

Os pares de contradição confirmados que exigem resolução prévia são:

- PROC-042 v1 × PROC-042-v2 (multiplicadores, fatores de peso, prazo, descontos)
- FAQ item 45 × PROC-042-v2 (limiar de desconto por volume)
- FAQ item 8 × PROC-042-v2 (prazo adicional de manuseio)

Resolução esperada: a Diretoria Comercial deve declarar formalmente a PROC-042-v2 como vigente, arquivar a v1 no SharePoint com status "revogado" e autorizar a exclusão dos itens divergentes do FAQ da base indexada.

### 3.2 Durante a indexação — marcação de status por documento

Cada documento indexado deve carregar um rótulo de status que o assistente usa para priorizar respostas:

| Status | Significado | Comportamento do assistente |
|---|---|---|
| Vigente | Documento com versão ativa e responsável confirmado | Usar como fonte primária |
| Revisado — use esta versão | Declarado formalmente como substituto de versão anterior | Usar como fonte primária; mencionar que há versão anterior arquivada se relevante |
| Arquivado — não usar | Versão revogada, mantida apenas para histórico | Nunca incluir em respostas |
| Em revisão — use com cautela | Documento em atualização pelo responsável formal | Usar somente se não houver outro documento sobre o tema; sinalizar instabilidade |
| Informal — baixa confiança | Documentos como FAQ sem validação normativa | Usar como último recurso; sempre sinalizar ao atendente |

### 3.3 Em tempo real — comportamento do assistente ao detectar conflito

Se, mesmo após a resolução editorial, o pipeline recuperar chunks de documentos que se contradizem para a mesma pergunta, o assistente deve:

1. Apresentar a resposta com base no documento de maior hierarquia (Nível 1 sobre Nível 4; mais recente sobre mais antigo dentro do mesmo nível).
2. Incluir obrigatoriamente um alerta visível indicando que existe outro documento com informação divergente, identificando os dois documentos pelo nome e versão.
3. Nunca combinar ou calcular médias entre valores contraditórios. Exemplo: se um documento diz multiplicador 1,0 e outro diz 1,1 para o Sudeste, o assistente apresenta apenas o valor do documento de maior hierarquia — não apresenta os dois como se fossem igualmente válidos.
4. Sugerir ao atendente que confirme com o supervisor caso a contradição seja relevante para o chamado em andamento.

---

## 4. Comportamento Quando uma Pergunta Não Tiver Resposta na Base

Esse é o comportamento mais crítico do assistente: o que ele faz quando não sabe. Inventar uma resposta é o pior resultado possível — pior do que não responder.

### 4.1 Cenários de ausência de resposta

| Cenário | Exemplo | Comportamento esperado |
|---|---|---|
| Tema não coberto por nenhum documento indexado | Frete padrão para cargas abaixo de 500 kg; seguro de carga | Declara ausência de informação na base. Indica área responsável para consulta. |
| Documento referenciado existe mas não foi indexado | PROC-088 (interceptação de carga), PROC-043 (cargas perigosas) | Informa que o procedimento existe mas não está disponível na base do assistente. Orienta a consultar diretamente a área responsável. |
| Pergunta fora do escopo da documentação interna | Questões sobre concorrentes, legislação externa, dados do cliente | Informa que o assistente responde apenas com base na documentação interna da NovaTech. |
| Documento em revisão é a única fonte disponível | PROC-043 enquanto em revisão | Informa que o único documento disponível sobre o tema está em revisão pelo Compliance. Orienta escalada para a área responsável. |
| Combinação de informações que geraria resposta incompleta | Pergunta sobre frete + seguro + prazo para carga perigosa | Responde os componentes que têm cobertura documental e declara explicitamente quais componentes não têm. |

### 4.2 Formato da resposta de ausência

O assistente deve sempre retornar uma mensagem estruturada contendo:

- Confirmação clara de que não há resposta disponível na base para aquele tema específico.
- Motivo identificável (tema não coberto / documento não indexado / documento em revisão).
- Próximo passo sugerido com área ou canal de contato, quando identificável a partir de documentos formais. O assistente não deve sugerir contatos que constam apenas no FAQ informal (ex: sinistros@novatech.com.br) sem indicar que o canal não está formalizado em normativo.
- Opção de registrar a pergunta sem resposta como feedback, para que a equipe de produto monitore lacunas recorrentes na base.

O assistente nunca deve usar linguagem que sugira que está estimando, deduzindo ou completando informação não encontrada. Frases como "provavelmente", "deve ser", "normalmente" ou "em geral" são proibidas em respostas sobre prazos, valores, condições contratuais e procedimentos operacionais.

---

## 5. Tempo para Novos Documentos Estarem Disponíveis no Assistente

A NovaTech atualiza documentação mensalmente por três áreas diferentes sem processo unificado de revisão. O pipeline precisa de um ciclo de atualização que minimize o risco de o assistente operar com informação desatualizada.

### 5.1 Tempo-alvo por tipo de atualização

| Tipo de atualização | Prazo-alvo para disponibilidade no assistente | Justificativa |
|---|---|---|
| Novo documento normativo (POL, PROC, SLA) | Até 24 horas após publicação oficial no SharePoint | Documentos normativos têm impacto imediato em atendimento. Atraso gera risco de resposta incorreta. |
| Revisão de documento existente (nova versão) | Até 24 horas após publicação, com arquivamento simultâneo da versão anterior | Garantir que a versão antiga seja removida no mesmo ciclo em que a nova entra. |
| Planilha de referência mensal (ex: tabela base de fretes) | Até o primeiro dia útil do mês de vigência | Planilhas de referência têm data de vigência mensal conhecida antecipadamente. |
| Correção de item do FAQ após ciclo de feedback | Até 48 horas após validação pela área responsável | Correções do FAQ dependem de aprovação humana antes de entrar na base. |
| Novo documento identificado como ausente (ex: PROC-088) | Até 48 horas após localização e validação do documento | Documentos referenciados mas ausentes devem entrar na base assim que localizados. |

### 5.2 Processo de atualização

Para que os prazos acima sejam viáveis, a NovaTech deve designar um responsável por área (Operações, Compliance, Comercial) que notifique o time de produto a cada publicação de documento novo ou revisado. Sem esse processo, o pipeline não tem como saber que há conteúdo para reindexar.

O assistente deve exibir, em cada resposta, a data da última atualização do documento fonte utilizado. Se um documento estiver com data de atualização superior a 6 meses, o assistente inclui um aviso de que a informação pode estar desatualizada e sugere confirmação com a área responsável.

---

## 6. Requisitos de Rastreabilidade

Rastreabilidade é o requisito que diferencia o assistente de uma resposta de memória: o atendente precisa saber de onde veio cada informação para confiar nela, contestá-la ou escalar quando necessário.

### 6.1 O que toda resposta deve conter

Cada resposta do assistente deve incluir obrigatoriamente os seguintes elementos de rastreabilidade:

| Campo | Descrição | Exemplo |
|---|---|---|
| Nome do documento | Identificador formal do documento | POL-001 — Política de Devolução de Mercadorias |
| Versão | Número de versão do documento | v3.1 |
| Seção de referência | Seção ou item específico do documento | Seção 3.1 |
| Data da última atualização | Data registrada nos metadados do documento | 15/01/2024 |
| Nível de confiança | Qualitativo: Alta / Moderada / Baixa | Alta confiança |
| Classificação do documento | Normativo / Contratual / Procedural / Informal | Documento normativo |
| Alerta de conflito | Indicação se há documento divergente na base | ⚠ Existe versão anterior deste documento com valores diferentes |

### 6.2 Rastreabilidade de perguntas sem resposta

Perguntas que não obtiveram resposta também devem ser registradas com rastreabilidade:

- Texto da pergunta (anonimizado se necessário).
- Motivo de ausência de resposta.
- Tema inferido da pergunta (para agrupamento posterior).
- Timestamp e identificador do chamado (se disponível via integração com o sistema de chamados).

Esses registros alimentam um painel de gaps recorrentes, que deve ser revisado mensalmente pela gestão de produto e pelas áreas responsáveis pela documentação.

### 6.3 Rastreabilidade do ciclo de feedback

Cada feedback registrado pelo atendente (⚠ incorreto / ❌ errado) deve gerar um registro rastreável contendo:

- Identificador único do feedback.
- Resposta original do assistente (texto e fontes citadas).
- Tipo de problema sinalizado pelo atendente.
- Descrição livre fornecida pelo atendente.
- Data e hora do registro.
- Área para qual foi roteado.
- Status de resolução (aberto / em análise / resolvido / descartado).
- Data de encerramento e descrição da ação tomada.

O atendente que originou o feedback deve receber notificação quando o ciclo for encerrado. Feedbacks não resolvidos em 15 dias úteis geram alerta automático para o gestor responsável pela base documental.

### 6.4 Auditoria mínima

O sistema deve manter log auditável das seguintes ações:

- Indexação ou remoção de documento (quem solicitou, quando, qual documento, qual versão).
- Alteração de status de documento (de vigente para arquivado, por exemplo).
- Perguntas realizadas e respostas geradas (com fontes utilizadas), retidas por no mínimo 90 dias para fins de auditoria de qualidade.

Esses logs não precisam ser visíveis ao atendente, mas devem estar acessíveis ao time de produto e ao gestor de qualidade documental da NovaTech.

---

## 7. Pré-condições para Lançamento

Os requisitos acima pressupõem que as seguintes ações sejam concluídas pela NovaTech antes do go-live do assistente. São pré-condições de produto, não de tecnologia.

| Pré-condição | Responsável NovaTech | Prazo sugerido |
|---|---|---|
| Declarar formalmente PROC-042-v2 como versão vigente e arquivar v1 no SharePoint | Diretoria Comercial | Antes do início da indexação |
| Corrigir itens 8 e 45 do FAQ com parâmetros da PROC-042-v2 | Coordenação de Atendimento | Antes do início da indexação |
| Bloquear itens 22 e 32 do FAQ da base ou validá-los com Compliance | Compliance + Coordenação de Atendimento | Antes do go-live |
| Publicar PROC-043 revisada ou emitir orientação provisória para cargas perigosas | Compliance | Até o go-live; se não concluída, o tema fica fora do escopo do assistente na fase 1 |
| Designar responsável de base por área (Operações, Compliance, Comercial) | Diretoria | Antes do go-live |
| Localizar e validar PROC-088 (Interceptação de Carga) para indexação | Diretoria de Operações | Pode ser fase 2, desde que o assistente sinalize corretamente a ausência |

---

## 8. Temas Fora do Escopo da Fase 1

Com base nos gaps identificados e nas pré-condições acima, os seguintes temas não estarão cobertos pelo assistente na fase 1 e o sistema deve declará-los explicitamente como fora de escopo quando perguntados:

- Frete para cargas perigosas (PROC-043 pendente)
- Seguro de carga (sem normativo formal)
- Carga danificada em trânsito (sem normativo formal; FAQ não validado)
- Frete padrão para cargas abaixo de 500 kg (sem documento na base)
- Interceptação de carga em trânsito (PROC-088 não localizada)
- Critérios de exceção para devolução de carga perigosa via Gestão de Riscos (processo não formalizado)

Esses temas devem ser documentados no painel de gaps e priorizados para cobertura nas fases subsequentes do produto.

---

*Documento elaborado com base no conjunto completo de análises do projeto (cenário, mapa de temas e gaps, análise de inconsistências, cruzamento FAQ × normativos, jornada do atendente, diagrama de fluxo, Anexo A e Anexo B). Todos os requisitos devem ser validados com as áreas de Operações, Comercial, Compliance e TI da NovaTech antes do início do desenvolvimento.*
