# Registro de Sessão — Projeto NovaTech × DB1
## Análise Documental e Especificação do Assistente de IA para Atendimento

**Data da sessão:** 05/06/2026  
**Papel:** Product Specialist  
**Projeto:** Assistente de IA para o time de atendimento da NovaTech (45 atendentes), integrado ao Microsoft Teams + SharePoint, com objetivo de reduzir o tempo médio de busca em documentação de 12 para menos de 2 minutos por chamado.

---

## Sumário da Sessão

| # | Entrega | Documentos de entrada | Arquivo gerado |
|---|---|---|---|
| 1 | Mapa de temas cobertos e hipóteses de gaps | FAQ, POL-001, PROC-042 v1, PROC-042-v2, SLA-2024 | `novatech-analise-documental.md` |
| 2 | Análise de inconsistências PROC-042 v1 × v2 | PROC-042 v1, PROC-042-v2 | `novatech-inconsistencias-proc042-v1-v2.md` |
| 3 | Análise de inconsistências FAQ × PROC-042-v2 | FAQ-Atendimento, PROC-042-v2 | `novatech-inconsistencias-faq-proc042v2.md` |
| 4 | Cruzamento de inconsistências × práticas informais do FAQ | FAQ, mapa de gaps, análises de inconsistências | `novatech-cruzamento-faq-inconsistencias-gaps.md` |
| 5 | Jornada do atendente (fluxos principal, fallback, feedback + guardrails) | Cenário, cruzamento, análises, mapa de gaps | `novatech-jornada-atendente-assistente-ia.md` |
| 6 | Diagrama visual de fluxo (SVG inline) | Jornada do atendente | Renderizado no chat |
| 7 | Especificação de requisitos do pipeline de RAG — v1.0 | Todos os documentos anteriores + Anexo A + Anexo B | `novatech-especificacao-requisitos-rag.md` |
| 8 | Especificação de requisitos do pipeline de RAG — v2.0 (revisada) | v1.0 + identificação de gaps e ambiguidades | `novatech-especificacao-requisitos-rag-v2.md` |

---

## Contexto do Projeto

A NovaTech é uma empresa de médio porte do setor de logística com 1.200 funcionários. A documentação interna está distribuída em três fontes: SharePoint (~800 documentos), Confluence (~400 páginas) e pasta de rede com planilhas mensais. O time de atendimento (45 pessoas) gasta em média 12 minutos por chamado buscando informações, abre em média 4 fontes por chamado, e em 15% dos casos não encontra resposta e escala para o supervisor. A NovaTech contratou a DB1 para construir um assistente de IA com respostas fundamentadas na documentação oficial, indicação de fonte, e integração ao ambiente Microsoft.

---

## Entrega 1 — Mapa de Temas Cobertos e Hipóteses de Gaps

**Objetivo:** Inventariar os 5 documentos disponíveis com título, metadados e resumo; gerar mapa de temas cobertos e hipóteses de gaps.

**Método:** Análise de metadados (versão, responsável, classificação, escopo) e conteúdo de cada documento.

**Principais achados:**
- 5 documentos inventariados: FAQ-Atendimento (informal), POL-001 v3.1 (normativo), PROC-042 v1.0 (procedimento sem status de vigência), PROC-042-v2.0 (revisão sem status formal), SLA-2024 v2024.1 (contratual).
- 15 temas mapeados: 4 cobertos (✅), 7 parciais ou conflitantes (⚠️), 2 não cobertos (❌).
- 7 gaps identificados: documentos ausentes (PROC-043, PROC-088), coexistência de versões conflitantes, ausência de normativo para carga danificada e seguro de carga, FAQ sem processo de validação, SLA de devolução não mapeado, autonomia do atendente não documentada.
- 8 recomendações priorizadas (3 alta, 3 média, 2 baixa).

---

## Entrega 2 — Análise de Inconsistências PROC-042 v1 × v2

**Objetivo:** Comparar as duas versões da PROC-042 e identificar todas as divergências.

**Principais achados:**
- **Problema estrutural:** nenhuma das duas versões indica formalmente que substitui ou é substituída pela outra. Coexistem no SharePoint sem hierarquia.
- **5 inconsistências diretas:**
  - IC-01 🔴 Multiplicadores regionais: v2 é maior em todas as 5 regiões (variação de +7,1% a +12,5%).
  - IC-02 🔴 Fatores de peso: v2 reduz fatores nas faixas acima de 1.000 kg (1,2→1,15; 1,5→1,4), criando assimetria com os multiplicadores.
  - IC-03 🟡 Prazo de manuseio: v1 = +2 dias úteis; v2 = +3 dias úteis.
  - IC-04 🔴 Desconto por volume: v1 tem negociação manual (>10 fretes); v2 tem desconto automático (≥8 fretes: 5%; >15 fretes: 10%).
  - IC-05 🟠 PROC-043: v1 trata como estável; v2 alerta que está em revisão.
- **2 divergências estruturais:** disposição transitória só na v2 (expirada desde dez/2023); justificativa de revisão ausente na v1.
- **Efeito financeiro simulado:** v2 gera fretes 5,4% a 7,8% mais altos nas combinações mais comuns.

---

## Entrega 3 — Análise de Inconsistências FAQ × PROC-042-v2

**Objetivo:** Cruzar o FAQ-Atendimento com a PROC-042-v2 para identificar onde o documento informal contradiz ou omite o normativo revisado.

**Principais achados:**
- **4 inconsistências diretas:**
  - IC-01 🔴 Limiar de desconto: FAQ usa >10 fretes; v2 usa ≥8.
  - IC-02 🔴 Percentuais de desconto: FAQ não informa; v2 define 5% e 10%.
  - IC-03 🟡 Prazo de manuseio: FAQ não menciona +3 dias da v2.
  - IC-04 🟡 Cargas >5.000 kg: FAQ omite a aprovação obrigatória do gerente regional.
- **3 zonas de ambiguidade:** disposição transitória ignorada pelo FAQ; critério de contrato "na tabela antiga" sem processo formal; PROC-043 instável em ambos.

---

## Entrega 4 — Cruzamento Inconsistências × Práticas Informais do FAQ

**Objetivo:** Consolidar em visão única quais inconsistências já estão sendo operadas incorretamente no dia a dia, quais gaps estão sendo cobertos pelo FAQ de forma arriscada, e onde há dupla exposição.

**Modelo de análise:** 4 quadrantes — 🔴 Dupla Exposição, 🟠 Gap operacionalizado pelo FAQ, 🟡 Inconsistência sem reflexo no FAQ, 🟢 Alinhamento.

**11 temas cruzados:**
- 🔴 Dupla Exposição (2): desconto por volume (CRZ-01) e prazo de entrega (CRZ-03).
- 🟠 Gap operacionalizado pelo FAQ (5): cargas perigosas (CRZ-05), devolução de carga perigosa (CRZ-06), carga danificada (CRZ-07), seguro de carga (CRZ-08), rastreamento (CRZ-11).
- 🟡 Inconsistência sem reflexo (2): multiplicadores regionais (CRZ-02), aprovação >5.000 kg (CRZ-04).
- 🟢 Alinhamento (2): tier Platinum (CRZ-09), SLAs de resposta/resolução (CRZ-10).

**3 padrões identificados:**
1. FAQ como "normativo de fato" em 5 áreas sem cobertura formal.
2. Hibridização de versões: FAQ construiu regra híbrida v1+v2 incorreta para desconto.
3. FAQ acerta apenas onde há normativo único e estável.

**Nova descoberta:** limiar de escalada diverge — FAQ usa R$50.000 (item 27); SLA-2024 usa R$100.000.

---

## Entrega 5 — Jornada do Atendente

**Objetivo:** Especificar os três fluxos operacionais do assistente e os guardrails de comportamento.

**Fluxo principal (5 passos):** chamado ativo → pergunta em linguagem natural → busca na base → resposta estruturada (com 5 campos obrigatórios) → uso no atendimento com sinalização ✅.

**Fluxo de fallback (3 situações):**
- Situação A: sem resposta confiante → declaração de limitação com motivo específico + escalada estruturada.
- Situação B: atendente discorda → não usa a resposta → aciona feedback → consulta supervisor.
- Situação C: única fonte é o FAQ informal → resposta com marcação ⚠ obrigatória → atendente decide.

**Fluxo de feedback (4 passos):** 3 ações rápidas (✅/⚠/❌) → formulário de detalhamento → roteamento automático por área → ciclo de atualização com notificação ao atendente.

**4 guardrails:**
- G1: Nunca afirmar prazo, valor ou condição não documentado.
- G2: Sempre sinalizar versões conflitantes do mesmo documento.
- G3: FAQ nunca equivale a normativo formal.
- G4: Escalada apenas para contatos formalizados em normativo.

---

## Entrega 6 — Diagrama Visual de Fluxo

**Objetivo:** Representar os 3 fluxos da jornada do atendente em diagrama visual interativo.

**Formato:** SVG renderizado inline no chat com 4 trilhas codificadas por cor (azul = principal, coral = fallback A, âmbar = fallback FAQ, teal = feedback), losangos de decisão, painel lateral de guardrails e seta tracejada de ciclo fechado.

**Limitação identificada:** exportação para JPG não é possível diretamente pelo assistente. Alternativas comunicadas: print screen, salvar imagem pelo navegador, ou gerar HTML standalone para download.

---

## Entrega 7 — Especificação de Requisitos do Pipeline de RAG — v1.0

**Objetivo:** Especificar em linguagem de produto (não técnica) os requisitos do pipeline de RAG.

**6 seções principais:**
1. Fontes de dados indexadas — 4 níveis hierárquicos.
2. Documentos excluídos — 8 categorias com motivo.
3. Documentos contraditórios — 3 camadas: editorial, marcação de status, tempo real.
4. Comportamento sem resposta — 5 cenários com formato de resposta.
5. Tempo de atualização — 5 tipos com SLA.
6. Rastreabilidade — 7 campos obrigatórios, ciclo de feedback, auditoria de 90 dias.
7. Pré-condições para lançamento — 6 itens com responsável.
8. Temas fora do escopo da fase 1 — 6 temas.

---

## Entrega 8 — Especificação de Requisitos do Pipeline de RAG — v2.0

**Objetivo:** Revisar a v1.0 corrigindo gaps e ambiguidades identificados.

**Gaps corrigidos da v1.0:**

| Gap na v1.0 | Correção na v2.0 |
|---|---|
| Critério de elegibilidade de fontes era vago | 4 critérios objetivos verificáveis adicionados |
| Lista de exclusões estática | Critério dinâmico + processo de quarentena com SLA |
| Conflito entre documentos do mesmo nível não tratado | Regra de desempate por data; protocolo se NovaTech não resolve antes do go-live |
| Pergunta ambígua não coberta | Cenário D adicionado com formato de resposta prescrito |
| Pergunta multi-partes não coberta | Cenário E adicionado |
| Baixa similaridade semântica não coberta | Cenário G adicionado |
| Responsável pelo push de reindexação indefinido | Tabela de responsáveis de base por área com obrigações |
| Sem SLA de contingência nem rollback | SLA de contingência + processo de rollback em 4 passos |
| Formato visual de rastreabilidade indefinido | Bloco colapsável especificado; alertas não ocultáveis |
| Critério de nível de confiança subjetivo | 4 níveis com critérios objetivos |
| Sem conformidade LGPD | Regras de anonimização e retenção diferenciada |
| Sem métricas de sucesso | 6 métricas com meta numérica e baseline |
| Sem critérios de qualidade de resposta | 6 critérios verificáveis adicionados |
| Acessibilidade e idioma não cobertos | Seção 10 nova |
| Pré-condições sem critério de aceite | Critério verificável para cada pré-condição |

**10 seções na v2.0** (vs. 8 na v1.0): adicionadas Seção 9 (Qualidade de Resposta) e Seção 10 (Acessibilidade e Idioma).

---

## Arquivos Produzidos nesta Sessão

| Arquivo | Descrição |
|---|---|
| `novatech-analise-documental.md` | Inventário de documentos, mapa de temas cobertos e 7 hipóteses de gaps |
| `novatech-inconsistencias-proc042-v1-v2.md` | 5 inconsistências diretas + 2 divergências estruturais entre versões da PROC-042 |
| `novatech-inconsistencias-faq-proc042v2.md` | 4 inconsistências diretas + 3 zonas de ambiguidade entre FAQ e PROC-042-v2 |
| `novatech-cruzamento-faq-inconsistencias-gaps.md` | Cruzamento de 11 temas em 4 quadrantes; 3 padrões sistêmicos identificados |
| `novatech-jornada-atendente-assistente-ia.md` | 3 fluxos operacionais + 4 guardrails + diagrama ASCII de resumo |
| `novatech-especificacao-requisitos-rag.md` | Especificação de produto v1.0 — 8 seções |
| `novatech-especificacao-requisitos-rag-v2.md` | Especificação de produto v2.0 — 10 seções, gaps da v1 corrigidos |

---

*Registro gerado ao final da sessão de trabalho. O conteúdo completo de cada entrega está nos arquivos individuais listados acima.*
