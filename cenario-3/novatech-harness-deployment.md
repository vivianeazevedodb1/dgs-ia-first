# Harness de Produto — NovaTech Assistant
## Práticas de Implantação Segura e Rollback

**Versão:** 1.0  
**Data:** 05/06/2026  
**Elaborado por:** Product Specialist Sênior  
**Destinatários:** Tech Lead · Engenharia · QA · Product Specialist · Delivery Manager

> Este documento define as práticas operacionais que tornam cada deploy reversível, rastreável e seguro. Cada prática é descrita com o mecanismo concreto, o responsável pela execução e o critério que determina quando ela é suficiente.

---

## 1. Ambiente de Staging

### Definição

O ambiente de staging é uma réplica completa do ambiente de produção, com acesso restrito ao time interno (Tech Lead, Engenharia, QA, Product Specialist). Nenhuma mudança chega à produção sem ter sido validada em staging primeiro.

### Componentes replicados

| Componente | Staging | Produção | Paridade exigida |
|---|---|---|---|
| Azure OpenAI (GPT-4o) | Mesmo endpoint, mesma versão de modelo | — | Versão idêntica |
| Azure AI Search | Índice separado (`novatech-idx-staging`) com snapshot do índice de produção | `novatech-idx-prod` | Snapshot atualizado a cada deploy candidato |
| System prompt | Arquivo `/prompts/system_prompt_staging.txt` — candidato em teste | `/prompts/system_prompt_prod.txt` — versão aprovada | Staging contém o candidato; prod contém o aprovado |
| Pipeline de ingestão | Mesmos parâmetros de chunking, threshold e filtros | — | Idêntico |
| Validadores de schema | Mesma versão dos validadores Zod/JSON Schema | — | Idêntico |
| Bot do Teams | Canal de staging separado (`#novatech-assistant-staging`) | Canal de produção | Versão candidata no canal de staging |
| Painel web interno | Instância de staging com URL separada | — | Mesma base de código |

### Atualização do snapshot do índice

Antes de cada ciclo de validação pré-deploy, o Tech Lead executa:

```bash
# Copiar snapshot do índice de produção para staging
az search index create --service-name novatech-search-staging \
  --name novatech-idx-staging \
  --source-index novatech-idx-prod \
  --timestamp $(date -u +%Y%m%dT%H%M%SZ)
```

O snapshot garante que os testes em staging usam exatamente os mesmos documentos e metadados que estão em produção, exceto pelas mudanças sendo testadas.

### Acesso e isolamento

- Atendentes não têm acesso ao canal de staging.
- Os 5 atendentes-piloto têm acesso para testes de usabilidade quando explicitamente convocados pelo Product Specialist.
- Dados de staging não são incluídos nas métricas de produção (M-01 a M-12).
- Feedbacks coletados em staging são armazenados em tabela separada (`feedback_staging`) e não se misturam com o BC-06 de produção.

---

## 2. Execução do Regression Suite

### Definição

O regression suite é a execução completa dos testes automatizados (TD-01 a TD-15) e semânticos (TS-01 a TS-08) sobre o golden dataset (`/tests/golden/golden-dataset.yaml`) contra o candidato em staging. É obrigatório antes de qualquer deploy.

### Estrutura do suite

```
tests/
├── regression/
│   ├── deterministic/       # TD-01 a TD-15
│   │   ├── schema_validation.test.ts
│   │   ├── source_document.test.ts
│   │   ├── confidence_rules.test.ts
│   │   ├── guardrail_strings.test.ts
│   │   └── latency.test.ts
│   ├── semantic/            # TS-01 a TS-08
│   │   ├── groundedness.test.ts
│   │   ├── completeness.test.ts
│   │   ├── rejected_evidence.test.ts
│   │   └── fallback_adequacy.test.ts
│   ├── golden/              # Casos GD-001 a GD-015
│   │   └── golden-dataset.yaml
│   └── derived/             # Casos derivados de feedbacks reais
│       ├── REG-001.test.ts
│       ├── REG-042.test.ts
│       └── ...
├── contracts/               # Testes de contrato do schema
│   └── response-schema.test.ts
└── performance/             # Testes de latência
    └── latency-p95.test.ts
```

### Gatilhos de execução

| Evento | Suite executado | Responsável |
|---|---|---|
| Push em qualquer branch de feature | Suite determinístico (TD-01 a TD-15) + casos derived | CI automático |
| Abertura de PR | Suite completo (determinístico + semântico + golden set) | CI automático |
| Deploy candidato em staging | Suite completo + latência | QA (manual com resultado registrado) |
| Alteração de modelo ou threshold | Suite completo + golden set + latência (20 execuções) | QA (manual) |
| Reindexação completa | Suite completo + M-04 (golden set de retrieval) | QA (manual) |

### Execução em CI

```yaml
# .github/workflows/regression.yml (ou equivalente Azure DevOps)
on:
  pull_request:
    branches: [main, staging]
  push:
    branches: [staging]

jobs:
  regression:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run deterministic tests
        run: npm run test:deterministic
        env:
          STAGING_ENDPOINT: ${{ secrets.STAGING_ENDPOINT }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_STAGING_KEY }}
      - name: Run semantic tests
        run: npm run test:semantic
      - name: Run golden dataset
        run: npm run test:golden
      - name: Generate evaluation table
        run: npm run report:evaluation-table
        # Salva tabela preenchida como artefato do CI
```

### Resultado obrigatório antes do deploy

O CI deve retornar `exit 0` (todas as etapas passando) para que o PR seja elegível para merge. Um PR com falha em qualquer teste determinístico `critical` não pode ser aprovado independentemente de qualquer outra consideração.

---

## 3. Canary Release e Grupo Piloto

### Definição

Antes de promover uma mudança para todos os 45 atendentes, ela é exposta a um grupo controlado. Há dois mecanismos, usados conforme o nível de risco da mudança:

| Mecanismo | Quando usar | Duração | Tamanho do grupo |
|---|---|---|---|
| **Grupo piloto** | Mudanças de nível Alto e Crítico; novas features; novo domínio | 24–48 horas antes do rollout completo | 5 atendentes (os mesmos do período de staging pré-go-live) |
| **Canary por percentual** | Mudanças de código no pipeline (BC-02, BC-03, validadores) | 2 horas antes do rollout completo | 10% das requisições (~19 consultas/dia) |

### Configuração do grupo piloto

O grupo piloto é definido no Azure AD como grupo `novatech-assistant-pilot`. O bot do Teams roteia as requisições desse grupo para a versão candidata e as demais para a versão de produção atual. A distinção é transparente para o atendente.

```typescript
// Roteamento por grupo no bot
async function routeRequest(userId: string, query: string): Promise<AssistantResponse> {
  const isPilot = await azureAD.isMemberOf(userId, 'novatech-assistant-pilot');
  const endpoint = isPilot ? STAGING_ENDPOINT : PRODUCTION_ENDPOINT;
  return await queryAssistant(endpoint, query);
}
```

### Configuração do canary por percentual

```typescript
// Feature flag de canary (integrado ao sistema de feature flags, Seção 4)
const CANARY_PERCENTAGE = 10; // % de requisições roteadas para o candidato

function isCanary(requestId: string): boolean {
  const hash = crc32(requestId) % 100;
  return hash < CANARY_PERCENTAGE;
}
```

### Critérios para expansão do canary para 100%

Após 2 horas (canary por percentual) ou 24–48 horas (grupo piloto), verificar:

- M-07 no grupo piloto / canary: ❌ ≤ 5% e ✅ ≥ 65%
- Nenhum feedback `F01` (incorreta) sobre tema de carga perigosa, SLA ou devolução
- TTFT p95 no canary ≤ 3s
- Nenhum caso `critical` do golden dataset falhou em produção (verificado por QA)

Se todos os critérios satisfeitos: Tech Lead expande para 100% e registra no log. Se qualquer critério falhar: rollback do canary (restringir novamente ao grupo piloto ou reverter totalmente).

---

## 4. Feature Flags

### Definição

Feature flags permitem ativar ou desativar comportamentos do assistente sem deploy de novo código. São o mecanismo primário para canary, piloto e rollback rápido de features específicas.

### Flags definidas

| Flag ID | Descrição | Estado padrão | Quem pode alterar |
|---|---|---|---|
| `ff_dangerous_cargo_template` | Ativa injeção de bloco fixo com ramal 4500 em respostas de carga perigosa | `enabled` | Tech Lead + PS |
| `ff_confidence_downgrade_level4` | Ativa downgrade automático de confiança para chunks Nível 4 (FAQ) | `enabled` | Tech Lead + PS |
| `ff_source_validation` | Ativa validação de schema que rejeita Alta confiança sem fonte | `enabled` | Tech Lead (nunca desativar sem aprovação crítica) |
| `ff_conflict_notice` | Ativa geração de `conflict_notice` quando `has_conflict: true` | `enabled` | Tech Lead + PS |
| `ff_blocked_topics` | Ativa interceptação de temas bloqueados antes do LLM | `enabled` | PS |
| `ff_sla_completeness_check` | Ativa verificador de completude de SLA (requer incidente crítico na resposta) | `enabled` | PS |
| `ff_streaming` | Ativa entrega de resposta via streaming (SSE) | `enabled` | Tech Lead |
| `ff_new_domain_{nome}` | Ativa novo domínio após aprovação (ex: `ff_new_domain_seguro_carga`) | `disabled` por padrão | PS + aprovadores do MA-10 |
| `ff_canary_v{X}` | Roteia grupo canary para versão candidata | `disabled` por padrão | Tech Lead |

### Armazenamento e acesso

As flags são armazenadas em Azure App Configuration com versionamento automático. Cada alteração de flag registra: `flag_id`, `old_value`, `new_value`, `changed_by`, `timestamp`, `reason`.

```typescript
import { AppConfigurationClient } from '@azure/app-configuration';

const client = new AppConfigurationClient(process.env.APP_CONFIG_CONNECTION_STRING);

async function getFlag(flagId: string): Promise<boolean> {
  const setting = await client.getConfigurationSetting({ key: `.appconfig.featureflag/${flagId}` });
  return JSON.parse(setting.value).enabled;
}
```

### Regra de ouro das flags

As flags `ff_source_validation` e `ff_dangerous_cargo_template` são **flags de segurança**. Desativá-las em produção requer aprovação de Compliance + Tech Lead + Product Specialist (mesmo nível que MA-11). Não podem ser desativadas por hotfix sem esse quórum.

---

## 5. Versionamento de Prompt

### Esquema de versionamento

O system prompt usa versionamento semântico `vX.Y.Z`:

- **X (major):** mudança que altera o comportamento para um guardrail ou domínio inteiro — ex: reescrita da instrução de carga perigosa, alteração do formato de resposta estruturada.
- **Y (minor):** adição de nova instrução de domínio, nova regra de completude, nova restrição de terminologia.
- **Z (patch):** correção de redação sem mudança de comportamento — ex: corrigir typo, clarificar phrasing sem alterar a regra.

### Armazenamento

```
prompts/
├── system_prompt_prod.txt        # Versão ativa em produção
├── system_prompt_staging.txt     # Candidato em teste
├── history/
│   ├── v1.0.0_20260501.txt
│   ├── v1.1.0_20260520.txt
│   ├── v1.2.0_20260601.txt       # Versão atual de produção
│   └── v1.2.1_20260607.txt       # Candidato (ainda não promovido)
└── CHANGELOG_PROMPT.md           # Registro de mudanças por versão
```

### CHANGELOG_PROMPT.md — estrutura obrigatória por entrada

```markdown
## v1.2.1 — 2026-06-07
**Tipo:** minor
**Motivo:** Adicionar instrução de completude de SLA com incidente crítico
**Feedback de origem:** fb_20260605_001a2b3c
**Mudança:**
- Adicionada regra: "Quando tier e SLA forem mencionados, incluir chamados gerais E incidentes críticos"
**Impacto esperado:** Respostas de SLA passam a cobrir os dois cenários
**Casos do golden set afetados:** GD-003, GD-015
**Aprovado por:** PS_01 (2026-06-07T08:30Z) · TL_01 (2026-06-07T08:45Z) · QA_01 (2026-06-07T09:00Z)
```

### Rastreabilidade em cada resposta

O campo `prompt_version` no payload do BC-06 e no registro de feedback registra a versão exata do prompt que gerou cada resposta. Isso permite identificar se um feedback negativo foi gerado por uma versão específica e correlacionar com mudanças de prompt.

---

## 6. Versionamento do Índice

### Esquema de versionamento

O índice usa formato `idx_YYYYMMDD_NNN` onde `YYYYMMDD` é a data de criação e `NNN` é o número sequencial do dia (para múltiplos snapshots no mesmo dia).

Exemplos:
- `idx_20260601_001` — snapshot de produção criado em 01/06/2026
- `idx_20260607_001` — snapshot após ingestão de novo documento
- `idx_20260607_002` — segundo snapshot do dia (após reindexação por problema de chunking)

### Operações que geram novo snapshot

| Operação | Cria novo snapshot? | Nome do snapshot |
|---|---|---|
| Ingestão de novo documento | Sim | `idx_YYYYMMDD_NNN` |
| Atualização de metadados de documento | Sim | `idx_YYYYMMDD_NNN` |
| Reindexação de documento por chunking | Sim | `idx_YYYYMMDD_NNN` |
| Reindexação completa (ex: mudança de embedding model) | Sim | `idx_YYYYMMDD_001` (reset do sequencial) |
| Atualização de `last_confirmed_at` apenas | Sim (metadado crítico) | `idx_YYYYMMDD_NNN` |

### Retenção de snapshots

- Últimos 5 snapshots mantidos no Azure AI Search (para rollback imediato sem reconstrução).
- Snapshots mais antigos exportados para Azure Blob Storage com retenção de 24 meses.
- O snapshot ativo em produção é referenciado na variável de configuração `PROD_INDEX_VERSION` no Azure App Configuration.

### Rastreabilidade

O campo `index_version` no registro de feedback e no log de deploy registra o snapshot exato usado em cada resposta. Isso permite identificar: "qual versão do índice estava ativa quando esse feedback negativo foi gerado?"

---

## 7. Versionamento do Modelo

### Esquema de versionamento

O modelo não tem versionamento próprio do produto — usa o identificador do Azure OpenAI diretamente: `gpt-4o-YYYY-MM-DD` (ex: `gpt-4o-2024-05-13`).

### Variável de configuração

```
PROD_MODEL_VERSION=gpt-4o-2024-05-13
```

Qualquer mudança nessa variável ativa o processo MA-06 (mudança do modelo) do harness de aprovação — nível Crítico com reunião síncrona obrigatória.

### Política de atualização

- **Atualização automática de modelo desabilitada.** O Azure OpenAI não deve atualizar o modelo automaticamente. A versão é fixada por configuração explícita.
- **Aviso de depreciação:** quando a Azure OpenAI notifica depreciação de versão, o Tech Lead abre item de backlog de tipo MA-06 com prazo definido pela data de depreciação.
- **Testes obrigatórios antes de qualquer atualização:** golden set completo com tabela comparativa entre versão atual e candidata (todas as 18 dimensões).

### Rastreabilidade

O campo `model_version` no registro de feedback registra o modelo exato em cada resposta. Em caso de comportamento inesperado após atualização de modelo, é possível filtrar todos os feedbacks gerados pela nova versão e comparar com os da versão anterior.

---

## 8. Registro da Configuração

### Definição

O registro de configuração (`/config/production.yaml` versionado no repositório) documenta o estado completo do sistema em produção a qualquer momento. É a "fotografia" que permite reproduzir exatamente o ambiente de produção ou reverter para qualquer estado anterior.

### Estrutura do registro de configuração

```yaml
# /config/production.yaml
# Atualizado automaticamente pelo pipeline de deploy
# NUNCA editar manualmente — alterações via processo de deploy

environment: production
last_updated: "2026-06-07T10:30:00Z"
last_updated_by: "deploy_pipeline_v1.2.1"
deploy_id: "dep_20260607_v1.2.1"

components:
  model:
    version: "gpt-4o-2024-05-13"
    endpoint: "https://novatech-openai.openai.azure.com/"
    deployment_name: "gpt-4o-prod"
    parameters:
      temperature: 0.0
      max_tokens: 1500
      top_p: 1.0

  prompt:
    version: "v1.2.1"
    file: "prompts/history/v1.2.1_20260607.txt"
    hash_sha256: "a3f8b2c1d4e5f6..."   # Hash do arquivo para detecção de adulteração

  index:
    version: "idx_20260607_001"
    service: "novatech-search-prod"
    index_name: "novatech-idx-prod"
    document_count: 847
    last_ingestion: "2026-06-07T09:00:00Z"

  retrieval:
    threshold: 0.72
    top_k: 5
    filters:
      - "status ne 'arquivado'"
      - "status ne 'blocked'"

  pipeline:
    chunking_strategy: "semantic_sentence"
    chunk_size_tokens: 1500
    chunk_overlap_tokens: 100
    no_cut_patterns: ["table", "numbered_list", "section_header"]

  feature_flags:
    ff_dangerous_cargo_template: true
    ff_confidence_downgrade_level4: true
    ff_source_validation: true
    ff_conflict_notice: true
    ff_blocked_topics: true
    ff_sla_completeness_check: true
    ff_streaming: true

  blocked_topics:
    - id: "carga_danificada"
      matchers: ["carga danificada", "dano em trânsito", "avaria durante"]
    - id: "seguro_carga"
      matchers: ["seguro de carga", "percentual de seguro"]

context_budget:
  system_prompt_tokens: 4000
  chunks_tokens: 8000
  history_turns: 3
  max_total_tokens: 16000

regression_suite:
  version: "v2.3"
  golden_dataset_version: "v1.4"
  last_run: "2026-06-07T08:00:00Z"
  last_run_result: "passed"
  cases_passed: 15
  cases_total: 15
```

### Atualização automática

O pipeline de deploy atualiza `production.yaml` automaticamente após cada deploy bem-sucedido. O arquivo é commitado no repositório com mensagem: `chore(config): update production config for deploy dep_20260607_v1.2.1`.

### Uso para rollback

Para reverter para qualquer estado anterior, o Tech Lead executa:

```bash
# Identificar a versão do config antes do deploy problemático
git log --oneline config/production.yaml

# Extrair configuração anterior
git show HEAD~1:config/production.yaml > config/rollback_target.yaml

# Aplicar cada componente da configuração anterior
./scripts/apply_config.sh config/rollback_target.yaml
```

---

## 9. Monitoramento Intensivo Após Publicação

### Janelas de monitoramento

| Janela | Período | Frequência de verificação | Responsável |
|---|---|---|---|
| **Crítica** | 0–2 horas pós-deploy | Contínua — verificação a cada 15 minutos | Tech Lead + QA |
| **Alta** | 2–24 horas pós-deploy | Horária | Tech Lead (automatizado) · PS (revisão manual no final da janela) |
| **Intensiva** | 1–7 dias pós-deploy | Diária | PS + Tech Lead |
| **Consolidação** | 8–14 dias pós-deploy | A cada 3 dias | PS |

### Métricas monitoradas por janela

**Janela crítica (0–2 horas) — critérios de rollback automático:**

| Métrica | Valor monitorado | Critério de rollback automático |
|---|---|---|
| M-07 taxa de ❌ | % feedbacks negativos | > baseline + 3pp em janela de 30 min |
| M-08 TTFT p95 | Segundos | > 5s em janela de 15 min |
| Casos critical em produção | Resultado dos GD-002, GD-007, GD-012, GD-013 | Qualquer falha |
| Violação de guardrail (automática) | Log de rejeições do validador | Qualquer ocorrência |

**Janela alta (2–24 horas) — critérios de alerta para decisão humana:**

| Métrica | Alerta |
|---|---|
| M-07 taxa de ❌ | > 8% em qualquer hora |
| M-03 alucinação | Qualquer detecção automática |
| M-09 violação de guardrail | Qualquer ocorrência na amostragem |
| M-08 TTFT p95 | > 4s por 2 horas consecutivas |

**Janela intensiva (1–7 dias):**

| Métrica | Frequência | Alerta |
|---|---|---|
| M-02 groundedness | Amostragem de 10 respostas/dia do domínio afetado | Queda > 5pp vs. baseline |
| M-11 ausência falsa | Amostragem de 5 declarações de ausência/dia | > 5% de ausências falsas |
| Feedbacks F01 no domínio afetado | Diário | Qualquer feedback F01 |
| M-12 taxa de aceitação | Semanal | < 55% |

### Dashboard de monitoramento pós-deploy

O Tech Lead configura um dashboard no Azure Monitor (ou equivalente) com:

```
Dashboard: novatech-assistant-post-deploy-{deploy_id}
Widgets:
  - Taxa de ❌ por hora (linha temporal, últimas 48h)
  - TTFT p95 por hora (linha temporal)
  - Contagem de feedbacks por categoria (F01-F07) por hora
  - Log de violações de guardrail (tabela)
  - Resultado dos casos critical do golden set (executados a cada hora automaticamente)
  - Comparação de métricas: janela atual vs. baseline pré-deploy
Alertas configurados:
  - Email + Teams para Tech Lead e PS em qualquer critério de rollback automático
  - Email para PS em qualquer alerta da janela alta
```

---

## 10. Critérios de Rollback

### 10.1 Rollback automático (sem aprovação humana necessária)

Ativado quando qualquer uma das condições abaixo for detectada. O Tech Lead é notificado simultaneamente ao rollback, mas não precisa aprovar — a reversão acontece automaticamente.

**Gatilhos automáticos nos primeiros 120 minutos:**

| Código | Condição | Fonte de detecção |
|---|---|---|
| `AUTO-01` | M-07 taxa de ❌ sobe > 3 pontos percentuais em relação ao baseline das últimas 24h, em qualela de 30 minutos | Azure Monitor (alerta configurado no deploy) |
| `AUTO-02` | TTFT p95 > 5s em janela de 15 minutos | Instrumentação do cliente |
| `AUTO-03` | Qualquer caso `critical` do golden dataset (GD-002, GD-007, GD-012, GD-013, GD-015) falha quando executado em produção | CI de verificação pós-deploy (executa a cada 30 min nas primeiras 2h) |
| `AUTO-04` | Validador de schema registra rejeição de resposta (violação de B-03 ou B-10) | Log do pipeline |
| `AUTO-05` | String de violação de guardrail detectada em resposta entregue ao atendente (ex: "7 dias" + "carga perigosa") | Detecção automática de strings proibidas |

### 10.2 Rollback por decisão humana (após 120 minutos ou por critérios abaixo)

Requer decisão conjunta de Tech Lead + Product Specialist. O Delivery Manager é notificado. Prazo para decisão: 2 horas após o alerta.

**Gatilhos por decisão humana:**

| Código | Condição | Detectado por |
|---|---|---|
| `HUMAN-01` | Violação de guardrail confirmada na amostragem manual (QA ou PS) — qualquer guardrail GR-01 a GR-17 | QA (amostragem) ou PS (revisão) |
| `HUMAN-02` | Taxa de alucinação > 1% na amostragem manual quinzenal (M-03) | QA (amostragem) |
| `HUMAN-03` | Regressão confirmada em caso `critical` do golden set que não foi detectada automaticamente | QA (execução manual) |
| `HUMAN-04` | Latência comprometendo o atendimento: TTFT p95 > 4s por 2 dias consecutivos (M-08) | Tech Lead (dashboard) |
| `HUMAN-05` | Atendentes reportam respostas perigosas ou inconsistentes via supervisor ou Compliance — mesmo que não capturadas pelo sistema de feedback | Supervisor de atendimento · Compliance |
| `HUMAN-06` | M-07 taxa de ❌ > 8% por 3 dias consecutivos (sem melhoria após investigação) | PS (dashboard diário) |
| `HUMAN-07` | Groundedness cai > 5pp em relação ao baseline (M-02) | QA (amostragem semanal) |
| `HUMAN-08` | Compliance identifica resposta com implicação regulatória (ANTT, LGPD) incorreta | Compliance (revisão periódica ou denúncia) |

### 10.3 Procedimento de rollback

**Passo 1 — Notificação imediata (0–5 minutos)**

```bash
# Notificação automática ou manual
POST https://hooks.microsoft.com/teams/novatech-alerts
{
  "text": "⚠️ ROLLBACK INICIADO — Assistente NovaTech\nGatilho: {codigo}\nDeploy: {deploy_id}\nIniciado por: {responsavel}\nTimestamp: {timestamp}"
}
```

O Tech Lead notifica o time de atendimento: "O assistente está sendo revertido para a versão anterior. Respostas podem ser temporariamente indisponíveis por até 5 minutos."

**Passo 2 — Reversão técnica (5–15 minutos)**

```bash
# Identificar a versão anterior no registro de configuração
git show HEAD~1:config/production.yaml > /tmp/rollback_config.yaml

# Extrair componentes a reverter
PREV_PROMPT_VERSION=$(cat /tmp/rollback_config.yaml | yq '.components.prompt.version')
PREV_INDEX_VERSION=$(cat /tmp/rollback_config.yaml | yq '.components.index.version')
PREV_MODEL_VERSION=$(cat /tmp/rollback_config.yaml | yq '.components.model.version')

# Reverter prompt
cp prompts/history/${PREV_PROMPT_VERSION}_*.txt prompts/system_prompt_prod.txt

# Reverter versão do índice no App Configuration
az appconfig kv set --name novatech-appconfig \
  --key PROD_INDEX_VERSION --value $PREV_INDEX_VERSION

# Reverter feature flags se necessário
# (flags alteradas no deploy problemático são revertidas ao estado anterior)

# Reiniciar o serviço
az functionapp restart --name novatech-assistant-api --resource-group novatech-rg
```

**Passo 3 — Validação do rollback (15–25 minutos)**

QA executa os 5 casos `critical` do golden dataset contra o endpoint de produção revertido:

```bash
npm run test:golden:critical --env=production
```

Todos os 5 devem PASSAR. Se qualquer um falhar, escalar imediatamente para Tech Lead — o rollback não resolveu o problema.

**Passo 4 — Registro (25–30 minutos)**

```json
// Log imutável no BC-06
{
  "rollback_id": "rb_20260607_001",
  "deploy_id_reverted": "dep_20260607_v1.2.1",
  "trigger_code": "AUTO-01",
  "trigger_description": "Taxa de ❌ subiu 4pp em 30 minutos após deploy",
  "version_reverted_to": {
    "prompt_version": "v1.2.0",
    "index_version": "idx_20260601_001",
    "model_version": "gpt-4o-2024-05-13"
  },
  "rolled_back_by": "tech_lead_01",
  "rollback_started_at": "2026-06-07T11:05:00Z",
  "rollback_completed_at": "2026-06-07T11:25:00Z",
  "validation_result": "all_critical_cases_passed",
  "item_id_for_investigation": "ITEM-143"
}
```

**Passo 5 — Comunicação pós-rollback**

- Tech Lead notifica o time de atendimento: "O assistente foi revertido para a versão anterior. Operação normal restaurada."
- Product Specialist notifica os aprovadores do deploy revertido: "O deploy [dep_id] foi revertido por [motivo]. Um novo item de investigação foi aberto ([ITEM-143])."
- Product Specialist abre item filhote com `root_cause_id` a ser determinado na investigação.

### 10.4 O que não é rollback

As seguintes situações não ativam rollback — são tratadas como itens de backlog:

| Situação | Tratamento correto |
|---|---|
| Um atendente marcou ❌ isolado sem padrão identificado | Triagem normal no fluxo de feedback |
| M-07 taxa de ❌ levemente acima do baseline por 1 dia (sem continuidade) | Monitorar por mais 2 dias antes de decidir |
| Groundedness caiu 2pp (abaixo do threshold de alerta de 5pp) | Registrar, monitorar, abrir item se tendência persistir |
| Atendente reportou resposta incompleta (não perigosa) sobre tema de baixo risco | Triagem normal — feedback `F03` |

---

## Resumo Operacional

| Prática | Responsável principal | Quando executar | Resultado |
|---|---|---|---|
| Staging | Tech Lead | Antes de qualquer candidato de deploy | Ambiente pronto para validação |
| Regression suite | QA (CI automático + manual) | A cada PR e a cada deploy candidato | Relatório de testes com resultado por dimensão |
| Grupo piloto | Tech Lead (roteamento) + PS (convocação) | Mudanças Alto e Crítico — 24–48h antes do rollout | Validação com 5 atendentes reais |
| Canary por percentual | Tech Lead | Mudanças de código de pipeline — 2h antes do rollout | 10% das requisições na versão candidata |
| Feature flags | Tech Lead + PS | Em todo deploy; flags de segurança requerem aprovação crítica | Ativação/desativação sem novo deploy |
| Versionamento de prompt | PS (criação) + repositório Git (histórico) | A cada mudança de prompt | Arquivo versionado + CHANGELOG |
| Versionamento de índice | Tech Lead | A cada ingestão ou atualização de metadados | Snapshot `idx_YYYYMMDD_NNN` |
| Versionamento de modelo | Tech Lead (configuração) | Apenas com aprovação MA-06 (Crítico) | `model_version` fixado na configuração |
| Registro de configuração | Pipeline de deploy (automático) | Após cada deploy bem-sucedido | `production.yaml` commitado no repositório |
| Monitoramento intensivo | Tech Lead + PS + QA | 14 dias após cada deploy | Relatório com `metric_deltas` e `outcome` |
| Rollback | Tech Lead (execução) + QA (validação) | Automático (gatilhos AUTO) ou por decisão humana (gatilhos HUMAN) | Produção na versão anterior validada + item filhote aberto |

---

*Documento elaborado com base no fluxo de promoção de alterações (novatech-harness-promocao.md), no harness de validação pré-produção (novatech-harness-validacao.md), na matriz de aprovação (novatech-harness-aprovacao.md) e nas métricas de qualidade (novatech-harness-metricas.md). Todos os comandos de rollback são exemplos indicativos — a implementação final deve seguir os padrões do repositório novatech-assistant e ser revisada pelo Tech Lead antes do go-live.*
