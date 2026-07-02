# Implementing DDD — guia prescritivo (base: "Implementing Domain-Driven Design", Vaughn Vernon)

Complementa [ddd.md](ddd.md) (linguagem de padrões de Evans) com o **como implementar**: regras de bolso, arquiteturas e decisões de código. Onde Evans diz "depende", Vernon dá o padrão para tentar primeiro — é essa a régua do squad.

## Espaço do problema × espaço da solução

- **Espaço do problema**: domínio e subdomínios — o que o negócio precisa. **Espaço da solução**: contextos delimitados — os modelos de software que respondem a isso. Avalie os dois antes de desenhar: subdomínio e contexto alinhados 1:1 é o ideal.
- Três tipos de subdomínio: **Central (Core)** — diferencia o negócio, recebe o melhor time e DDD tático completo; **de Suporte (Supporting)** — necessário e específico do negócio, mas não diferencial: modele o suficiente, sem sofisticação; **Genérico** — resolvido no mercado (auth, e-mail, boleto): compre ou use lib.
- Um contexto delimitado por time, um repositório/deploy por contexto (candidato). Contexto não é componente técnico: "camada de serviços" ou "o frontend" não são contextos.
- **Grande Bola de Lama existente**: desenhe a fronteira no Mapa de Contexto e integre via camada anticorrupção — não tente "arrumar por dentro" enquanto extrai o modelo novo.

## Arquiteturas — DDD não exige uma; escolha pelo requisito

| Estilo | Quando | Nota de implementação |
|---|---|---|
| **Camadas + DIP** | ponto de partida universal | Infraestrutura implementa interfaces definidas no domínio; dependências apontam para o domínio, nunca o contrário |
| **Hexagonal (Ports & Adapters)** | padrão recomendado como evolução natural das camadas | Aplicação+domínio no centro; HTTP, banco, mensageria são adapters plugados em ports. Teste de aplicação roda sem nenhum adapter real |
| **REST** | expor o contexto a consumidores | Recursos REST ≠ modelo de domínio: o contrato é uma projeção; mudar o modelo não pode quebrar o contrato silenciosamente |
| **CQRS** | leituras com forma muito diferente das escritas, ou escala assimétrica | Comando muta agregado; consulta lê um modelo de leitura próprio (view/projeção), sem passar por repositório de agregado. UI deve tolerar a defasagem (consistência eventual) |
| **Arquitetura dirigida a eventos** | integração entre contextos, reação a fatos | Pipes & filters, e **processos de longa duração (Sagas)** para coordenar múltiplos agregados/contextos: máquina de estado + rastreador de timeout, nunca transação distribuída |
| **Event Sourcing** | auditoria total, reconstrução de estado, análise temporal | Agregado persiste como sequência de eventos; estado atual = replay. Custo alto de infraestrutura — só com justificativa real |

## Entidades — identidade e validação

- Quatro estratégias de geração de identidade: **o usuário fornece** (placa, ISBN — validar unicidade), **a aplicação gera** (UUID/ULID — a preferida: disponível antes de persistir, sem ida ao banco), **a persistência gera** (sequence/identity — cuidado: identidade tardia complica igualdade e publicação de eventos antes do flush), **outro contexto fornece** (sincronização necessária).
- **Identidade de domínio ≠ chave técnica**: exponha a identidade de negócio (VO tipado, ex.: `TenantId`); a chave substituta do ORM (long autoincremento) fica escondida na infraestrutura.
- Identidade é estável: setter de ID privado/inexistente após criação.
- Validação em três níveis: **atributo/propriedade** (autovalidação no setter/construtor — guard clauses), **objeto inteiro** (invariantes entre atributos — método `validate()` ou validador separado que devolve TODAS as violações, não só a primeira), **composição de objetos** (mais de um agregado — validador em serviço de domínio, chamado quando o caso de uso exige).
- Construção segura: construtor/fábrica garante instância válida desde o nascimento; nada de objeto "meio pronto" esperando setters.

## Objetos de Valor — prefira-os a entidades

Teste de VO (todas as respostas "sim" → é VO): mede/quantifica/descreve algo do domínio? Imutável? Conceito completo (todo conceitual — `Money` = valor+moeda, nunca `decimal` solto)? Substituível por outro de mesmo valor? Comparado por igualdade de valor? Comportamento livre de efeito colateral?

- **Minimalismo na integração**: ao consumir dado de outro contexto, traduza para VO local com só o que este contexto usa — não replique a entidade alheia.
- **Tipos padrão como VOs**: status/tipo como VO ou enum rico com comportamento, não string mágica.
- Persistência: VO é detalhe do agregado — serialize na mesma tabela (owned type/embeddable/JSON) sempre que puder; tabela separada para coleção de VOs é decisão de ORM, não do modelo (o VO continua sem identidade no domínio).

## Serviços de Domínio

- Só quando a operação **não cabe** em entidade ou VO (cálculo que cruza agregados, política que precisa de repositório para decidir). Sem estado; nomeado pela Linguagem Onipresente.
- **Anti-padrão**: "mini-camada" de serviços que rouba todo o comportamento das entidades → modelo anêmico. Primeiro tente colocar o comportamento na entidade/VO; serviço é exceção justificada.
- Serviço de domínio ≠ Application Service (ver "Aplicação").

## Eventos de Domínio

- Publique um evento para **todo fato que o negócio nomeia no passado** (`PedidoPago`, `SprintCommitted`) — especialmente quando outro agregado ou contexto precisa reagir.
- Modelagem: nome na Linguagem Onipresente no passado; **imutável**; carrega identidade dos agregados envolvidos, timestamp e os dados que os consumidores precisam (enriqueça conscientemente: evento magro força o consumidor a consultar de volta; evento gordo acopla).
- Publicação: publisher leve (observer) dentro do contexto; o handler no mesmo contexto pode modificar **outro** agregado em **outra** transação (consistência eventual).
- **Event Store + encaminhamento**: salve o evento na MESMA transação que o agregado (outbox/event store) e encaminhe depois — publicar direto no meio da transação perde ou duplica eventos. Dois estilos de entrega para outros contextos: **notification log RESTful** (consumidor pergunta; simples, pull, sem middleware) ou **mensageria** (RabbitMQ/Kafka; push, exige idempotência e dedup no consumidor).
- Consumidor é responsável pela própria consistência: dedup por ID do evento, retry, tolerância a reordenação.

## Módulos

- Nomeie pelo domínio (`com.acme.billing.invoice`), não por tipo técnico (`helpers`, `managers`). Módulo conta a história do modelo.
- Acoplamento acíclico entre módulos; módulo novo antes de contexto novo — não crie um bounded context para o que é só um módulo do mesmo modelo.

## Agregados — as 4 regras de bolso (o coração do livro)

1. **Modele invariantes verdadeiros na fronteira de consistência.** Agregado = fronteira transacional. Só entra no agregado o que PRECISA ser consistente na mesma transação. "Seria conveniente navegar" não é invariante.
2. **Desenhe agregados pequenos.** Comece com raiz + VOs; cada entidade a mais exige justificativa por invariante. Agregado-cluster gigante = falha de concorrência (usuários disputando o mesmo lock) e memória desperdiçada.
3. **Referencie outros agregados por identidade**, não por objeto (`ProductId`, não `Product`). Isola a fronteira, barateia o carregamento, viabiliza distribuição/sharding.
4. **Fora da fronteira, consistência eventual.** Precisa atualizar dois agregados? UMA transação modifica UM agregado; a segunda mudança acontece via evento de domínio + handler em outra transação. Pergunta-guia: **de quem é o job?** Se é o usuário desta transação que exige o dado consistente AGORA, repense a fronteira; se é outro usuário/o sistema, consistência eventual está certa.

Razões legítimas para quebrar as regras (documente em ADR): conveniência de UI para operação em lote, falta de mecanismo de mensageria/eventos no projeto, transação global imposta (legado), performance de consulta. Exceção justificada ≠ regra ignorada.

Implementação: raiz com identidade única global; partes internas com identidade local se entidades — mas **prefira VOs como partes**; "Tell, Don't Ask" e Lei de Deméter (cliente chama método de negócio da raiz, nunca navega `raiz.getFilhos().get(0).set...`); **concorrência otimista** (campo `version`) protegendo o invariante da raiz; **sem injeção de dependência para dentro do agregado** — repositório/serviço não entram no agregado; passe o dado (ou serviço como parâmetro de método) quando necessário.

## Fábricas

- Fábrica só quando a criação é expressiva demais para um construtor. Melhor lugar: **método-fábrica na raiz do agregado** que cria outro elemento/agregado dentro das regras (`forum.startDiscussion(...)` — garante invariantes e passa identidades corretas).
- Fábrica em serviço de domínio quando a criação exige colaboração de outro contexto (tradução de tipos alheios).

## Repositórios

- **Um repositório por raiz de agregado** — dá acesso ao agregado INTEIRO, nunca a pedaços.
- Dois estilos: **orientado a coleção** (simula `Set` em memória: `add`, sem `save` — mudanças rastreadas pela unidade de trabalho do ORM; EF Core/Hibernate) e **orientado a persistência** (`save` explícito a cada mudança; MongoDB, stores sem unit of work — a escolha do banco dita o estilo, e trocar de estilo depois DÓI: decida cedo).
- Interface do repositório no domínio, implementação na infraestrutura (DIP). Assinaturas na Linguagem Onipresente (`nextIdentity()`, `ofTenant(...)`).
- **Repositório ≠ DAO**: DAO expõe tabela/linhas; repositório expõe agregados. Nada de `update(campo, valor)` genérico.
- **Consultas ótimas por caso de uso**: tela que precisa de dados de N agregados não carrega N agregados — use consulta dedicada que devolve DTO/projeção (caminho natural para CQRS). Repositório de agregado serve a ESCRITA.
- Teste com implementação em memória do MESMO contrato de interface + teste de integração da implementação real (mapeamento, SQL e migration só se provam no banco real).

## Integração entre Contextos delimitados

- Sistemas distribuídos são fundamentalmente diferentes: rede falha, latência varia, consistência é eventual — desenhe para isso, não finja chamada local.
- **Sempre por camada anticorrupção no consumidor**: traduza o modelo publicado do outro contexto para tipos locais (VOs mínimos) na borda — o modelo alheio NUNCA viaja para dentro do domínio.
- **Via REST**: consumidor implementa cliente + tradutor (ACL). O produtor publica um contrato estável (Linguagem Publicada, ex.: OpenAPI) — media types/DTOs versionados, não entidades serializadas.
- **Via mensageria**: consumidor assina eventos de domínio do produtor e mantém o próprio estado derivado. Obrigatório: idempotência no handler, dedup por ID de evento, e pensar "e se a mensageria cair?" (retry, dead letter, catch-up pelo notification log).
- **Processo de longa duração (Saga)**: operação que cruza contextos = máquina de estados explícita (agregado próprio que rastreia o progresso) + timeout tracker para os passos que não respondem — nunca two-phase commit entre contextos.

## Aplicação (a camada em volta do domínio)

- **Application Service = caso de uso, FINO**: controla transação e segurança, busca o agregado no repositório, chama UM método de negócio, persiste, publica eventos. **Zero regra de negócio** — se tem `if` de negócio no application service, a regra escapou do domínio.
- Uma transação por caso de uso, um agregado modificado por transação (regra 4 dos agregados).
- Entrada/saída por DTOs ou tipos primitivos — entidade de domínio não sai pela API. Alternativas de rendição: DTO montado do agregado (padrão), mediator/double-dispatch, ou payload dedicado por caso de uso.
- UI/controller nunca fala com repositório ou entidade direto — sempre pelo application service.
- Composição de múltiplos contextos para uma tela é papel da aplicação/UI (API composition), não do domínio.

## Como o squad usa isto

1. **As 4 regras de agregado são régua de revisão**: transação tocando dois agregados, referência a agregado alheio por objeto, agregado com coleção que só cresce — apontamento de arquitetura.
2. **Application service gordo é o cheiro nº 1** em código "com DDD": regra de negócio em service = mover para entidade/VO.
3. **Todo evento entre contextos passa por outbox/notification log** — nada de publicar direto de dentro da transação.
4. Config `arquitetura: clean-architecture | hexagonal` → estrutura hexagonal deste guia; `monolito-modular` → módulos = contextos candidatos com comunicação por interface pública ou evento.
5. CQRS e Event Sourcing são opt-in por ADR — nunca "de brinde".
