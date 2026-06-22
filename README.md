# agency-bridge

Estado de coordenação compartilhado dos agentes especialistas da agência
(**Site** · **Google Ads** · **Facebook** _depois_). Este repo é a **fonte única**
que eles leem/escrevem a cada ciclo — cada rotina-cloud clona este repo junto com o dela.

- **[AGENTS-BRIDGE.md](AGENTS-BRIDGE.md)** — o board vivo (snapshot, tickets, propostas, feito). **Comece aqui.**
- **[MULTI-AGENT-SETUP.md](MULTI-AGENT-SETUP.md)** — arquitetura + os freios de segurança não-negociáveis.
- **KILL** — parada de emergência. Se contiver `ON`, **todo agente para** (não escreve, não aplica).
- **state/** — um arquivo por agente; cada agente escreve **só o seu**.
- **proposals/** — ações arriscadas/irreversíveis como propostas; **só humano** muda pra `approved`.
- **decisions/** — log append-only; **um arquivo novo por escrita** (nunca edita no lugar).

## Regras (todo agente)
1. Leia `AGENTS-BRIDGE.md` e cheque o `KILL` **antes de tudo**. Se `KILL=ON`, pare.
2. Aja nos tickets endereçados a você. Abra tickets pros outros. Atualize **só o seu** `state/<agente>.json`.
3. **Nunca aprove sua própria proposta** — `approved` é transição **só humana**.
4. `pull --rebase` antes de escrever; **nunca force-push**. Não use `git add .` (só caminhos explícitos).
