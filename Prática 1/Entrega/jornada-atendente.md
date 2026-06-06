# Jornada do Atendente — Assistente de IA NovaTech
**Elaborado por:** Product Specialist  
**Data:** 05/06/2026  
**Contexto:** Assistente de IA integrado ao Microsoft Teams + SharePoint, destinado aos 45 atendentes da NovaTech para consulta em linguagem natural à documentação oficial durante chamados ativos.

---

## 1. Visão Geral da Jornada

O assistente não substitui o julgamento do atendente nem toma decisões por ele. Seu papel é reduzir o tempo de busca em documentação (meta: de 12 para menos de 2 minutos por chamado), entregar a informação correta com rastreabilidade de fonte e sinalizar ativamente quando não há resposta confiável disponível.

A jornada é composta por três fluxos operacionais e um conjunto de guardrails que governam o comportamento do assistente em todas as situações.

---

## 2. Fluxo Principal — Consulta com Resposta Confiante

**Contexto de ativação:** O atendente está em um chamado ativo e precisa de uma informação que normalmente buscaria em uma ou mais das fontes documentais (SharePoint, Confluence, planilhas de referência).

---

**Passo 1 — Recebimento da dúvida do cliente**

O atendente identifica que precisa de uma informação para responder ao cliente. A dúvida pode ser sobre prazo de entrega, regra de frete, política de devolução, SLA aplicável, desconto, seguro ou outro tema coberto pela documentação interna.

O atendente pode colocar o cliente em espera breve ("vou verificar isso para o senhor") ou, se a conversa for por escrito, simplesmente consultar o assistente em paralelo.

---

**Passo 2 — Formulação da pergunta ao assistente**

O atendente digita a dúvida em linguagem natural no painel do assistente, integrado ao Teams. Não é necessário saber o nome do documento ou o número do procedimento.

Exemplos de perguntas válidas:
- "Qual o prazo para devolução de mercadoria após a entrega?"
- "Qual o multiplicador regional para frete especial com destino ao Norte?"
- "Cliente com 9 fretes especiais por mês tem direito a desconto automático?"
- "Qual o SLA de resolução para cliente Gold em chamado crítico?"

---

**Passo 3 — Processamento e resposta do assistente**

O assistente busca a resposta na base documental indexada (SharePoint + Confluence + planilhas) e retorna uma resposta estruturada contendo obrigatoriamente:

- **Resposta direta** à pergunta, em linguagem clara e objetiva.
- **Fonte citada**, com nome do documento, versão e seção de referência (ex: `POL-001 v3.1 — Seção 3.1`).
- **Data da última atualização** do documento fonte.
- **Nível de confiança** da resposta, expresso de forma qualitativa: `Alta confiança`, `Confiança moderada` ou `Baixa confiança / ver fallback`.
- **Alerta contextual**, quando aplicável — por exemplo, quando o tema consultado possui inconsistência conhecida na base documental ou quando o documento fonte está sinalizado como em revisão.

Exemplo de resposta bem formada:

> **Resposta:** O prazo para devolução de mercadoria é de até 7 dias úteis após a data de recebimento confirmada no sistema de tracking. Sábados, domingos e feriados nacionais não são contados.
>
> **Fonte:** POL-001 — Política de Devolução de Mercadorias, v3.1, Seção 3.1 (atualizada em 15/01/2024).
>
> **Confiança:** Alta.
>
> **Atenção:** Esta regra não se aplica a cargas perigosas, cargas refrigeradas com ruptura de cadeia de frio ou cargas com lacre violado. Para esses casos, oriente o cliente a contatar o ramal 4500 (Gestão de Riscos).

---

**Passo 4 — Uso da resposta no atendimento**

O atendente avalia a resposta, considera o contexto específico do chamado e utiliza a informação para responder ao cliente. O atendente retém o julgamento final — o assistente é uma ferramenta de consulta, não uma autoridade decisória.

Se a resposta fizer sentido no contexto do chamado, o atendente a utiliza diretamente. Se houver dúvida sobre aplicabilidade ao caso específico, o atendente pode fazer uma pergunta de seguimento ao assistente antes de responder ao cliente.

---

## 3. Fluxo de Fallback — Resposta Incerta ou Ausente

**Contexto de ativação:** O assistente não encontra resposta com confiança suficiente na base documental, ou o atendente recebe a resposta mas discorda dela com base em experiência prévia ou informação contraditória que conhece.

Esse fluxo também se aplica quando o tema consultado se enquadra em uma das categorias de risco identificadas na análise documental: documentos em revisão (PROC-043), versões coexistentes sem hierarquia formal (PROC-042 v1 e v2), ou temas cobertos apenas pelo FAQ informal sem normativo de respaldo.

---

**Situação A — O assistente não tem resposta confiante**

O assistente sinaliza explicitamente a limitação, sem inventar ou inferir informação não documentada. A resposta retornada segue este padrão:

> **Resposta:** Não foi encontrada resposta com confiança suficiente para esta pergunta na base documental disponível.
>
> **Motivo:** [uma das opções abaixo, conforme o caso]
> - O tema não está coberto por nenhum documento indexado.
> - O documento referente a este tema está sinalizado como em revisão (PROC-043) e pode estar desatualizado.
> - Existem dois documentos com regras divergentes sobre este tema sem indicação formal de qual prevalece (PROC-042 v1 e v2). Consulte a seção de inconsistências conhecidas.
> - A única fonte disponível é o FAQ-Atendimento, que é um documento informal não validado por Compliance.
>
> **Próximo passo sugerido:** Escale para o supervisor ou consulte diretamente a área responsável ([Operações / Comercial / Compliance / Gestão de Riscos], conforme o tema).

O atendente nunca deve comunicar ao cliente uma informação que o assistente marcou como de baixa confiança sem antes obter validação de uma fonte humana.

---

**Situação B — O atendente discorda da resposta**

Se o atendente recebe uma resposta marcada como de alta ou moderada confiança, mas identifica que ela contradiz informação que conhece (por experiência, por orientação recebida recentemente, ou por acesso a outro documento), o fluxo é:

1. O atendente **não utiliza a resposta** diretamente no atendimento ao cliente.
2. O atendente aciona a opção **"Discordo desta resposta"** no painel do assistente (ver Fluxo de Feedback).
3. O atendente coloca o chamado em espera se necessário e consulta o supervisor ou a área responsável para obter a versão correta.
4. Após resolução, o atendente registra a resposta correta via feedback para atualização da base.

A discordância do atendente é tratada como sinal de possível desatualização da base documental — não como erro do atendente. O sistema não penaliza o uso do fallback; ao contrário, registra a ocorrência para análise de qualidade da base.

---

**Situação C — Tema coberto apenas pelo FAQ informal**

Quando a única fonte disponível para um tema é o FAQ-Atendimento (documento informal, não validado por Compliance ou Operações), o assistente retorna a resposta com marcação explícita:

> **Resposta:** [conteúdo da resposta]
>
> **Fonte:** FAQ-Atendimento — documento informal, não validado por Compliance. Versão não controlada.
>
> **⚠️ Atenção:** Esta resposta está baseada em conhecimento tácito do time de atendimento e não possui respaldo em documento normativo formal. Use com cautela. Temas como seguro de carga (item 22), carga danificada em trânsito (item 38) e cargas perigosas com frete expresso (item 32) se enquadram nesta categoria.

O atendente decide se utiliza a orientação como referência geral ou se escala para validação antes de responder ao cliente.

---

## 4. Fluxo de Feedback — Sinalização de Erro, Desatualização ou Incompletude

**Contexto de ativação:** O atendente identifica, em qualquer momento do uso, que uma resposta do assistente está errada, desatualizada, incompleta ou baseada em documento que foi substituído.

O feedback é o mecanismo central de melhoria contínua da base documental e da qualidade do assistente. Sem ele, inconsistências como as identificadas na PROC-042 (v1 coexistindo com v2 sem hierarquia) ou no FAQ (regras híbridas não documentadas) continuarão sendo reproduzidas.

---

**Passo 1 — Sinalização imediata no painel**

Após receber qualquer resposta, o atendente tem acesso a três ações rápidas:

- **✅ Resposta útil** — confirma que a resposta foi usada com sucesso.
- **⚠️ Resposta incompleta ou imprecisa** — usado quando a resposta é parcialmente correta mas falta contexto, detalhe ou condição relevante.
- **❌ Resposta incorreta ou desatualizada** — usado quando a resposta está errada ou baseada em informação que o atendente sabe ter sido alterada.

Qualquer sinalização negativa (⚠️ ou ❌) abre automaticamente um formulário de detalhamento.

---

**Passo 2 — Preenchimento do formulário de feedback**

O formulário solicita:

- **Tipo de problema** (seleção): Informação desatualizada / Documento errado usado como fonte / Resposta conflita com outro documento / Resposta incompleta — faltou condição ou exceção / Outro.
- **Descrição livre** (campo de texto, opcional): o atendente pode descrever o que estava correto ou indicar a fonte correta se souber.
- **Número do chamado** (preenchido automaticamente pelo sistema): permite correlacionar o feedback com o contexto operacional.

O preenchimento deve ser rápido — o objetivo é não interromper o atendimento. O campo de descrição livre é opcional para não criar fricção.

---

**Passo 3 — Roteamento e tratamento do feedback**

O feedback é encaminhado automaticamente conforme o tema identificado:

| Tema do feedback | Área receptora |
|---|---|
| Frete especial / multiplicadores / descontos | Diretoria Comercial |
| Política de devolução / carga danificada | Diretoria de Operações |
| SLA / tiers de clientes | Diretoria Comercial + Operações |
| Cargas perigosas / compliance | Compliance |
| FAQ (qualquer tema) | Coordenação de Atendimento |

Feedbacks marcados como ❌ em temas de risco alto (cargas perigosas, divergência entre versões de PROC) são escalados com prioridade para o gestor responsável pela base documental.

---

**Passo 4 — Ciclo de atualização da base**

A área receptora tem prazo definido para avaliar o feedback e, se procedente, atualizar o documento fonte ou registrar formalmente a versão vigente. Após a atualização, o documento é reindexado pelo assistente e o atendente que sinalizou o problema recebe notificação de encerramento do ciclo.

Feedbacks acumulados sobre o mesmo tema sem resolução em prazo definido geram alerta para a gestão de qualidade documental — tornando visível o padrão identificado na análise: documentos que só são corrigidos quando alguém "pergunta para quem sabe".

---

## 5. Guardrails de Comportamento do Assistente

Os guardrails são restrições e obrigações de comportamento do assistente que não podem ser desativadas por configuração nem contornadas por instruções do usuário. Eles derivam diretamente dos riscos identificados na análise documental da NovaTech.

---

### Guardrail 1 — Nunca afirmar prazo, valor ou condição que não esteja explicitamente documentado

**Regra:** O assistente não infere, estima, arredonda nem interpola informações numéricas ou temporais. Se o documento não contém o valor exato para a combinação consultada, o assistente declara a ausência da informação e sugere escalada.

**Motivação:** A análise identificou que prazos divergem entre versões do mesmo documento (PROC-042 v1: +2 dias úteis; v2: +3 dias úteis) e que percentuais de seguro existem apenas no FAQ informal sem respaldo normativo. Uma resposta inventada ou estimada nessas áreas gera comprometimento com o cliente sem respaldo contratual.

**Comportamento esperado:**
- ✅ "O prazo adicional para frete especial é de +3 dias úteis, conforme PROC-042-v2, Seção 3."
- ✅ "Não há informação documentada sobre o prazo para este tipo de carga. Recomendo consultar a área de Operações."
- ❌ "O prazo deve ser em torno de 3 a 4 dias, dependendo da rota." *(inferência não documentada — proibido)*
- ❌ "Pelo que consta nos documentos, provavelmente o desconto seria de aproximadamente 5%." *(uso de linguagem de estimativa — proibido)*

---

### Guardrail 2 — Sempre sinalizar quando existem versões conflitantes de um mesmo documento

**Regra:** Quando o tema consultado é coberto por mais de um documento com regras divergentes e sem hierarquia formal estabelecida, o assistente obrigatoriamente inclui um alerta na resposta — mesmo que uma versão seja claramente mais recente. O assistente pode indicar a versão mais recente como referência preferencial, mas não pode suprimir a existência do conflito.

**Motivação:** A coexistência da PROC-042 v1 e v2 no SharePoint sem status formal é o risco mais alto identificado na análise. O FAQ-Atendimento já construiu uma regra híbrida incorreta (limiar de desconto da v1 com automaticidade da v2) precisamente porque o conflito entre versões nunca foi sinalizado sistematicamente. Um assistente que silencia sobre o conflito replicaria o mesmo problema.

**Comportamento esperado:**
- ✅ "Conforme PROC-042-v2 (nov/2023), o desconto automático começa a partir de 8 fretes/mês. ⚠️ Atenção: existe uma versão anterior do mesmo documento (PROC-042-v1, mar/2023) com regra diferente (>10 fretes/mês, sem desconto automático), ainda ativa no SharePoint. A v2 é a mais recente, mas não há declaração formal de revogação da v1. Use a v2 e registre feedback se o contrato do cliente indicar versão diferente."
- ❌ "O desconto começa a partir de 8 fretes por mês." *(omite o conflito existente — proibido)*

---

### Guardrail 3 — Nunca apresentar orientação do FAQ como equivalente a normativo formal

**Regra:** Respostas baseadas exclusivamente no FAQ-Atendimento devem ser sempre marcadas com identificação de fonte informal e nível de confiança moderado ou baixo. O assistente não pode equiparar o FAQ a documentos das categorias POL (política), PROC (procedimento) ou SLA (contratual) na hierarquia de confiabilidade da resposta.

**Motivação:** O FAQ é a única fonte para temas críticos como seguro de carga, carga danificada em trânsito e frete expresso para cargas perigosas — todos sem normativo formal. A análise identificou que o FAQ opera como "normativo de fato" nessas áreas sem ter sido concebido para esse papel e sem passar por validação. Apresentar essas respostas com o mesmo peso de uma POL-001 v3.1 induziria o atendente a tomar decisões de alto risco com falsa segurança.

**Comportamento esperado:**
- ✅ "Segundo o FAQ-Atendimento (documento informal, não validado por Compliance), o seguro de carga é de 0,3% do valor declarado para cargas padrão. ⚠️ Não existe normativo formal sobre seguro de carga na base documental. Confirme com o Comercial antes de informar ao cliente."
- ❌ "O seguro de carga é de 0,3% do valor declarado." *(omite a origem informal e o risco — proibido)*

---

### Guardrail 4 — Nunca recomendar escalada para área ou contato que não esteja documentado na base oficial

**Regra:** Quando o assistente sugere que o atendente escale uma questão, o destino da escalada (área, ramal, e-mail) deve estar referenciado em um documento formal da base. O assistente não pode recomendar contatos baseados apenas no FAQ informal se não houver confirmação em normativo.

**Motivação:** O FAQ menciona o e-mail `sinistros@novatech.com.br` para carga danificada (item 38) e o ramal 4500 para Gestão de Riscos (item 3), mas nenhum desses contatos está formalizado em documento normativo do conjunto analisado. Contatos informais mudam sem atualização do FAQ, e o assistente que os reproduz propaga informação potencialmente desatualizada.

**Comportamento esperado:**
- ✅ "Para devolução de carga perigosa, oriente o cliente a contatar o setor de Gestão de Riscos, conforme POL-001 v3.1, Seção 3.2. O contato específico deve ser confirmado com o supervisor, pois não está formalizado no documento."
- ❌ "Encaminhe para sinistros@novatech.com.br." *(contato não formalizado em normativo — proibido sem confirmação de fonte)*

---

## 6. Resumo dos Fluxos

```
CHAMADO ATIVO
     │
     ▼
[Atendente formula pergunta ao assistente]
     │
     ▼
[Assistente busca na base documental]
     │
     ├─── RESPOSTA ENCONTRADA com alta/moderada confiança ──────────────────────────┐
     │         │                                                                     │
     │         ▼                                                                     │
     │    [Assistente retorna: resposta + fonte + versão + data + alertas]          │
     │         │                                                                     │
     │         ├─── Atendente CONCORDA → usa no atendimento → sinaliza ✅          │
     │         │                                                                     │
     │         └─── Atendente DISCORDA → aciona ❌ → fallback B → escala           │
     │                                                                               │
     ├─── RESPOSTA com fonte APENAS do FAQ informal ─────────────────────────────── ┤
     │         │                                                                     │
     │         ▼                                                                     │
     │    [Assistente retorna com marcação ⚠️ FONTE INFORMAL]                       │
     │         │                                                                     │
     │         └─── Atendente decide: usa com cautela OU escala para validação      │
     │                                                                               │
     └─── RESPOSTA NÃO ENCONTRADA / BAIXA CONFIANÇA ────────────────────────────── ┘
               │
               ▼
          [Assistente declara limitação + motivo + próximo passo sugerido]
               │
               ▼
          [Atendente escala para supervisor ou área responsável]
               │
               ▼
          [Após resolução: atendente registra feedback com resposta correta]
               │
               ▼
          [Área responsável atualiza documento → base reindexada → ciclo fechado]
```

---

## 7. Relação entre Jornada e Riscos Documentais Mapeados

| Risco identificado na análise | Mecanismo de mitigação na jornada |
|---|---|
| PROC-042 v1 e v2 coexistentes sem hierarquia (IC-01 a IC-04, Gap 2) | Guardrail 2 — alerta obrigatório de conflito em toda resposta sobre frete especial |
| FAQ com regras híbridas incorretas (CRZ-01, CRZ-03) | Guardrail 3 — FAQ nunca apresentado como normativo; Guardrail 1 — sem inferência de valores |
| Temas cobertos só pelo FAQ sem normativo (CRZ-05 a CRZ-08) | Fallback C — resposta com marcação explícita de fonte informal |
| PROC-043 em revisão pelo Compliance (IC-05, CRZ-05) | Fallback A — assistente sinaliza documento em revisão como motivo de baixa confiança |
| Contatos informais no FAQ (ramal 4500, sinistros@) | Guardrail 4 — contatos não formalizados não são reproduzidos sem ressalva |
| Atendente hoje "pergunta para quem sabe" sem registro (cenário) | Fluxo de feedback — escalada gera registro, roteamento e ciclo de atualização da base |
| 15% dos chamados escalam sem resposta (cenário) | Fallback A estruturado — escalada com contexto documentado, não "pergunta em branco" |

---

*Documento elaborado com base no cenário operacional da NovaTech, nas análises de inconsistências e gaps da base documental, e no cruzamento entre práticas informais do FAQ e normativos vigentes. Os fluxos descritos são uma proposta de design de produto — implementação requer validação com as equipes de Atendimento, Compliance, Operações e TI.*
