# decisions/

Log **append-only** (auditoria). **Um arquivo NOVO por escrita:** `<agente>-<YYYYMMDDTHHMMSS>-<uuid>.md`.

Nunca edite um arquivo existente (mantém o rebase do git sem conflito). Registre: o que mudou,
por quê, o `as_of` do dado em que se baseou, e o **rollback handle**. É assim que uma ação
fica idempotente (um run que repetiu/sobrepôs não aplica de novo o que já está logado aqui).
