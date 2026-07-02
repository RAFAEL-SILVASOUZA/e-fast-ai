# Código Limpo — base compartilhada do squad

Resumo operacional dos princípios de *Código Limpo* (Robert C. Martin) que **todo especialista do squad aplica ao escrever ou revisar código**.

## Nomes

- Nomes revelam intenção: `diasDesdeUltimoAcesso`, nunca `d`. Se precisa de comentário para explicar o nome, o nome está errado.
- Classes = substantivos (`Invoice`, `UserRepository`); métodos = verbos (`calculateTotal`, `findByEmail`).
- Um conceito, uma palavra: não misture `fetch`, `get` e `retrieve` para a mesma operação no mesmo código.
- Sem desinformação: `accountList` só se for de fato uma lista; sem números mágicos — extraia constante nomeada.
- Pronunciável e buscável. Evite prefixos húngaros e abreviações criativas.

## Funções

- **Pequenas.** Fazem UMA coisa, em um nível de abstração. Se dá para extrair uma função com nome significativo de dentro dela, extraia.
- Poucos parâmetros: 0–2 ideal; 3 exige justificativa; mais que isso, agrupe em objeto (`CriteriosDeBusca` em vez de 5 parâmetros).
- Sem flag boolean como parâmetro (`render(true)`) — divida em duas funções.
- Sem efeitos colaterais escondidos: `checkPassword()` não inicializa sessão.
- Separe comando de consulta: ou a função faz algo, ou responde algo — nunca ambos.

## Comentários

- Comentário bom explica **por quê** (decisão, restrição, trade-off), nunca **o quê** (o código já diz).
- Código comentado é lixo: delete — o git lembra.
- Não escreva comentários redundantes, changelog em comentário, ou marcadores de seção. Prefira refatorar até o comentário ficar desnecessário.

## Formatação

- O arquivo lê como artigo de jornal: o alto nível no topo, o detalhe descendo. Quem lê de cima para baixo entende a história sem pular.
- Afinidade vertical: variável declarada perto do primeiro uso; função chamada logo abaixo de quem a chama; linhas em branco separam conceitos, a ausência delas agrupa.
- A regra do time vence a preferência pessoal — formatador automático configurado no projeto encerra a discussão; nunca reformate arquivo inteiro em commit de feature.

## Tratamento de erros

- Exceções em vez de códigos de retorno. Exceções específicas do domínio (`SaldoInsuficienteException`), não genéricas.
- Nunca retorne `null` nem passe `null`: use Optional/nullable types/coleção vazia.
- Não engula exceção (`catch` vazio). Trate, enriqueça com contexto e relance, ou deixe subir.
- O try/catch delimita uma transação conceitual — extraia o corpo para função própria.

## Classes e design

- **SRP**: uma classe, uma razão para mudar. Descreva a classe em uma frase sem usar "e"/"ou".
- Coesão alta: métodos usam os campos da classe. Muitos campos usados por poucos métodos = classe querendo se dividir.
- **Lei de Deméter**: fale só com amigos diretos — `a.getB().getC().faz()` é acoplamento em cadeia.
- Prefira composição a herança; dependa de abstração, não de implementação (DIP).
- Objetos expõem comportamento e escondem dados; estruturas de dados (DTOs) expõem dados e não têm comportamento. Não crie híbridos.

## Sistemas — separar construção de uso

- O grafo de objetos é montado num único lugar (main/composition root/container de DI); o resto do código **recebe** dependências prontas — não dá `new` em colaborador com lógica, não usa service locator, não acessa singleton/estático mutável.
- Isso é o que torna tudo testável: quem recebe a dependência recebe também o dublê no teste.
- Adie decisões até o último momento responsável — sistema cresce de simples para complexo com os testes garantindo cada passo, não nasce "preparado para tudo".

## Design simples emergente (4 regras, em ordem de prioridade)

1. **Passa em todos os testes** — design que não se consegue verificar não é design, é aposta.
2. **Sem duplicação** — cada regra decidida em um lugar só.
3. **Expressa a intenção** — nomes, funções pequenas, testes como documentação.
4. **Mínimo de classes e métodos** — a regra de MENOR prioridade: não crie cerimônia para satisfazer 1–3.

Código limpo não nasce limpo — **fica** limpo por refinamento sucessivo: primeiro faça funcionar (com testes), depois melhore em passos pequenos, rodando os testes a cada passo. Reescrever tudo de uma vez é como largar a sujeira: quase nunca termina bem.

## Concorrência

- Concorrência é decisão de design, não detalhe: **separe** o código concorrente da lógica de negócio (SRP também aqui) — a regra de negócio não sabe que roda em thread.
- Limite ao máximo dados compartilhados e seções críticas; prefira cópias imutáveis e threads independentes a sincronização esperta.
- Conheça a biblioteca da plataforma (coleções thread-safe, executores, async/await) antes de sincronizar na mão.
- **Nunca ignore falha intermitente de teste como "flaky"**: bug de threading aparece uma vez em mil execuções — trate a primeira aparição como bug real.

## Testes (as 3 Leis do TDD + F.I.R.S.T.)

- Teste é código de primeira classe: mesma qualidade do código de produção.
- Um conceito por teste; estrutura **Arrange / Act / Assert** (ou Given/When/Then).
- **F**ast, **I**ndependent (sem ordem entre testes), **R**epeatable (qualquer ambiente), **S**elf-validating (passa/falha, sem inspeção manual), **T**imely (junto com o código, não "depois").
- Nome do teste descreve o cenário e o resultado esperado: `deveRejeitarEmailDuplicado`, não `teste1`.

## Fronteiras e código de terceiros

- Envolva bibliotecas de terceiros atrás de uma interface sua (adapter). O dia que a lib mudar, muda um arquivo.
- Não deixe tipo de framework vazar para o domínio (entidade de domínio não conhece anotação de HTTP nem tipo do ORM).

## Cheiros de código (checklist do cap. 17, os que mais aparecem)

Sinais de alerta para escrita e revisão — cheiro não é veredito, é convite a olhar mais perto:

- **Rigidez/fragilidade**: mudança simples exige tocar N arquivos; conserto aqui quebra ali.
- **Duplicação**: a mesma decisão tomada em dois lugares — vão divergir. (Cuidado com o oposto: código *parecido* de conceitos diferentes não é duplicação.)
- **Complexidade especulativa**: abstração/configuração "para o futuro" que ninguém pediu (YAGNI).
- **Código morto**: função nunca chamada, branch impossível, feature flag fossilizada — delete.
- **Inveja de recursos (feature envy)**: método que usa mais os dados de OUTRA classe do que os da própria — está no lugar errado.
- **Switch/if em cadeia sobre tipo** repetido em vários pontos — polimorfismo resolvendo uma vez só.
- **Acoplamento temporal escondido**: métodos que só funcionam chamados numa ordem que nada força.
- **Variáveis explicativas ausentes**: expressão gigante inline em vez de passos intermediários nomeados.
- **Função/classe no nível errado de abstração**: detalhe de infraestrutura no meio de regra de negócio.
- **Nome que mente**: promete uma coisa, faz outra (ou faz MAIS — `save()` que também envia e-mail).

## A Regra do Escoteiro

Deixe o código mais limpo do que encontrou — mas em commit separado (`refactor:`), nunca misturado com a feature.
