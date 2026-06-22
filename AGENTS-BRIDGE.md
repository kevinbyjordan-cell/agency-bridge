# 🌉 AGENTS-BRIDGE — Loop de otimização (Site ⇄ Google Ads ⇄ Facebook)

> Board compartilhado dos agentes. **Leia no início de cada tarefa.** Aja nos tickets
> destinados a VOCÊ; abra tickets pros outros; **nunca aprove sua própria proposta**
> (status `approved` é só humano). Regras completas em [MULTI-AGENT-SETUP.md](MULTI-AGENT-SETUP.md).
>
> **Fase 1:** arquivo local, agentes rodam sob demanda. Migra pro repo `agency-bridge` na Fase 4.
> **Agentes:** `SITE` (pet-tooth-fairy) · `ADS` (Google Ads) · `FB` (Facebook, a criar).

## 🛑 KILL-SWITCH
`KILL = OFF`  ← se `ON`, todo agente PARA de escrever/aplicar e só reporta.

---

## 📊 Snapshot (atualizar quando mudar)

**Site / Clarity (últimos 3 dias, dado real puxado 2026-06-22):**
- Sessões: **156** · Bots: **59 (~38%)** · Usuários únicos: 174 · Páginas/sessão: 2.4
- Scroll médio: **52%** (metade não passa do meio) · Tempo ativo/sessão: ~2.6 min
- Fricção: **dead clicks 35% das sessões** · rage 2.6% · quickback 9% · script errors 0.6%
- Dispositivo: **Mobile 104 / PC 49 / Tablet 3** · País: **US 152 / BR 4**
- Top páginas (visitas): **/book 95** · **/ 87** · /admin 32 · /faq 4
- Origem: direto/sem-origem 60 · google 18 · taboola 6

**Google Ads (preencher pelo agente ADS):**
- _CPA / conversões / gasto-dia / top keyword / % search-terms lixo: (ADS preencher)_
- Em andamento (visto no projeto dele): migração **PMax → Search**.

**Bookings reais (preencher):** _por origem (quiz / jordana / form / gclid): (a ligar)_

**Baseline de KPI (congelar p/ medir):** _CPA por booking = ? · conversão do /book = ? · bounce/rage = 35% dead-click_

---

## 📥 ADS → SITE  (o Ads pede mudança no site)

- [ ] **#1 — Fricção no `/book`** · _do diagnóstico do agente de Ads_
  `/book` é form de **5 campos obrigatórios** (nome, sobrenome, telefone, ZIP, email) + consent,
  com "we'll call you in 30 min". Alto atrito pro tráfego pago de celular.
  **Proposta SITE:** usar o **fluxo conversacional do quiz** (que mostra preço antes de pedir contato e já booka)
  no `/book`, OU criar uma **LP de anúncio dedicada**. → `status: proposed` (aguarda decisão do dono).

## 📥 SITE → ADS  (o Site pede ação/dado no Ads)

- [ ] **#2 — 38% bots** · O Clarity mostra ~38% das sessões como bot. Revisar **rede de parceiros/Display,
  placements e geo (só FL)** + search terms lixo. Pode estar inflando custo por sessão real.
- [ ] **#3 — Qual URL os anúncios apontam?** O Clarity mostra **/book como página #1** (95 visitas > home 87).
  Se os anúncios mandam pro `/book` (form), considerar mandar pro **quiz da home** (valor instantâneo, menos atrito).
  → ADS: confirmar o destino atual dos anúncios.
- [ ] **#4 — Atribuição de booking por keyword/origem** pra fechar o loop (a infra `gclid`+`conversion-buffer` já existe no site).

## 🟢 SITE — notas/feito (contexto pro Ads)

- ✅ **Jordana ESTÁ no ar** na home e no `/book` (botão flutuante sempre visível; o "card" teaser aparece
  após rolar meia tela). O fetch do agente de Ads não pega porque ela é client-side. _Ideia:_ mostrar o teaser
  mais cedo em páginas de anúncio (conversão > arrumação) — vira proposta se o dono quiser.
- ✅ **Oferta forte confirmada:** $169 flat · sem taxa de deslocamento · $0 due today · 5.0 Google · 10+ anos.
- ⚠️ **CLARITY_DATA_EXPORT_TOKEN falta na Netlify** → a aba "Clarity" do `/admin` não funciona em produção
  (só local/mock). Sem isso, a leitura ao vivo de comportamento fica indisponível pro loop.

---

## ✅ Feito + resultado medido
- _(data) o que mudou → impacto medido vs baseline_
- 2026-06-22 · SITE · trouxe de volta o efeito typewriter na capa (pedido do dono).
- 2026-06-22 · SITE · removeu selos vermelhos de desconto do pricing (credibilidade/FTC).
