# DDD — Sumário de padrões (base: "DDD Reference" de Eric Evans, CC BY 4.0, tradução PT)

Cobre a linguagem de padrões completa da Referência: definições, modelo em ação, blocos táticos, design flexível, mapeamento de contexto, destilação e estrutura em larga escala. O **como implementar** cada padrão (regras de agregados, arquiteturas, eventos, repositórios, integração) está em [implementing-ddd.md](implementing-ddd.md) — leia os dois juntos.

## Definições centrais

- **Domínio**: a esfera de conhecimento/atividade à qual o software se aplica.
- **Modelo**: sistema de abstrações que descreve aspectos selecionados do domínio para resolver problemas dele.
- **Linguagem Onipresente (Ubiquitous Language)**: linguagem estruturada em torno do modelo, usada por TODO o time (negócio e dev) dentro de um contexto delimitado. Na prática: o nome da classe é o nome que o negócio usa. Se o negócio diz "Apólice", a classe não se chama `Contract`.
- **Contexto Delimitado (Bounded Context)**: fronteira (subsistema, time) dentro da qual um modelo se aplica e os termos têm significado único. "Cliente" no faturamento ≠ "Cliente" no suporte — e está tudo bem, desde que sejam contextos separados.

## Colocando o modelo em ação

- **Design Dirigido por Modelos**: o código EXPRESSA o modelo — se o modelo muda, o código muda, e vice-versa. Modelo que só vive em diagrama é decoração.
- **Modeladores Envolvidos (hands-on)**: quem modela programa, quem programa modela. Modelo entregue "de cima" para devs executarem perde o vínculo com o código.
- **Integração Contínua (sentido DDD)**: dentro de um contexto, merge frequente + testes + exercício constante da Linguagem Onipresente, para o modelo não fragmentar entre cabeças.
- **Refatorando para uma Visão Mais Profunda**: quando um conceito escondido do domínio aparece (o "estalo"), refatore o modelo para expressá-lo — as maiores simplificações vêm daí, não de limpeza cosmética.

## Padrões táticos (dentro de um contexto)

| Padrão | Definição operacional |
|---|---|
| **Entidade** | Objeto definido por identidade e continuidade, não por atributos (`Pedido #123` continua o mesmo se o endereço mudar) |
| **Objeto de Valor** | Definido só pelos atributos, imutável, sem identidade (`CPF`, `Money`, `Endereco`). Prefira-o a tipos primitivos soltos |
| **Agregado** | Grupo de entidades/VOs tratado como unidade de consistência, com uma **raiz**: referências externas só à raiz; transação não cruza fronteira de agregado |
| **Repositório** | Acesso a agregados com ilusão de coleção em memória; um repositório por raiz de agregado, nunca por tabela |
| **Serviço de Domínio** | Operação de domínio que não pertence naturalmente a entidade/VO (ex.: transferência entre duas contas). Sem estado |
| **Evento de Domínio** | Fato relevante para o negócio, no passado (`PedidoPago`). Base para integração entre contextos e para event sourcing |
| **Fábrica** | Criação de agregados complexos garantindo invariantes desde o nascimento |
| **Módulos** | Pacotes que contam a história do domínio (por conceito de negócio), com baixo acoplamento entre si — não por tipo técnico (`controllers/`, `helpers/`) |
| **Arquitetura em Camadas** | Isole a expressão do modelo: lógica de domínio não depende de UI, persistência ou infraestrutura |

## Design Flexível (supple design)

Qualidades que mantêm o modelo agradável de evoluir (sobrepõem-se aos princípios de `../../squad/references/clean-code.md` — lá está o "como" no código):

- **Interfaces Reveladoras de Intenção**: nome diz o efeito e o propósito, na Linguagem Onipresente — quem usa não precisa ler a implementação.
- **Funções Livres de Efeitos Colaterais**: o máximo da lógica em funções que só retornam valor; comandos (que mutam estado) separados e simples.
- **Asserções**: pós-condições e invariantes explícitas (asserts/testes de unidade) — o efeito de uma operação é garantido, não adivinhado.
- **Classes Autônomas**: reduza dependências ao ponto de a classe ser compreensível sozinha.
- **Fechamento de Operações**: quando couber, operação retorna o mesmo tipo dos argumentos (`Money + Money = Money`) — composição sem tipos estranhos.
- **Contornos Conceituais**: decomponha pelo desenho natural do domínio; se refatorações sucessivas "cortam" sempre no mesmo lugar, o contorno está certo.
- **Design Declarativo / Formalismos Estabelecidos**: quando um trecho do domínio tem formalismo pronto (matemática monetária, conjuntos, gramáticas), apoie-se nele em vez de inventar semântica.

## Padrões estratégicos (entre contextos) — Mapa de Contexto

Relações possíveis entre dois contextos delimitados:

- **Parceria**: dois contextos que sucedem ou falham juntos — coordenação de planejamento e integração conjunta.
- **Núcleo Compartilhado (Shared Kernel)**: subconjunto do modelo compartilhado por dois times — mudanças exigem acordo mútuo. Use pouco.
- **Cliente/Fornecedor**: contexto downstream é "cliente" do upstream; prioridades negociadas.
- **Conformista**: downstream adota o modelo do upstream como está (quando não há poder de negociação).
- **Camada Anticorrupção (ACL)**: camada de tradução que protege seu modelo de um modelo externo/legado. **O padrão mais útil ao integrar com sistemas legados ou de terceiros.**
- **Serviço de Anfitrião Aberto / Linguagem Publicada**: protocolo público (ex.: API REST + OpenAPI) para muitos consumidores.
- **Caminhos Separados**: os contextos simplesmente não se integram — às vezes a integração não vale o custo.
- **Grande Bola de Lama**: reconheça a região do sistema onde modelos estão misturados sem fronteira; desenhe um limite ao redor dela e **não tente aplicar modelagem sofisticada dentro** — contenha, não conserte tudo.

## Destilação

- **Domínio Central (Core Domain)**: a parte do modelo que diferencia o negócio — onde vai o melhor esforço e o melhor código.
- **Subdomínios Genéricos**: necessários mas não diferenciais (autenticação, e-mail, boleto) — compre, use lib ou faça o mínimo.
- **Declaração da Visão de Domínio**: ~1 página dizendo o que é o domínio central e por que vale a pena — escrita cedo, revisada sempre.
- **Núcleo Destacado**: marque/descreva o core no próprio repositório (docs curtos, anotações) para todo dev saber onde está o que importa.
- **Mecanismos Coesos**: separe "computação complicada" (grafo, cálculo, motor de regras) em mecanismo próprio; o domínio expressa O QUE, o mecanismo resolve COMO.
- **Núcleo Segregado / Núcleo Abstrato**: refatore para o core ficar em módulo próprio, desacoplado do suporte; em sistemas grandes, o núcleo pode virar abstrações puras com implementações nos módulos.

## Estrutura em Larga Escala (sistemas grandes, vários times)

Só quando uma estrutura "cair naturalmente": **Ordem de Evolução** (a estrutura macro também evolui, não é congelada), **Metáfora de Sistema** (só se realmente iluminar), **Camadas de Responsabilidade** (ex.: operacional × política × decisão), **Nível de Conhecimento** (configuração que descreve regras sobre o modelo), **Componentes Plugáveis** (só com domínio muito maduro). Estrutura imposta cedo demais engessa mais do que ajuda.

## Como o squad usa isto

1. **Módulos do monolito modular = contextos delimitados candidatos.** Nomeie módulos pela linguagem do negócio.
2. **Uma migration/transação por agregado**: se a operação precisa alterar dois agregados atomicamente, questione o desenho.
3. **Camada anticorrupção sempre que integrar legado** — nunca deixe o modelo do legado vazar para o domínio novo.
4. **Objetos de Valor primeiro**: `Email`, `Cpf`, `Money` como tipos, não `string`/`decimal` soltos — validação nasce no construtor.
5. Nem todo sistema precisa de DDD tático completo. CRUD simples com Linguagem Onipresente bem aplicada já é 80% do valor.
