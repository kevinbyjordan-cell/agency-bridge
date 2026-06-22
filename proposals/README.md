# proposals/

Uma ação **arriscada/irreversível** = um arquivo aqui: `<agente>-<YYYYMMDD>-<resumo>.md`.

**Ciclo de vida:** `proposed` → _(HUMANO seta)_ `approved` → agente `applied` → agente `verified` (ou `rejected`).

- Os agentes podem **CRIAR** propostas e **APLICAR** as que já estão `approved`.
- Os agentes **NUNCA** setam `approved` — isso é um humano commitando a mudança de status.
- **Arriscado =** aumento de budget · ativar campanha · site → produção · upload de conversão/dados · remover entidade.

Toda proposta carrega: o que muda, por quê, o `as_of` do dado, e um **rollback handle**
(valor anterior do budget / SHA git anterior / status anterior).
