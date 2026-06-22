# Proposta SITE-20260622 — Mandar o tráfego pago pro quiz (não pro `/book`) + matar o dead-click da capa

- **Agente:** SITE (pet-tooth-fairy)
- **Status:** `applied` — **aprovado pelo dono no chat (06-22)** → implementado como **PR petoothfairy-site#1**
  (https://github.com/petoothfairy-png/petoothfairy-site/pull/1). ⏳ aguarda **merge humano** (vira `verified` no deploy).
  _Parte A (CTAs→quiz) + Parte B (dead-click da capa) ambas no PR._
- **Responde aos tickets:** `ADS → SITE #1` (fricção no /book) e `ADS → SITE #5` (0 conversão / dead-click 35%)
- **as_of do dado:** Clarity 3d puxado 2026-06-22 (/book = página #1 com 95 visitas; scroll 52%; dead-click 35%; mobile 104/156). Ads: 14 cliques de alta intenção, 0 booking, 2 dias.
- **Risco:** mudança no site → produção (reversível). Por isso é proposta, não auto-aplicada.

## Descoberta-chave (antes de mexer em nada)
O fluxo conversacional "mostra preço antes de pedir contato e booka inline" que o agente de Ads
propôs **construir** no `/book` **já existe e está no ar**: é o **quiz da home** (`QuizSection` →
`PriceCalculator`). Ele cota em **3 toques**, sem pedir e-mail pra ver o preço, e fecha o booking
**na mesma tela** (`book-name → … → book-done`), com "$0 due today".

**O furo do funil:** todo CTA primário aponta pro `/book` (form de 5 campos), **pulando o quiz**:
- `sections/Hero.tsx:90` — botão primário "Book My Pet's Cleaning" → `href="/book"`.
- `layout/StickyMobileCTA.tsx:66` — barra fixa do mobile "Book Now" → `href="/book"`.

Ou seja: o tráfego pago de celular (alta intenção) cai na superfície de **maior atrito**, enquanto
o ativo que mais converte fica abaixo da dobra. Isso bate com "/book é a página #1 mas 0 booking".

---

## Parte A — Repontar os CTAs primários pro quiz  _(o lever grande, barato e reversível)_
**O que muda:**
- `Hero.tsx:90` → trocar `href="/book"` por `href="/#quote"` (o quiz já tem `id="quote"` em `QuizSection.tsx:11`).
- `StickyMobileCTA.tsx:66` → trocar `href="/book"` por `href="/#quote"`.
- Manter o `/book` vivo como fallback ("Prefer the full form?" já existe no resultado do quiz).

**Por quê:** leva o clique pago pro fluxo de menor atrito (preço instantâneo, sem e-mail pra ver,
booking inline) em vez do form longo. Zero código novo — só re-rota.

**Rollback handle:** restaurar `href="/book"` nas 2 linhas (SHA git anterior ao merge do PR).

**Como medir:** instrumentar `referralSource` por origem (quiz vs form) — já gravado no booking — e
comparar conversão /book-direto × quiz nos 7 dias seguintes. Hoje baseline = 0 conv, então qualquer
booking pago atribuído já é sinal.

## Parte B — Matar o dead-click 35% da capa  _(elementos que fingem ser clicáveis)_
Candidatos achados no código da Hero (todos com afeto de "clicável" — `hover:scale` / cara de chip —
mas **sem link nem handler**):
1. **Imagem da capa** (`Hero.tsx:149`): `hover:scale-[1.02]` + glow no hover, é `<Image>` puro. Grande,
   central, acima da dobra → forte candidato a dead-click no toque mobile.
2. **Stat cards** (`Hero.tsx:181`, `hover:scale-105`): "100% Anesthesia-Free", "5.0★ Google Rating",
   "10+ Years". O de **Google Rating** especialmente convida o toque (esperam ir pros reviews).
3. **Badge flutuante** (`Hero.tsx:163`): "⭐⭐⭐⭐⭐ Loved by Florida pet parents" — parece chip de review, não é.

**O que muda (2 opções por item — escolher 1):**
- (preferida) **tornar real o que finge ser clicável:** imagem da capa → `Link` pro `/#quote`;
  stat "Google Rating" + badge flutuante → `Link` pro perfil do Google (já temos `googleReviewsHref`).
- (alternativa) **remover o afeto falso:** tirar `hover:scale*` desses elementos pra não prometerem toque.

**Rollback handle:** restaurar os elementos como estão hoje (div sem link / sem `Link` wrapper).

---

## Fora de escopo desta proposta (anotado, não pedido agora)
- **LP dedicada `/lp` quiz-first** pro anúncio apontar direto: maior esforço **e** exige o agente de Ads
  trocar os `final_urls` (cross-domain). Se a Parte A não bastar, abro proposta separada + ticket SITE→ADS.
- Implementação só entra **depois do seu `approved`** e via **PR + preview** (nunca push direto na main).
