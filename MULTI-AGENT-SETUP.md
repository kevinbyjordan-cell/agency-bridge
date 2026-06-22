# 🤖 Multi-Agent Setup — Otimização contínua (Site · Google Ads · Facebook)

> **Status:** desenho / Fase 1. Documento vivo. Escrito pelo agente do **Site**.
> **Objetivo:** 2–3 agentes especialistas que otimizam o negócio em **ciclos
> agendados** (não em chat ligado 24h), coordenando por **estado compartilhado**,
> com **freios de segurança** porque mexem em **site de produção + budget de anúncio**.

---

## 1. Os agentes

| Agente | Pasta | Especialidade | Ferramentas |
|---|---|---|---|
| **Site** (eu) | `pet-tooth-fairy/` | Front-end, CRO, conteúdo, Clarity | Next.js/Netlify/Firebase, git |
| **Google Ads** | `GOOGLE ADS PRO/` | Campanhas, keywords, negativas, budget | `gads-cli` (Google Ads API) |
| **Facebook Ads** | _(a criar)_ | Campanhas Meta | _(a definir — Meta API)_ |

Cada um é **dono da sua área** e **só escreve no que é seu**. O Facebook entra
depois, plugando na mesma estrutura — nada precisa ser refeito.

---

## 2. Como "conversam" (o conceito que importa)

**NÃO é** três IAs num chat ligado 24h (caro, frágil, descontrolado).
**É:** cada agente **acorda num horário (cron), lê o estado compartilhado, faz seu
trabalho, escreve o resultado, dorme.** A comunicação acontece pelo **estado
compartilhado** (o "bridge"), não por um socket sempre aberto. Isso é mais barato,
sobrevive a reboot, dá pra auditar, e é como sistemas multi-agente sérios funcionam.

```
        ┌──────────────── BRIDGE (estado compartilhado, versionado) ────────────────┐
        │  state/site.json · state/gads.json · state/fbads.json  (cada um só o seu)  │
        │  proposals/ (ledger de propostas)  ·  decisions/ (log append-only)         │
        │  KILL flag  ·  metrics snapshot  ·  heartbeats                             │
        └──────┬──────────────────────┬──────────────────────┬──────────────────────┘
        ┌──────┴──────┐        ┌───────┴───────┐       ┌──────┴───────┐
        │  SITE (eu)  │        │  GOOGLE ADS   │       │ FACEBOOK ADS │
        └─────────────┘        └───────────────┘       └──────(depois)┘
              ▲ cron/cloud-routine acorda cada um por ciclo (1x/dia por padrão) ▲
```

---

## 3. Hospedagem: Claude Code Cloud Routines

Escolha do dono: rodar nas **rotinas-cloud do Claude Code** (cron gerenciado pela
Anthropic, **sem a máquina do dono ligada**). Implicações **importantes**:

### Limitações (precisa desenhar em volta delas)
- **Cada run é efêmero e isolado.** Sobe um ambiente novo, **clona UM repositório**,
  roda, e some — **sem disco persistente** e **sem ver as pastas irmãs**. ⇒ um arquivo
  local na pasta-mãe **não existe** em produção; o agente de Ads não enxerga a pasta do Site.
- **Headless.** Não tem humano no meio do run pra aprovar nada. **MCP que dependem de
  login OAuth interativo podem não existir** (ex.: dashboard do Clarity) — só sobrevive
  o que usa **API key / refresh token**.
- **Periódico, não contínuo.** É cron (granularidade do cron), e cada run é **limitado
  em tempo/turnos**.
- **Cobrança por uso de tokens** do modelo, por run. (Verificar limites exatos de
  duração/concorrência/preço no plano — não invento números aqui.)

### O que isso obriga (decisões de arquitetura)
1. **O bridge tem que ser um repo git dedicado** (`agency-bridge`, com remote próprio),
   que **toda rotina clona junto** com o repo dela. **Nunca** um arquivo só local.
2. **Segredos entram pelo cofre da rotina-cloud** (injetados 1x pelo dono, fora do git).
3. **Fontes que exigem OAuth interativo saem do caminho headless** — usar a versão
   API-key (ex.: `CLARITY_DATA_EXPORT_TOKEN` no lugar do dashboard).

> **Fase 1 (agora):** o bridge é um arquivo local (`AGENTS-BRIDGE.md`) e os agentes
> rodam **sob demanda** (você dispara). **Fase 4 (cloud):** promover o bridge pro repo
> `agency-bridge` e ligar as rotinas. (ver §6).

---

## 4. Os freios 🛡️ (NÃO-NEGOCIÁVEIS — vieram de um red-team de 5 frentes)

> Resumo dos 32 modos de falha encontrados (24 high/critical). Estes são os freios
> que **têm que existir antes** de qualquer autonomia real.

### 4.1 Site — nunca deployar em produção sozinho
- ❌ **Proibido `git push origin main`** no modo autônomo (a main auto-deploya na Netlify).
- ✅ Agente **commita numa branch + abre PR** → Netlify gera **Deploy Preview** → **humano dá merge**.
- ✅ Check obrigatório no PR: `next build`/`tsc` verdes; **`NEXT_PUBLIC_MOCK_MODE !== 'false'` reprova**;
  diff que toca `firestore.rules`/`storage.rules`/`netlify.toml` exige revisão.
- ✅ Credencial git da rotina = **só push de branch + abrir PR**, sem permissão na main.

### 4.2 Google Ads — gastar é a direção irreversível
- ✅ **Teto por cliente** (`max_daily_budget_usd`) + **cap mensal de conta** + **cap de delta
  por passo** (rejeitar aumento > X% ou > $Y), **dentro do `gads-cli`** antes do mutate.
  _(Hoje o único cap é `MAX_DAILY_USD = $100k`, que é anti-typo, não limite de negócio.)_
- ✅ **Aumento de budget = gated por humano**; só **redução** passa sem gate.
- ✅ **`campaign expand` tem que criar PAUSED** (hoje cria **ENABLED** → abre gasto vivo sozinho).
- ✅ **`REMOVE` nunca autônomo** (tirar `--confirm-remove` do caminho da rotina).
- ✅ Antes de pausar/ativar: **re-buscar métrica viva no mesmo run** e confirmar a precondição;
  recusar decisão em cima de dado mais velho que o TTL.

### 4.3 Aprovação — ledger de propostas, humano é a única chave
- Toda ação **arriscada/irreversível** (budget↑, ativar campanha, push do site pra prod,
  upload de conversão, remover) entra como **`status: proposed`**. **Só o HUMANO** muda pra
  `approved` (campo que os agentes são **proibidos** de setar).
- Ações **seguras/reversíveis** (negativa de keyword, pausar SKAG claramente desperdiçando,
  copy atrás de flag) podem ir **proposed → applied** sozinhas.

### 4.4 Kill-switch + disjuntor independente
- **Flag `KILL` no bridge** que **todo agente checa antes de escrever** e aborta se ligada.
- **Cron separado, sem modelo**, que lê o **gasto vivo** e **pausa campanhas** se passar do teto —
  **não depende** dos agentes de otimização estarem saudáveis.
- **Teto mensal de tokens** no nível de billing + alerta antes do limite.

### 4.5 Segredos & headless
- **Nenhum segredo no git.** Entram só pelo **cofre da rotina-cloud** (1x, fora do repo).
- **Least-privilege:** conta de serviço Google restrita ao **cliente `7493280264`** (não a MCC toda);
  token Netlify **só deploy**; chave Firebase **só servidor**. Cada um **revogável** sozinho.
- **Fail-CLOSED:** sem credencial → **erro duro + 'BLOCKED' no bridge + exit≠0**. Nunca cair em
  no-op silencioso nem em **mock mode** (`NEXT_PUBLIC_MOCK_MODE=false` forçado e checado no boot).
- **Pre-push secret-scanner** (gitleaks) bloqueia commit com cara de credencial. **Nunca `git add .`** —
  só caminhos explícitos.
- **Sem segredo em log/transcript/bridge** (são inspecionáveis).

### 4.6 Estado consistente (bridge)
- **Cada agente escreve só o arquivo dele**; decisões = **arquivos novos append-only** (um por
  escrita, nome com UUID/timestamp) → **rebase sempre limpo, zero conflito**.
- **`pull --rebase` → push**, com retry; **nunca `--force`**. Locks **com lease/expiry, por recurso**
  (não global, não booleano) — run morto não trava a frota.
- **Reconciliar** estado do bridge vs **sistema vivo** (gads API / Netlify) no início do run;
  na divergência, **o vivo manda**; **verificar** o apply relendo a fonte; **anti-flapping**
  (congela alvo trocado em sentidos opostos em K runs).

### 4.7 Observabilidade & humano no loop
- **Heartbeat:** todo run grava `run-started` e `run-finished`. **Faltou o `run-finished`** dentro
  da janela esperada = **avisa o humano** (silêncio ≠ "tá tudo bem").
- **Digest diário empurrado** (push/email, fora do localhost) — o que cada agente fez, o que espera
  aprovação, o que travou, e o **delta de KPI vs baseline**. (O dashboard local `agency-hq` fica
  cego com a máquina desligada.)
- **Baseline + guard band de KPI:** toda mudança referencia o KPI que mira; se o KPI piorar além da
  banda, **congela a área e escala** — o loop não pode "otimizar" enquanto o CPA/booking piora.
- **Cooldown + amostra mínima:** não mexer num alvo mudado nos últimos N dias; não decidir em cima de
  amostra abaixo do piso (sessões pro Clarity, conversões pro Ads) — senão otimiza ruído.

### 4.8 Conflito entre agentes
- Mudança que afeta a superfície de outro agente (intenção do anúncio ↔ copy da página) = **proposta
  ACKada pelo dono** antes de subir. Um **"message map"** (oferta/intenção canônica por página) que os
  dois leem como fonte da verdade.
- **Tickets** têm dono + estado + contador de tentativas + TTL; **auto-escalam** pro humano depois de
  K idas-e-vindas; `wontfix` é **sticky** (não re-abre).

### 4.9 Custo do run
- **Cadência padrão: 1x/dia** pra agente que mexe em gasto (depois do dado assentar, ~08:00 ET);
  sub-diário só pra **check read-only**. Separar **"observar"** (barato/frequente) de **"agir"** (diário/gated).
- **Teto de tokens/turnos por run** + **early-exit "nada a fazer"** (conta parada custa ≈ zero).
- **Modelo por tarefa:** trabalho mecânico no Node (sem modelo); modelo barato pra triagem; topo só
  pra a decisão de verdade.

---

## 5. Custo (modelo mental honesto)

Dois custos **separados**:
1. **Tokens** = `Σ (tokens por run × frequência × nº de agentes)`. Alavancas: cadência (diário),
   escopo por run (lookback curto + early-exit), tier de modelo, e o early-exit de conta parada.
   ⇒ comece **1x/dia**, meça o gasto, e só aumente a frequência onde dá lucro.
2. **Budget de anúncio** = controlado pelos **tetos de §4.2/§4.4**, independente dos tokens.

_(Preço exato por run depende do plano/modelo — verificar na doc do Claude Code. Não cravo número.)_

---

## 6. Rollout em fases

- **Fase 1 (hoje):** `AGENTS-BRIDGE.md` local + agentes **sob demanda** (você dispara). Valida o loop.
- **Fase 2:** **1 ciclo/dia agendado** — cada agente gera relatório + propostas no bridge; você revisa de manhã.
- **Fase 3:** liberar **autonomia nas ações seguras** (§4.3); gate humano nas arriscadas. Implementar os
  freios de código (PR-not-push no site; caps + expand-PAUSED no `gads-cli`).
- **Fase 4 (cloud):** promover o bridge pro repo **`agency-bridge`**; segredos no cofre da rotina;
  ligar as cron-routines; heartbeat + digest + disjuntor.

---

## 7. Tarefas de setup abertas (checklist)

**Antes de qualquer autonomia (Fase 3+):**
- [ ] **Site:** trocar `push main` por **branch + PR**; credencial git sem acesso à main; check de `MOCK_MODE`.
- [ ] **Ads (`gads-cli`):** tetos por cliente + cap de delta + `expand` cria PAUSED + tirar `--confirm-remove` do caminho autônomo.
- [ ] **Bridge:** criar repo `agency-bridge`; mover o board pra lá; estado por-agente + decisions append-only.
- [ ] **Segredos:** cofre da rotina-cloud (least-privilege); `CLARITY_DATA_EXPORT_TOKEN` e `CONVERSION_SYNC_TOKEN` single-source; gitleaks pre-push.
- [ ] **Kill-switch + disjuntor** de gasto (cron separado).
- [ ] **Heartbeat + digest diário** empurrado (push/email).
- [ ] **Wiring:** instrução no `CLAUDE.md` de cada agente: "no início de cada tarefa, leia o bridge; aja nos tickets seus; escreva os seus; nunca aprove suas próprias propostas."
