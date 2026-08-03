# Harness de Produto — NovaTech Assistant
## Métricas de Qualidade · Tarefa 1

**Versão:** 1.0  
**Data:** 05/06/2026  
**Elaborado por:** Product Specialist Sênior  
**Destinatários:** Product · Tech Lead · QA · Engenharia · Delivery Manager · Operações · Compliance · Comercial

> Métricas marcadas com **(H)** são hipóteses de produto a serem validadas e calibradas após o go-live com base nos dados reais do ambiente de produção da NovaTech. As metas iniciais foram definidas com base no baseline do período de discovery e nos benchmarks de sistemas RAG corporativos comparáveis.

---

## Contexto de medição

O assistente processa uma média de 320 chamados/dia, dos quais ~60% envolvem consulta documental (~192 consultas/dia). As métricas são calculadas sobre o universo de consultas documentais, não sobre o total de chamados, salvo indicação contrária.

O sistema de medição usa dois instrumentos: (1) metadados estruturados do pipeline — campos do `AssistantResponse` como `source_document`, `confidence_level`, `has_conflict`, `is_absence_declaration` — coletados automaticamente; e (2) feedback explícito dos atendentes via botões ✅/⚠️/❌ na interface do Teams.

---

## Métricas de Qualidade

---

### M-01 — Taxa de respostas com citação válida

| Campo | Conteúdo |
|---|---|
| **Definição** | Percentual de respostas com conteúdo factual que contêm `source_document` com `doc_name`, `doc_version` e `section` preenchidos e não nulos. Exclui declarações de ausência (`is_absence_declaration: true`), que têm comportamento de citação diferente. |
| **Como medir** | Contagem automática no pós-processamento: `respostas_com_source_document_valido / total_respostas_factuais × 100`. Um `source_document` é válido quando `doc_name ≠ null AND doc_version ≠ null AND section ≠ null`. O validador de schema bloqueia respostas sem fonte — essa métrica deve se aproximar de 100% se P-07 estiver implementado. |
| **Frequência** | Diária (dashboard automático) · Semanal (revisão editorial) |
| **Meta** | 100% — não há tolerância para resposta factual sem fonte |
| **Alerta** | < 99% em qualquer período de 24h |
| **Responsável** | Tech Lead (alerta técnico) · Product Specialist (revisão editorial) |
| **Ação corretiva** | Verificar se o validador de schema (P-07) está ativo; inspecionar logs para identificar chamadas que contornaram o validador; corrigir no pipeline antes do próximo deploy |

---

### M-02 — Groundedness (aderência à fonte)

| Campo | Conteúdo |
|---|---|
| **Definição** | Percentual de afirmações factuais na resposta que podem ser rastreadas literalmente ao(s) chunk(s) citado(s) no `source_document`. Mede se o assistente está reproduzindo fielmente o que está na documentação ou interpolando/extrapolando. |
| **Como medir** | Avaliação por amostragem semanal: seleção aleatória de 30 respostas factuais; revisor humano (QA ou Product Specialist) verifica cada afirmação da resposta contra o texto do chunk citado e classifica como: `aderente` (afirmação presente literalmente ou por paráfrase próxima), `extrapolada` (afirmação derivada mas não literal) ou `sem sustentação` (afirmação não encontrada no chunk). Groundedness = `afirmações_aderentes / total_afirmações × 100`. Registrar em planilha de avaliação manual. **(H)** |
| **Frequência** | Semanal (amostragem de 30 respostas) · Mensal (análise de tendência) |
| **Meta** | ≥ 95% das afirmações aderentes **(H)** |
| **Alerta** | < 90% em qualquer semana de amostragem |
| **Responsável** | QA (execução da amostragem) · Product Specialist (análise de padrões) |
| **Ação corretiva** | Identificar os chunks ou temas com maior taxa de extrapolação; revisar instrução do system prompt de não inferência; se padrão for recorrente em tema específico, adicionar instrução de domínio ou revisar chunking do documento |

---

### M-03 — Taxa de alucinação

| Campo | Conteúdo |
|---|---|
| **Definição** | Percentual de respostas factuais que contêm afirmações não presentes em nenhum chunk recuperado e não marcadas como ausência ou baixa confiança. Alucinação é distinta de incompletude: é a presença de informação fabricada, não a ausência de informação correta. Inclui: valores numéricos inventados, critérios inexistentes ("negligência"), políticas não documentadas, tiers não existentes. |
| **Como medir** | Combinação de dois instrumentos: (1) detecção automática — verificar se a resposta contém strings proibidas em contexto factual (lista de critérios inexistentes, tiers inválidos, termos de estimativa como "provavelmente"); (2) amostragem manual quinzenal de 20 respostas com classificação de alucinação por revisor. Taxa automática = `respostas_com_string_proibida / total_respostas_factuais × 100`. Taxa manual = `respostas_com_alucinacao_confirmada / amostra × 100`. |
| **Frequência** | Diária (detecção automática) · Quinzenal (amostragem manual) |
| **Meta** | 0% detectado automaticamente · < 2% na amostragem manual **(H)** |
| **Alerta** | Qualquer detecção automática (zero tolerance) · > 1% na amostragem manual |
| **Responsável** | Engenharia (detecção automática) · QA (amostragem manual) |
| **Ação corretiva** | Alerta automático: revisar imediatamente o prompt e os filtros do pré-processamento; identificar se o tema está na lista de temas bloqueados (P-06) e adicionar se não estiver. Alerta manual: identificar padrão dos temas afetados; avaliar se o problema é de chunking, de prompt ou de ausência de documento na base |

---

### M-04 — Precisão da recuperação documental

| Campo | Conteúdo |
|---|---|
| **Definição** | Percentual de consultas em que o chunk recuperado como vencedor pertence ao documento e seção esperados para o tema da query. Mede se o pipeline de retrieval está buscando no documento correto, não apenas retornando chunks semanticamente próximos mas de contexto errado. |
| **Como medir** | Criar conjunto de testes dourados (`golden set`) com 50 pares query → documento esperado cobrindo os temas mais frequentes (prazos 35%, frete 25%, devolução 20%, SLA 20%). Para cada query do conjunto, executar o retrieval e verificar se `context_package.chunks[0].doc_name` e `context_package.chunks[0].section` correspondem ao esperado. Precisão = `chunks_corretos / total_queries_golden × 100`. Executar o golden set a cada alteração no pipeline de retrieval. |
| **Frequência** | A cada alteração no pipeline (threshold, chunking, embedding model) · Semanal como baseline |
| **Meta** | ≥ 90% no golden set **(H)** |
| **Alerta** | < 85% após qualquer alteração no pipeline |
| **Responsável** | QA (manutenção do golden set) · Tech Lead (análise de falhas) |
| **Ação corretiva** | Identificar as queries que falharam; verificar se o problema é de chunking (documento cortado na seção errada), de embedding (similaridade semântica baixa para o tema), ou de metadados (seção não indexada corretamente); corrigir no pipeline de ingestão ou ajustar o threshold |

---

### M-05 — Taxa de respostas com baixa confiança

| Campo | Conteúdo |
|---|---|
| **Definição** | Percentual de respostas com `confidence_level: "Baixa"` sobre o total de respostas entregues ao atendente (incluindo declarações de ausência). Alta taxa de baixa confiança pode indicar: base documental com muitos gaps, excesso de FAQ na cobertura, threshold de retrieval mal calibrado ou pipeline de governança documental com muitos documentos em revisão. |
| **Como medir** | Contagem automática nos logs: `respostas_confidence_baixa / total_respostas × 100`. Segmentar por causa: `confidence_baixa_por_faq`, `confidence_baixa_por_doc_em_revisao`, `confidence_baixa_por_ausencia`. |
| **Frequência** | Diária · Semanal (segmentação por causa) |
| **Meta** | < 15% do total de respostas **(H)** |
| **Alerta** | > 20% em qualquer período de 7 dias |
| **Responsável** | Product Specialist (análise de gaps) · Operações (cobertura documental) |
| **Ação corretiva** | Se aumento for por FAQ: verificar se novos temas precisam de normativo formal e acionar área responsável. Se por doc em revisão: cobrar Compliance pela publicação. Se por ausência: adicionar ao painel de gaps para priorização de cobertura |

---

### M-06 — Taxa de escalação ao supervisor

| Campo | Conteúdo |
|---|---|
| **Definição** | Percentual de consultas documentais em que o atendente aciona o supervisor após receber a resposta do assistente. Serve como proxy de insatisfação com a resposta ou de incerteza do atendente. Baseline pré-produto: 15% dos chamados com consulta documental escalavam sem resposta. Meta é reduzir esse índice. |
| **Como medir** | Requer integração com o sistema de chamados (Azure DevOps): rastrear chamados que receberam consulta ao assistente e, dentro de 30 minutos, tiveram escalação registrada. Taxa = `chamados_com_consulta_e_escalação / chamados_com_consulta × 100`. Alternativamente, usar o botão ❌ como proxy de escalação até que a integração esteja disponível. |
| **Frequência** | Semanal · Mensal (tendência) |
| **Meta** | < 8% (baseline: 15% pré-produto) **(H)** |
| **Alerta** | > 12% em qualquer semana |
| **Responsável** | Delivery Manager (acompanhamento de tendência) · Product Specialist (investigação de causas) |
| **Ação corretiva** | Identificar os temas que geraram escalação; se concentrado em temas específicos, revisar cobertura documental e qualidade dos chunks; se distribuído, revisar calibração de confiança |

---

### M-07 — Taxa de feedback positivo e negativo

| Campo | Conteúdo |
|---|---|
| **Definição** | Proporção de feedbacks ✅ (útil), ⚠️ (incompleta) e ❌ (incorreta) sobre o total de respostas que receberam feedback explícito. Mede satisfação do atendente com a qualidade das respostas. |
| **Como medir** | Coleta automática via botões de feedback na interface do Teams. Calcular: `taxa_positivo = respostas_✅ / total_feedback × 100`; `taxa_incompleta = respostas_⚠️ / total_feedback × 100`; `taxa_incorreta = respostas_❌ / total_feedback × 100`. Acompanhar também a taxa de cobertura: `respostas_com_feedback / total_respostas × 100` — feedback só em 30% das respostas é dado insuficiente. |
| **Frequência** | Diária · Semanal (segmentação por tema e tier de cliente) |
| **Meta** | ✅ ≥ 75% · ⚠️ ≤ 20% · ❌ ≤ 5% · Cobertura de feedback ≥ 40% **(H)** |
| **Alerta** | ❌ > 8% em qualquer período de 7 dias · ✅ < 65% em qualquer período de 7 dias |
| **Responsável** | Product Specialist (análise diária) · QA (investigação de ❌) |
| **Ação corretiva** | Aumento de ❌: abrir revisão manual de todas as respostas marcadas como incorretas naquele período; se padrão identificado, criar caso de regressão e priorizar no backlog. Aumento de ⚠️ concentrado em tema: revisar completude das instruções de prompt para aquele domínio |

---

### M-08 — Tempo de resposta

| Campo | Conteúdo |
|---|---|
| **Definição** | Dois SLAs distintos: (1) TTFT — Time to First Token: tempo entre envio da query e chegada do primeiro token ao cliente via streaming; (2) Tempo de resposta completa: tempo entre envio da query e recebimento do último token. Ambos medidos do lado do cliente (Teams ou painel web) no percentil 95. |
| **Como medir** | Instrumentação no cliente: registrar timestamp de envio da query, timestamp do primeiro token recebido (TTFT) e timestamp do último token. Calcular percentil 95 de cada métrica sobre o conjunto de consultas do dia. Segmentar por: tipo de resposta (factual simples, multi-partes, declaração de ausência), horário do dia e carga de uso. |
| **Frequência** | Contínua (coleta) · Diária (p95 calculado) · Semanal (análise de tendência e correlação com carga) |
| **Meta** | TTFT p95 < 3 segundos · Resposta completa p95 < 30 segundos |
| **Alerta** | TTFT p95 > 4 segundos · Resposta completa p95 > 25 segundos por 2 dias consecutivos |
| **Responsável** | Tech Lead (infraestrutura) · Engenharia (otimização) |
| **Ação corretiva** | Investigar se o problema é de latência do Azure OpenAI, do Azure AI Search ou da rede interna. Se TTFT degradado: verificar se context budget aumentou além do definido no ADR-0002. Se resposta completa degradada: verificar se há queries multi-partes consumindo mais tokens |

---

### M-09 — Taxa de violação de guardrails

| Campo | Conteúdo |
|---|---|
| **Definição** | Percentual de respostas que violam ao menos um dos guardrails formalizados. Detecção automática cobre os guardrails implementados em código. Detecção manual cobre os guardrails de prompt. Guardrails monitorados automaticamente: citação sem fonte + confiança Alta (P-07), tier inválido na resposta, string de estimativa em contexto factual (P-08 proxy), chunk de documento arquivado no contexto. |
| **Como medir** | Detecção automática: log de rejeições do validador de schema (P-07) + log de detecções de string proibida + log de chunks arquivados no contexto. Taxa automática = `respostas_com_violacao_detectada / total_respostas × 100`. Detecção manual: subconjunto da amostragem de groundedness (M-02) com checklist de guardrails por resposta. |
| **Frequência** | Contínua (detecção automática) · Quinzenal (amostragem manual) |
| **Meta** | 0% para guardrails implementados em código · < 1% na amostragem manual **(H)** |
| **Alerta** | Qualquer violação automática detectada (zero tolerance) · > 0,5% na amostragem manual |
| **Responsável** | Engenharia (detecção automática) · QA + Compliance (amostragem manual) |
| **Ação corretiva** | Violação automática: parar deploy em andamento se ocorrer durante janela de release; investigar contorno do validador; corrigir e revalidar antes de reabrir produção. Violação manual: criar caso de teste de regressão; priorizar ajuste de prompt ou pipeline |

---

### M-10 — Taxa de conflitos documentais corretamente sinalizados

| Campo | Conteúdo |
|---|---|
| **Definição** | Percentual de respostas em que `has_conflict: true` estava presente no `context_package` E o campo `conflict_notice` foi incluído corretamente na resposta entregue ao atendente. Mede se o pipeline está sinalizando os conflitos que detecta, em vez de silenciá-los. |
| **Como medir** | Contagem automática: de todas as respostas em que `context_package.has_conflict == true`, verificar quantas contêm `conflict_notice` não nulo e no formato canônico ("Existe outro documento na base..."). Taxa = `respostas_com_conflict_notice / respostas_com_has_conflict × 100`. Complementar com amostragem manual para detectar conflitos que o pipeline não detectou mas que deveriam ter sido detectados. |
| **Frequência** | Diária (automático) · Mensal (amostragem manual de cobertura) |
| **Meta** | 100% de sinalização quando `has_conflict: true` · ≥ 80% de cobertura de detecção (pipeline detecta os conflitos reais) **(H)** |
| **Alerta** | < 100% de sinalização quando `has_conflict: true` · < 70% de cobertura detectada na amostragem |
| **Responsável** | Tech Lead (cobertura de detecção) · Compliance (impacto dos não detectados) |
| **Ação corretiva** | Sinalização < 100%: verificar validador de pós-processamento do `conflict_notice`; corrigir imediatamente. Cobertura < 70%: revisar lógica de comparação de metadados em BC-02 que detecta `has_conflict`; aumentar o escopo de comparação |

---

### M-11 — Taxa de ausência falsa (falso negativo de retrieval)

| Campo | Conteúdo |
|---|---|
| **Definição** | Percentual de declarações de ausência (`is_absence_declaration: true`) emitidas para temas que tinham cobertura documental disponível na base. Este é o erro do Incidente 3: o assistente diz que não encontrou informação quando o documento estava indexado. Distinto da ausência real (tema genuinamente não coberto). |
| **Como medir** | Amostragem semanal de 15 respostas com `is_absence_declaration: true`: revisor verifica manualmente no índice do Azure AI Search se existe chunk relevante para o tema da query com score que deveria estar acima do threshold. Taxa = `ausencias_com_documento_disponivel / amostra_ausencias × 100`. Complementar com: quando atendente clica em "Registrar como lacuna", verificar automaticamente se o tema já tem cobertura no índice. |
| **Frequência** | Semanal (amostragem manual) · Diária (verificação automática de lacunas registradas) |
| **Meta** | < 3% de ausências falsas na amostragem **(H)** |
| **Alerta** | > 5% em qualquer semana de amostragem |
| **Responsável** | QA (amostragem) · Tech Lead (diagnóstico de retrieval) |
| **Ação corretiva** | Identificar se o problema é de threshold muito alto (chunk estava abaixo de 0,72 por problema de embedding), de chunking (informação dividida entre dois chunks), ou de metadados (seção não indexada). Ajustar no pipeline de ingestão ou recalibrar threshold com dados de produção |

---

### M-12 — Taxa de aceitação da resposta pelo atendente

| Campo | Conteúdo |
|---|---|
| **Definição** | Percentual de consultas ao assistente em que o atendente usou a resposta diretamente no atendimento sem ação adicional (sem abrir outra fonte, sem escalar para supervisor, sem registrar feedback negativo). Proxy do valor real entregue ao atendente por chamado. |
| **Como medir** | Proxy calculado: `(total_consultas - consultas_com_❌ - consultas_com_escalacao - consultas_com_fonte_externa_aberta) / total_consultas × 100`. A abertura de fonte externa requer integração com o navegador do atendente ou dados de acesso ao SharePoint/Confluence após a consulta ao assistente (disponível via Microsoft 365 audit log se provisionado). Enquanto a integração não estiver disponível, usar: `(total_consultas - consultas_com_❌ - consultas_com_escalacao) / total_consultas × 100` como métrica intermediária. Baseline pre-produto: atendente abre 4 fontes por chamado em média. |
| **Frequência** | Semanal · Mensal (correlação com satisfação do atendente via pesquisa NPS interna) |
| **Meta** | ≥ 70% de aceitação direta **(H)** · NPS interno ≥ 30 após 60 dias de produção **(H)** |
| **Alerta** | < 55% em qualquer semana |
| **Responsável** | Delivery Manager (acompanhamento de adoção) · Product Specialist (diagnóstico de baixa aceitação) |
| **Ação corretiva** | Investigar as consultas não aceitas: se concentradas em tema específico, revisar cobertura e qualidade do chunk; se distribuídas, realizar entrevista com atendentes para identificar barreira de confiança ou de interface; se por tempo de resposta, escalar para Tech Lead |

---

## Painel de Métricas — Visão Resumida

| Métrica | Definição | Como medir | Frequência | Meta | Alerta | Responsável | Ação corretiva |
|---|---|---|---|---|---|---|---|
| M-01 Taxa de citação válida | % respostas factuais com source_document completo | Contagem automática de schema | Diária | 100% | < 99% | Tech Lead · PS | Verificar validador P-07 |
| M-02 Groundedness | % afirmações aderentes ao chunk citado | Amostragem manual de 30 respostas/semana | Semanal | ≥ 95% **(H)** | < 90% | QA · PS | Revisar prompt de não inferência |
| M-03 Taxa de alucinação | % respostas com afirmação sem sustentação | Automático (strings proibidas) + amostragem manual | Diária / Quinzenal | 0% auto · < 2% manual **(H)** | Qualquer detecção / > 1% | Engenharia · QA | Revisar P-06; adicionar tema a BLOCKED_TOPICS |
| M-04 Precisão de recuperação | % queries do golden set com chunk correto | Golden set de 50 queries executado no pipeline | Por alteração / Semanal | ≥ 90% **(H)** | < 85% | QA · Tech Lead | Revisar chunking e threshold |
| M-05 Taxa de baixa confiança | % respostas com confidence_level Baixa | Contagem automática nos logs segmentada por causa | Diária | < 15% **(H)** | > 20% | PS · Operações | Acionar área responsável por gaps |
| M-06 Taxa de escalação | % consultas seguidas de escalação ao supervisor | Integração Azure DevOps · proxy ❌ interim | Semanal | < 8% (baseline 15%) **(H)** | > 12% | Delivery Manager · PS | Investigar temas que geraram escalação |
| M-07 Feedback positivo/negativo | % ✅ ⚠️ ❌ sobre total de feedbacks | Botões de feedback na interface Teams | Diária | ✅ ≥ 75% · ❌ ≤ 5% **(H)** | ❌ > 8% · ✅ < 65% | PS · QA | Revisar respostas ❌; criar casos de regressão |
| M-08 Tempo de resposta | TTFT p95 e resposta completa p95 | Instrumentação no cliente | Contínua / Diária | TTFT < 3s · total < 30s | TTFT > 4s · total > 25s por 2 dias | Tech Lead · Engenharia | Investigar latência Azure; verificar context budget |
| M-09 Violação de guardrails | % respostas violando ao menos um guardrail | Automático (código) + manual (checklist) | Contínua / Quinzenal | 0% auto · < 1% manual **(H)** | Qualquer violação / > 0,5% | Engenharia · QA · Compliance | Parar deploy; corrigir validador |
| M-10 Conflitos sinalizados | % respostas com has_conflict que incluem conflict_notice | Contagem automática de campos | Diária | 100% sinalização **(H)** | < 100% sinalização | Tech Lead · Compliance | Verificar validador de conflict_notice |
| M-11 Ausência falsa | % declarações de ausência com documento disponível | Amostragem manual de ausências + verificação automática | Semanal | < 3% **(H)** | > 5% | QA · Tech Lead | Ajustar threshold ou chunking |
| M-12 Taxa de aceitação | % consultas usadas diretamente sem ação adicional | Proxy via ausência de ❌ + escalação · audit log | Semanal | ≥ 70% **(H)** | < 55% | Delivery Manager · PS | Entrevista com atendentes; revisar cobertura ou interface |

---

## Periodicidade de Revisão das Metas

As metas marcadas como **(H)** devem ser revisadas nos seguintes marcos após o go-live:

| Marco | Ação |
|---|---|
| **Semana 2 pós-go-live** | Revisar M-05 (baixa confiança) e M-07 (feedback) com dados reais dos 5 atendentes-piloto |
| **Semana 4 pós-go-live** | Revisão completa de todas as metas **(H)** com base nos primeiros 20 dias de produção; calibrar thresholds de alerta |
| **Semana 8 pós-go-live** | Revisão com todos os 45 atendentes ativos; definir metas permanentes para o restante do trimestre |
| **Trimestral (recorrente)** | Revisão de metas com Operações, Compliance e Comercial; incorporar novos tipos de consulta e novos documentos na base de medição |

---

*Documento elaborado com base no cenário completo do projeto NovaTech Assistant, nas decisões arquiteturais (ADR-0001 a ADR-0004), nos guardrails formalizados (guardrails-novatech.md) e na revisão crítica de respostas pré-go-live (novatech-revisao-critica-final.md).*
