# PostgreSQL — referência do squad

## Tipos recomendados

| Dado | Use | Evite |
|---|---|---|
| PK | `bigint GENERATED ALWAYS AS IDENTITY` ou `uuid` (v7 se disponível) | `serial` (legado) |
| Texto | `text` (sem limite artificial) ou `varchar(n)` quando o limite é regra de negócio | `char(n)` |
| Dinheiro | `numeric(19,4)` | `money`, `float` |
| Data/hora | `timestamptz` (sempre UTC) | `timestamp` sem timezone |
| Booleano | `boolean` | `smallint` 0/1 |
| JSON | `jsonb` (indexável) | `json` |
| Enum de domínio | coluna `text` + `CHECK` ou tabela de referência | tipo `ENUM` nativo (ALTER doloroso) |

## Padrões que importam

- **Índice sem lock**: `CREATE INDEX CONCURRENTLY` em tabela com tráfego (não roda dentro de transação — migration separada com `atomic off` na ferramenta).
- **`NOT NULL` em tabela grande**: adicionar coluna com `DEFAULT` constante é barato (PG 11+, sem rewrite); mudar coluna existente para NOT NULL exige validação — use `CHECK ... NOT VALID` + `VALIDATE CONSTRAINT` em passo separado.
- Busca textual: `ILIKE 'x%'` usa índice com `text_pattern_ops`; busca ampla → `pg_trgm` (GIN) ou full-text search.
- FK sempre com índice na coluna referenciadora (PG **não** cria automaticamente).
- Paginação de tela funda: keyset (`WHERE (col, id) > (?, ?) ORDER BY col, id`) em vez de `OFFSET` gigante.
- `EXPLAIN (ANALYZE, BUFFERS)` antes de qualquer conclusão sobre performance.

## Armadilhas

- Identificador sem aspas vira minúsculo: nunca crie objetos com aspas/maiúsculas (`"Users"` ≠ `users`).
- `count(*)` em tabela grande é caro por design (MVCC) — para contadores de UI, aproxime ou materialize.
- Transação longa segura o vacuum e infla bloat — migrations de backfill em lotes (`UPDATE ... WHERE id BETWEEN`), commit por lote.
- Conexões são caras: pool (PgBouncer) desde cedo em app com muitas instâncias.
