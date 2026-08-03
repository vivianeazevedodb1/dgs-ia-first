# Harness de Produto — NovaTech Assistant
## Governança de Mudanças: Matriz de Aprovação Humana

**Versão:** 1.0  
**Data:** 05/06/2026  
**Elaborado por:** Product Specialist Sênior  
**Destinatários:** Product · Tech Lead · QA · Operações · Compliance · Comercial · Delivery Manager

> Este documento define, para cada tipo de mudança no assistente, quem propõe, quem revisa, quem aprova, quem pode bloquear e quais evidências são obrigatórias antes do deploy em produção. Expressões vagas como "área responsável" ou "quando necessário" foram substituídas por papéis e ações concretas.

---

## Princípios da Governança de Mudanças

**P1 — Separação entre quem propõe e quem aprova.** O mesmo papel não pode propor e aprovar uma mudança. Toda aprovação é feita por alguém diferente de quem a iniciou.

**P2 — Escalada por impacto.** Mudanças com impacto em clientes, obrigações legais ou regulação externa (ANTT) exigem Compliance como aprovador obrigatório, independentemente do tipo técnico da mudança.

**P3 — Evidências obrigatórias antes da aprovação.** Aprovação sem evidências não é válida. Cada tipo de mudança tem uma lista mínima de evidências que devem existir como artefatos verificáveis (documentos, logs, resultados de testes) antes que qualquer aprovador possa assinar.

**P4 — Direito de bloqueio.** Qualquer papel listado como "pode bloquear" pode impedir o deploy unilateralmente por até 5 dias úteis, período em que o problema identificado deve ser resolvido ou o bloqueio deve ser escalado ao Delivery Manager para mediação.

**P5 — Aprovação assíncrona vs. síncrona.** Mudanças de nível crítico exigem reunião síncrona de aprovação (mínimo 30 minutos com todos os aprovadores presentes). Mudanças de nível alto aceitam aprovação via PR com comentário registrado. Mudanças de nível médio e baixo aceitam aprovação assíncrona por e-mail ou comentário no sistema de chamados.

---

## Papéis e Responsabilidades

| Papel | Responsabilidade no processo de aprovação |
|---|---|
| **Product Specialist (PS)** | Avalia impacto no produto, na experiência do atendente e nos guardrails de produto. Propõe a maioria das mudanças de prompt e interface. |
| **Tech Lead (TL)** | Avalia viabilidade técnica, impacto no pipeline, risco de regressão. Aprova mudanças de código, infra e pipeline. |
| **QA** | Executa a suite de validação, preenche a tabela de avaliação, emite parecer formal de qualidade. Sem parecer do QA, nenhuma mudança avança. |
| **Operações** | Aprova mudanças que afetam procedimentos operacionais e regras de atendimento ao cliente (temas: devolução, rastreamento, frete). |
| **Compliance** | Aprova mudanças que envolvem obrigações legais, regulação ANTT, cargas perigosas, dados pessoais e vigência de documentos normativos. Pode bloquear qualquer mudança. |
| **Comercial** | Aprova mudanças que envolvem SLAs, valores contratuais, tiers de clientes e regras de frete com impacto em contrato. |
| **Gestor Documental** | Responsável por validar alterações de vigência, resolução de conflitos documentais e publicação de novos normativos. É o responsável de base das áreas Operações, Compliance ou Comercial conforme o documento. |
| **Delivery Manager (DM)** | Media conflitos de aprovação, garante que os prazos do projeto não prejudiquem a qualidade, aprova exceções documentadas. Não aprova mudanças de conteúdo. |

---

## Níveis de Mudança

| Nível | Descrição geral | Tipo de aprovação | SLA para decisão |
|---|---|---|---|
| **Crítico** | Afeta guardrails, modelo, cargas perigosas, obrigações legais ou contratuais | Reunião síncrona obrigatória | 2 dias úteis |
| **Alto** | Afeta prompt de sistema, thresholds, fallback, domínios, SLAs ou novos documentos normativos | PR com revisão assíncrona | 3 dias úteis |
| **Médio** | Afeta metadados, chunking, documentos informativos, interface | Aprovação assíncrona por e-mail ou comentário | 5 dias úteis |
| **Baixo** | Correções de texto, ingestão de documentos de Nível 3, treinamento de usuário | Aprovação por Tech Lead ou QA | 1 dia útil |

---

## Matriz de Aprovação por Tipo de Mudança

### MA-01 — Mudança de regras de negócio no pipeline

**Descrição:** Alteração em código que implementa uma regra de negócio — ex: critério de downgrade de confiança, lógica de detecção de carga perigosa, lista de temas bloqueados (`BLOCKED_TOPICS`), critério de seleção do chunk vencedor.  
**Nível:** Crítico

| Campo | Responsável |
|---|---|
| **Propõe** | Product Specialist |
| **Implementa** | Engenharia (sob orientação do Tech Lead) |
| **Revisa (técnico)** | Tech Lead |
| **Revisa (produto)** | QA |
| **Aprova** | Product Specialist + Tech Lead + QA |
| **Pode bloquear** | Compliance (se a regra afeta tema regulatório ou dados pessoais) · Operações (se a regra afeta procedimentos de atendimento) |
| **Aprovação final** | Product Specialist (responsável pelo produto) |

**Evidências obrigatórias antes da aprovação:**
- Descrição escrita da regra anterior e da regra nova, com justificativa.
- Resultado da suite de validação pré-produção (todos os 15 casos do golden set).
- Caso de regressão derivado do feedback ou incidente que motivou a mudança (quando aplicável).
- Parecer formal do QA com resultado da tabela de avaliação por dimensão.

---

### MA-02 — Novo documento normativo (Nível 1)

**Descrição:** Ingestão de novo documento com status `vigente` de categoria POL, PROC ou SLA — ex: publicação da PROC-043 (Frete de Cargas Perigosas) ou novo aditivo ao SLA-2024.  
**Nível:** Alto

| Campo | Responsável |
|---|---|
| **Propõe** | Gestor Documental da área publicadora (Operações, Compliance ou Comercial) |
| **Valida conteúdo** | Gestor Documental |
| **Executa ingestão** | Tech Lead |
| **Revisa impacto no produto** | Product Specialist |
| **Testa retrieval** | QA (executa M-04 — precisão de recuperação para o tema do novo documento) |
| **Aprova** | Gestor Documental (conteúdo correto) + Tech Lead (ingestão concluída) + QA (golden set aprovado) |
| **Pode bloquear** | Compliance (se o documento envolve regulação externa ou cargas perigosas) |
| **Aprovação final** | Gestor Documental da área responsável pelo documento |

**Evidências obrigatórias:**
- Documento publicado na fonte oficial (SharePoint ou Confluence) com metadados completos: `doc_name`, `doc_version`, `responsável`, `data_publicação`.
- Confirmação escrita do Gestor Documental de que o conteúdo está correto e vigente.
- Resultado de M-04 para as queries do tema do novo documento (precisão ≥ 90%).
- Golden set executado com resultado ≥ aprovação para todos os casos do domínio afetado.

---

### MA-03 — Alteração de vigência documental (arquivamento ou reativação)

**Descrição:** Mudança de `status` de um documento no índice — ex: arquivar PROC-042 v1.0, reativar documento que estava em revisão, atualizar `last_confirmed_at`.  
**Nível:** Alto (arquivamento de Nível 1) · Médio (confirmação de vigência sem arquivamento)

| Campo | Responsável |
|---|---|
| **Propõe** | Gestor Documental da área responsável pelo documento |
| **Implementa** | Tech Lead (alteração de metadado no índice) |
| **Revisa impacto** | Product Specialist (verificar se o documento afeta casos do golden set) |
| **Testa** | QA (executa golden set para todos os casos do domínio do documento alterado) |
| **Aprova** | Gestor Documental (declaração formal de vigência) + Tech Lead (execução) + QA (parecer) |
| **Pode bloquear** | Compliance (se o documento contiver regras com implicação regulatória) · Comercial (se for documento com implicação contratual, ex: SLA-2024) |
| **Aprovação final** | Gestor Documental da área responsável |

**Evidências obrigatórias:**
- Declaração formal escrita do Gestor Documental: "O documento [nome] [versão] está [arquivado / vigente] a partir de [data]. Substitui / é substituído por [nome + versão], se aplicável."
- Para arquivamento: confirmação de que a versão substituta está indexada e com status `vigente`.
- Golden set executado; nenhum caso critico ou high regrediu.

---

### MA-04 — Resolução de documentos conflitantes

**Descrição:** Declaração formal de qual versão de um documento prevalece quando dois documentos coexistem com regras divergentes — ex: PROC-042 v1.0 vs. v2.0. Inclui decisão de arquivamento da versão perdedora.  
**Nível:** Crítico

| Campo | Responsável |
|---|---|
| **Propõe** | Product Specialist (identifica o conflito) |
| **Decide qual versão prevalece** | Gestor Documental da área responsável + Compliance (se regulatório) + Comercial (se contratual) |
| **Implementa resolução no índice** | Tech Lead |
| **Testa** | QA (golden set + casos específicos do documento conflitante) |
| **Aprova** | Gestor Documental + Tech Lead + QA + Compliance (obrigatório se envolve carga perigosa ou obrigação contratual) |
| **Pode bloquear** | Compliance · Comercial · Gestor Documental (qualquer um pode bloquear até que a decisão editorial esteja documentada) |
| **Aprovação final** | Gestor Documental da área responsável |

**Evidências obrigatórias:**
- Documento de decisão editorial assinado pelo Gestor Documental: "A versão [X] prevalece sobre a versão [Y] a partir de [data]. Motivo: [justificativa]."
- Confirmação de que a versão perdedora foi marcada `status: "arquivado"` no índice.
- `has_conflict: false` para as queries do domínio afetado após a resolução (verificado via log).
- Golden set executado; casos GD-006 e GD-014 aprovados.

---

### MA-05 — Alteração de guardrail

**Descrição:** Mudança em qualquer guardrail formalizado (GR-01 a GR-17) — seja no system prompt, no código de validação (P-06, P-07, P-08, P-09, P-10) ou na lista de bloqueados.  
**Nível:** Crítico

| Campo | Responsável |
|---|---|
| **Propõe** | Product Specialist |
| **Revisa impacto técnico** | Tech Lead |
| **Revisa impacto regulatório** | Compliance (obrigatório para guardrails que envolvem carga perigosa, dados pessoais ou documentos normativos) |
| **Testa** | QA (suite completa de validação pré-produção) |
| **Aprova** | Product Specialist + Tech Lead + QA + Compliance |
| **Pode bloquear** | Compliance (direito de bloqueio unilateral em guardrails que afetam regulação ANTT ou LGPD) · Operações (em guardrails que afetam procedimentos de atendimento) |
| **Aprovação final** | Product Specialist (com Compliance co-assinando para guardrails regulatórios) |

**Evidências obrigatórias:**
- Documento descrevendo a mudança: guardrail anterior (texto exato), guardrail proposto (texto exato), justificativa, casos afetados.
- Para remoção de guardrail: análise de risco assinada por Compliance.
- Suite completa de validação pré-produção executada; nenhuma regra bloqueante B-01 a B-10 ativada.
- Avaliação humana TH-04 (riscos de comunicação) para os casos do golden set afetados.

---

### MA-06 — Mudança do modelo LLM

**Descrição:** Troca de versão do GPT-4o ou mudança para outro modelo (ex: GPT-4o → GPT-4o-mini ou novo modelo Azure OpenAI).  
**Nível:** Crítico

| Campo | Responsável |
|---|---|
| **Propõe** | Tech Lead (proposta técnica) com validação do Product Specialist |
| **Avalia impacto de produto** | Product Specialist |
| **Avalia impacto técnico** | Tech Lead |
| **Testa** | QA (golden set completo + amostragem de groundedness de 30 respostas) |
| **Aprova** | Product Specialist + Tech Lead + QA + Delivery Manager (impacto em prazo/custo) |
| **Pode bloquear** | Compliance (se o novo modelo mudar o comportamento em temas regulatórios) · Product Specialist (se a qualidade das respostas degradar) |
| **Aprovação final** | Tech Lead (execução) com co-assinatura do Product Specialist |

**Evidências obrigatórias:**
- Resultado do golden set para o modelo candidato vs. modelo atual (tabela comparativa completa com todas as 18 dimensões).
- Resultado de groundedness (M-02) para o modelo candidato em amostra de 30 respostas.
- Resultado de latência (M-08): TTFT p95 e resposta completa p95 para o modelo candidato.
- Confirmação de que o modelo candidato está disponível no Azure OpenAI com SLA de uptime equivalente (ADR-0001).
- Nenhuma das 10 regras bloqueantes (B-01 a B-10) ativada para o modelo candidato.

---

### MA-07 — Alteração de prompt de sistema

**Descrição:** Qualquer mudança no conteúdo do system prompt — adição de instrução de domínio, reescrita de guardrail de prompt (GR-05, GR-08, GR-12), alteração de formato de resposta, mudança de instrução de completude (ex: SLA com incidente crítico).  
**Nível:** Alto (instrução de domínio ou completude) · Crítico (guardrail de prompt ou instrução sobre carga perigosa)

| Campo | Responsável |
|---|---|
| **Propõe** | Product Specialist |
| **Implementa** | Prompt Engineering (ou Tech Lead quando não há papel dedicado) |
| **Revisa impacto técnico** | Tech Lead |
| **Testa** | QA (golden set + amostragem de groundedness para o tema afetado) |
| **Aprova — nível Alto** | Product Specialist + Tech Lead + QA |
| **Aprova — nível Crítico** | Product Specialist + Tech Lead + QA + Compliance |
| **Pode bloquear** | Compliance (instrução que afeta carga perigosa, dados pessoais, regulação) · Operações (instrução que afeta procedimentos de atendimento) |
| **Aprovação final** | Product Specialist |

**Evidências obrigatórias:**
- Diff exato do prompt: texto removido e texto adicionado.
- Justificativa com referência ao feedback ou incidente que motivou a mudança.
- Resultado do golden set para o prompt candidato; todos os casos do domínio afetado aprovados.
- Para instruções sobre carga perigosa: casos GD-002, GD-007, GD-012, GD-013 aprovados (TH-05).

---

### MA-08 — Mudança de thresholds de confiança

**Descrição:** Alteração nos critérios que definem Alta/Moderada/Baixa confiança ou recalibração do threshold de retrieval (0,72).  
**Nível:** Alto

| Campo | Responsável |
|---|---|
| **Propõe** | Tech Lead (threshold de retrieval) ou Product Specialist (critérios de confiança) |
| **Implementa** | Tech Lead |
| **Analisa impacto** | Product Specialist (impacto na experiência do atendente e nos guardrails) |
| **Testa** | QA (M-04 precisão de recuperação + M-05 taxa de baixa confiança + golden set) |
| **Aprova** | Product Specialist + Tech Lead + QA |
| **Pode bloquear** | Compliance (se a mudança reduz a sinalização de baixa confiança para temas regulatórios) · Operações (se aumenta a taxa de ausência em temas de alta frequência) |
| **Aprovação final** | Product Specialist |

**Evidências obrigatórias:**
- Análise comparativa: resultado do golden set com threshold atual vs. threshold candidato (tabela com diferença de scores por caso).
- M-04 (precisão de recuperação) para o golden set com threshold candidato.
- M-11 (ausência falsa) medido com threshold candidato sobre amostra de 20 consultas.
- Confirmação de que nenhum caso `critical` do golden set passou de "aprovado" para "reprovado" com a mudança.

---

### MA-09 — Alteração de comportamento de fallback

**Descrição:** Mudança nas condições que ativam declaração de ausência — ex: adição ou remoção de tema da lista `BLOCKED_TOPICS`, alteração na lógica de declaração de ausência para documentos em revisão.  
**Nível:** Alto

| Campo | Responsável |
|---|---|
| **Propõe** | Product Specialist |
| **Implementa** | Tech Lead (alteração de código em BC-02 ou BC-03) |
| **Revisa** | QA |
| **Aprova** | Product Specialist + Tech Lead + QA |
| **Pode bloquear** | Compliance (se a mudança altera o comportamento para temas de carga perigosa ou regulatórios) · Operações (se remove ausência em tema que precisa de escaldo) |
| **Aprovação final** | Product Specialist |

**Evidências obrigatórios:**
- Lista explícita dos temas adicionados ou removidos do fallback, com justificativa.
- Resultado do golden set para os casos GD-005, GD-007, GD-009, GD-010 (casos de ausência e fora do escopo).
- Confirmação de que M-11 (ausência falsa) não piorou com a mudança.

---

### MA-10 — Inclusão de novo domínio

**Descrição:** Expansão do escopo do assistente para cobrir um novo tema antes declarado fora do escopo — ex: habilitar seguro de carga quando o normativo formal for publicado, ou incluir PROC-043 após publicação pelo Compliance.  
**Nível:** Crítico

| Campo | Responsável |
|---|---|
| **Propõe** | Product Specialist (com base no painel de gaps e na disponibilidade do normativo) |
| **Valida normativo** | Gestor Documental da área responsável |
| **Avalia impacto de produto** | Product Specialist |
| **Avalia impacto técnico** | Tech Lead |
| **Cria casos do golden set** | QA (mínimo 2 casos: 1 factual e 1 de fronteira para o novo domínio) |
| **Testa** | QA (golden set atualizado) |
| **Aprova** | Product Specialist + Tech Lead + QA + Gestor Documental + Compliance (se o domínio envolve regulação) + Comercial (se envolve contrato ou SLA) |
| **Pode bloquear** | Qualquer aprovador listado |
| **Aprovação final** | Product Specialist |

**Evidências obrigatórias:**
- Normativo formal publicado na fonte oficial com metadados completos.
- Remoção do tema da lista de temas fora do escopo (atualização de documentação e código).
- Golden set atualizado com mínimo 2 novos casos para o domínio, aprovados.
- Comunicado de atualização de escopo para os atendentes (via Teams) pronto para envio após o deploy.

---

### MA-11 — Mudança em respostas sobre cargas perigosas

**Descrição:** Qualquer alteração que afete — direta ou indiretamente — o comportamento do assistente para queries sobre cargas perigosas ANTT classes 1–6: mudança de prompt, código, filtros, metadados ou documentos que incidam sobre POL-001 Seção 3.2, PROC-043 ou qualquer regra de frete/devolução de carga perigosa.  
**Nível:** Crítico (sem exceção)

| Campo | Responsável |
|---|---|
| **Propõe** | Product Specialist ou Tech Lead, conforme o tipo técnico da mudança |
| **Avalia impacto regulatório** | Compliance (obrigatório — não pode ser substituído por outro papel) |
| **Avalia impacto operacional** | Operações |
| **Testa** | QA (casos GD-002, GD-007, GD-012, GD-013 obrigatórios + golden set completo) |
| **Aprova** | Product Specialist + Tech Lead + QA + Compliance + Operações |
| **Pode bloquear** | Compliance (direito de bloqueio com prazo de resposta de 1 dia útil) · Operações |
| **Aprovação final** | Compliance (co-assinatura obrigatória) + Product Specialist |

**Evidências obrigatórias:**
- Descrição explícita de como a mudança afeta o comportamento para queries de carga perigosa.
- Casos GD-002, GD-007, GD-012, GD-013 executados com resultado "Aprovado" em todos os testes determinísticos e semânticos.
- TH-05 (avaliação humana — carga perigosa): 4/4 aprovados.
- Parecer escrito do Compliance confirmando que a mudança não introduz risco regulatório.

---

### MA-12 — Mudança em SLAs e valores contratuais

**Descrição:** Alteração que afeta respostas sobre SLAs, prazos contratuais, multiplicadores de frete, limiares financeiros de tier ou penalidades — qualquer informação que gere expectativa contratual para o cliente.  
**Nível:** Crítico

| Campo | Responsável |
|---|---|
| **Propõe** | Gestor Documental do Comercial (para mudanças que originam de novos documentos) ou Product Specialist (para mudanças de prompt ou pipeline) |
| **Valida valores** | Gestor Documental do Comercial |
| **Avalia impacto de produto** | Product Specialist |
| **Avalia impacto técnico** | Tech Lead |
| **Testa** | QA (casos GD-003, GD-008, GD-014, GD-015 obrigatórios + golden set completo) |
| **Aprova** | Product Specialist + Tech Lead + QA + Comercial + Compliance (se há implicação contratual legal) |
| **Pode bloquear** | Comercial (valores ou SLAs incorretos) · Compliance (implicação contratual ou legal) |
| **Aprovação final** | Gestor Documental do Comercial + Product Specialist |

**Evidências obrigatórias:**
- Confirmação escrita do Gestor Documental do Comercial de que os valores na nova versão estão corretos.
- Casos GD-003, GD-008, GD-014, GD-015 aprovados.
- Nenhum valor numérico de SLA, multiplicador ou limiar financeiro diferente do documento fonte na resposta do candidato.

---

### MA-13 — Alteração que afeta clientes ou obrigações legais

**Descrição:** Mudança que altera o que o atendente comunica ao cliente em temas com implicação legal ou contratual — inclui: mudança de prazo de devolução, SLA de resolução, elegibilidade de devolução, critérios de penalidade, qualquer tema coberto por POL-001, SLA-2024 ou PROC-042-v2.  
**Nível:** Crítico (sem exceção)

| Campo | Responsável |
|---|---|
| **Propõe** | Gestor Documental da área responsável (quando origina de mudança documental) ou Product Specialist (quando origina de melhoria de produto) |
| **Valida impacto legal** | Compliance (obrigatório) |
| **Valida impacto contratual** | Comercial (obrigatório) |
| **Avalia impacto de produto** | Product Specialist |
| **Testa** | QA (golden set completo + avaliação humana TH-04 riscos de comunicação) |
| **Aprova** | Product Specialist + Tech Lead + QA + Compliance + Comercial + Operações |
| **Pode bloquear** | Compliance (direito de bloqueio unilateral) · Comercial · Operações |
| **Aprovação final** | Compliance + Product Specialist (co-assinaturas obrigatórias) |

**Evidências obrigatórias:**
- Análise de impacto escrita: quais respostas mudam, para quais temas e quais clientes são potencialmente afetados.
- Parecer de Compliance sobre risco legal.
- Parecer de Comercial sobre risco contratual.
- Golden set executado sem nenhuma regra bloqueante ativada.
- TH-04 (riscos de comunicação): zero respostas com risco identificado.
- Comunicado de mudança para atendentes preparado e aprovado pela Coordenação de Atendimento antes do deploy.

---

## Tabela Resumida da Matriz de Aprovação

| Tipo de mudança | Product Specialist | Tech Lead | QA | Área de negócio | Compliance | Aprovação final |
|---|---|---|---|---|---|---|
| **MA-01** Regras de negócio | Propõe + Aprova | Revisa + Aprova | Aprova | Operações: pode bloquear | Pode bloquear | Product Specialist |
| **MA-02** Novo documento normativo | Revisa impacto | Executa ingestão + Aprova | Testa + Aprova | Gestor Documental: Propõe + Aprova | Pode bloquear | Gestor Documental |
| **MA-03** Alteração de vigência | Revisa impacto | Implementa + Aprova | Testa + Aprova | Gestor Documental: Propõe + Aprova | Pode bloquear | Gestor Documental |
| **MA-04** Resolução de conflito documental | Propõe | Implementa + Aprova | Testa + Aprova | Gestor Documental: Aprova · Comercial/Operações: pode bloquear | Aprova (se regulatório) | Gestor Documental |
| **MA-05** Alteração de guardrail | Propõe + Aprova | Revisa + Aprova | Aprova | Operações: pode bloquear | Aprova + pode bloquear | Product Specialist (+ Compliance em guardrails regulatórios) |
| **MA-06** Mudança do modelo | Valida + Aprova | Propõe + Implementa + Aprova | Aprova | — | Pode bloquear | Tech Lead + Product Specialist |
| **MA-07** Alteração de prompt | Propõe + Aprova | Revisa + Aprova | Aprova | Operações: pode bloquear | Aprova (nível crítico) + pode bloquear | Product Specialist |
| **MA-08** Thresholds de confiança | Aprova | Propõe + Implementa + Aprova | Aprova | Operações: pode bloquear | Pode bloquear | Product Specialist |
| **MA-09** Comportamento de fallback | Propõe + Aprova | Implementa + Aprova | Aprova | Operações: pode bloquear | Pode bloquear | Product Specialist |
| **MA-10** Novo domínio | Propõe + Aprova | Aprova | Cria casos + Aprova | Gestor Documental: Aprova · Comercial/Operações: aprova/pode bloquear | Aprova (se regulatório) | Product Specialist |
| **MA-11** Carga perigosa (qualquer mudança) | Propõe + Aprova | Aprova | Aprova | Operações: Aprova | **Co-assinatura obrigatória** + pode bloquear | Compliance + Product Specialist |
| **MA-12** SLAs e valores contratuais | Aprova | Aprova | Aprova | Gestor Documental Comercial: Propõe + **Aprova** | Aprova (se implicação legal) + pode bloquear | Gestor Documental Comercial + Product Specialist |
| **MA-13** Obrigações legais / impacto em clientes | Aprova | Aprova | Aprova (+ TH-04) | Operações: Aprova · Comercial: **Aprova** | **Co-assinatura obrigatória** + pode bloquear | Compliance + Product Specialist |

---

## Combinações de Mudança

Quando uma única alteração ativa mais de um tipo de mudança, aplica-se a matriz mais restritiva de cada tipo ativado. Exemplos:

**Exemplo 1:** Publicação da PROC-043 (Frete de Cargas Perigosas)  
Ativa: MA-02 (novo documento normativo) + MA-10 (novo domínio) + MA-11 (carga perigosa)  
Resultado: nível Crítico. Aprovadores: Product Specialist + Tech Lead + QA + Gestor Documental Compliance + Compliance + Operações. Aprovação final: Compliance + Product Specialist.

**Exemplo 2:** Recalibração do threshold de 0,72 para 0,70 após feedback sobre ausência falsa  
Ativa: MA-08 (threshold de confiança)  
Resultado: nível Alto. Aprovadores: Product Specialist + Tech Lead + QA. Possíveis bloqueadores: Compliance, Operações. Aprovação final: Product Specialist.

**Exemplo 3:** Ajuste de prompt para incluir SLA de incidente crítico em respostas de SLA  
Ativa: MA-07 (prompt de sistema) + MA-12 (SLAs e valores contratuais)  
Resultado: nível Crítico. Aprovadores: Product Specialist + Tech Lead + QA + Comercial + Compliance. Aprovação final: Gestor Documental Comercial + Product Specialist.

---

## Evidências Padrão Obrigatórias para Qualquer Mudança

Independentemente do tipo de mudança, as seguintes evidências são sempre obrigatórias:

1. **Parecer do QA** — documento formal preenchido com resultado da tabela de avaliação por dimensão (seção 5 do harness de validação). Sem parecer do QA, nenhuma aprovação é válida.
2. **Resultado do golden set** — resultado da execução dos 15 casos, identificando quais passaram e quais falharam. Deve ser o resultado do candidato, não do baseline.
3. **Registro de deploy planejado** — `deploy_id` reservado, janela de deploy confirmada, procedimento de rollback documentado.
4. **Comunicação planejada** — para mudanças de nível Alto e Crítico, a comunicação ao atendente (MA-10 / MA-13) deve estar preparada e aprovada pela Coordenação de Atendimento antes do deploy.

---

## Tratamento de Exceções e Urgências

Em situações onde uma mudança precisa ser promovida em menos de 24 horas (ex: resposta incorreta crítica identificada em produção causando orientação errada sobre carga perigosa):

1. O Product Specialist classifica a situação como **hotfix crítico** e notifica o Delivery Manager.
2. O Delivery Manager convoca reunião de emergência com Product Specialist + Tech Lead + QA + Compliance (se aplicável) com prazo máximo de 2 horas.
3. As evidências obrigatórias continuam sendo exigidas, mas podem ser produzidas em paralelo à implementação (não antes).
4. O deploy em produção exige aprovação de Product Specialist + Tech Lead + QA. Compliance pode aprovar assincronamente em até 4 horas após o deploy.
5. O registro do hotfix inclui: `hotfix: true`, `emergency_justification`, `post_deploy_compliance_approval_deadline`.
6. Se Compliance bloquear o hotfix após o deploy, o rollback é executado imediatamente sem discussão.

---

*Documento elaborado com base nos guardrails formalizados (guardrails-novatech.md), no harness de validação pré-produção (novatech-harness-validacao.md) e no fluxo de feedback (novatech-harness-feedback.md). Revisão esperada após 90 dias de operação, com base nos casos de aprovação reais processados.*
