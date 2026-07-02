# SQL Server — referência do squad

## Tipos recomendados

| Dado | Use | Evite |
|---|---|---|
| PK | `bigint IDENTITY(1,1)` ou `uniqueidentifier` (com `NEWSEQUENTIALID()`/UUIDv7 para não fragmentar clustered index) | `NEWID()` como clustered PK |
| Texto | `nvarchar(n)` (Unicode); `nvarchar(max)` só para conteúdo realmente longo | `text`/`ntext` (deprecados), `varchar` para dados com acento |
| Dinheiro | `decimal(19,4)` | `money` (arredondamento), `float` |
| Data/hora | `datetime2(3)` ou `datetimeoffset` (sempre UTC) | `datetime` (precisão de 3ms, range menor) |
| Booleano | `bit` | — |
| JSON | `nvarchar(max)` + `ISJSON()` check (tipo `json` nativo no 2025+ — confirme a versão) | — |

## Padrões que importam

- **Índice online**: `CREATE INDEX ... WITH (ONLINE = ON)` — só Enterprise/Azure SQL; confirme a edição antes de prometer zero lock.
- `NOT NULL` com `DEFAULT` constante em tabela grande é operação de metadado (2012+) — barato.
- FK **não** ganha índice automático — crie na coluna referenciadora.
- Paginação: `ORDER BY ... OFFSET n ROWS FETCH NEXT m ROWS ONLY`; para telas fundas, keyset.
- Collation define case-sensitivity (padrão geralmente CI): comparações são case-insensitive — não "conserte" com `UPPER()`, e cuidado ao portar código de outro engine.
- Plano real: `SET STATISTICS IO, TIME ON` + plano de execução real; `sys.dm_exec_query_stats` para os piores ofensores.

## Armadilhas

- **Parameter sniffing**: procedure/query parametrizada com plano ruim para parâmetro atípico — conheça `OPTION (RECOMPILE)` antes de culpar o índice.
- **Lock escalation e deadlocks**: transação que atualiza muitas linhas escala para lock de tabela; backfill em lotes (`UPDATE TOP (n) ... WHERE ...` em loop com commit).
- `READ COMMITTED` padrão bloqueia leitores contra escritores — avalie `READ_COMMITTED_SNAPSHOT ON` no início do projeto (mudar depois exige janela).
- `nvarchar` vs `varchar` em join/where com tipos diferentes causa conversão implícita e mata índice — consistência de tipo entre coluna e parâmetro.
- Trigger com lógica de negócio é dívida instantânea — regra vive na aplicação (restrição geral do squad).
