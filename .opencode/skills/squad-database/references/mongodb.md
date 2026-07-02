# MongoDB — referência do squad

## Modelagem: a pergunta que decide tudo

**Modele pelo padrão de acesso, não pela forma "normalizada" dos dados.** O que é lido junto, vive junto.

- **Embutir** (subdocumento/array): relação 1-poucos, dado lido sempre com o pai, sem acesso independente (`endereco` dentro de `usuario`).
- **Referenciar** (`user_id` em outra coleção): relação 1-muitos crescendo sem limite, dado acessado sozinho, ou compartilhado entre pais.
- Array que cresce sem limite dentro de documento é bug de modelagem (limite de 16 MB por documento e realocação cara) — vire coleção referenciada.
- Agregado do DDD ≈ documento: a fronteira de consistência natural é o documento (update de 1 documento é atômico). Se precisa de transação multi-documento com frequência, o desenho (ou a escolha do Mongo) merece revisão com o usuário.

## Schema e validação

- Mongo é schema-flexible, **não** schema-free: defina `$jsonSchema` validator por coleção (nível `moderate` para não travar documentos legados durante transição).
- Versione o schema no próprio documento (`schemaVersion: 2`) quando a forma mudar — a aplicação lê ambas as versões durante a transição.
- **"Migration" no Mongo = script de backfill versionado** (na pasta de migrations do projeto, com a mesma disciplina do relacional: imutável após aplicado, em lotes, idempotente/retomável).

## Fluxo: novo campo em documento existente

1. Campo opcional no schema da aplicação (código tolera ausência) → deploy.
2. Se o campo precisa existir em todos: script de backfill em lotes (`bulkWrite` com filtro `{campo: {$exists: false}}`, limite por lote).
3. Só então aperte o validator/`$jsonSchema` e o código para exigir o campo.

(É o expand → migrate → contract do relacional.)

## Índices e queries

- Índice composto segue a regra **ESR**: Equality primeiro, Sort depois, Range por último.
- Todo filtro recorrente tem índice; confirme com `.explain("executionStats")` — `COLLSCAN` em coleção grande é defeito.
- Índice único para chaves naturais (`email`) — a aplicação sozinha não garante unicidade sob concorrência.
- Índices custam RAM e escrita: crie por evidência, remova os não usados (`$indexStats`).
- Paginação: range query em campo indexado (`_id` ou timestamp), não `skip()` fundo.
- Agregation pipeline: `$match` e `$project` o mais cedo possível; `$lookup` recorrente em toda leitura é cheiro de modelagem relacional forçada no Mongo.

## Armadilhas

- Escreva com **write concern `majority`** para dado de negócio; leitura em secundário só aceitando dado potencialmente atrasado.
- Sem `multi: true`/`updateMany` explícito, update atinge 1 documento — e o inverso: `updateMany` sem filtro preciso é o `UPDATE sem WHERE` do Mongo (restrição geral do squad se aplica).
- Case-insensitive: use collation no índice, não regex `^.../i` (não usa índice de forma eficiente).
- Datas sempre como `Date` (BSON, UTC), nunca string ISO.
