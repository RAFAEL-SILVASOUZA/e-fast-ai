# Oracle — referência do squad

## Tipos recomendados

| Dado | Use | Evite |
|---|---|---|
| PK | `NUMBER(19) GENERATED ALWAYS AS IDENTITY` (12c+) | sequence+trigger manual em projeto novo |
| Texto | `VARCHAR2(n CHAR)` — **CHAR semantics**, senão o limite é em bytes e acentos estouram | `CHAR`, `LONG` |
| Texto longo | `CLOB` | `LONG` (deprecado) |
| Dinheiro | `NUMBER(19,4)` | `BINARY_FLOAT/DOUBLE` |
| Data/hora | `TIMESTAMP WITH TIME ZONE` ou `TIMESTAMP` armazenando UTC por convenção documentada | `DATE` quando precisar de fração de segundo |
| Booleano | `NUMBER(1)` + `CHECK (col IN (0,1))` (23c tem `BOOLEAN` nativo — confirme a versão) | strings 'S'/'N' em projeto novo |
| JSON | `JSON` (21c+) ou `CLOB` + `IS JSON` check | — |

## Padrões que importam

- **String vazia é NULL** no Oracle (`'' IS NULL` é verdadeiro). Isso muda semântica de `NOT NULL`, comparações e o comportamento do ORM — trate no design, não descubra em produção.
- Identificadores viram MAIÚSCULOS sem aspas; **nunca** crie objetos com aspas (`"users"`). Limite de 30 caracteres para nomes antes do 12.2 — nomes de índice/constraint longos quebram.
- Índice online: `CREATE INDEX ... ONLINE` para não bloquear DML.
- `NOT NULL` com `DEFAULT` em tabela grande é otimizado (11g+: default é metadado, sem rewrite).
- Paginação: `OFFSET n ROWS FETCH NEXT m ROWS ONLY` (12c+); nada de `ROWNUM` aninhado em código novo.
- FK sem índice na coluna referenciadora causa **lock de tabela** no delete/update do pai — sempre indexe FKs.
- Plano de execução: `EXPLAIN PLAN FOR ...` + `DBMS_XPLAN.DISPLAY`, ou `DBMS_XPLAN.DISPLAY_CURSOR` para o plano real.

## Armadilhas

- `DATE` sempre tem componente de hora — comparação com `TRUNC(data)` mata índice; use range (`>= :d AND < :d+1`).
- Bind variables sempre (ORMs já fazem): literal em query repetida enche o shared pool e força hard parse.
- Transações: Oracle não tem autocommit no servidor — DDL faz commit implícito (migration mista de DDL+DML não é atômica; separe).
- Case-sensitive por padrão em comparação de strings; padronize com constraint/normalização na escrita, não com `UPPER()` na query (ou crie function-based index).
