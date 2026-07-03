# Squad Skills — um squad inteiro no seu projeto

Coleção de 16 skills para agentes de código (Claude Code, opencode e compatíveis com o padrão Agent Skills) que transforma qualquer projeto — vazio ou existente — em um projeto atendido por um squad virtual completo: arquitetura, backend, frontend, banco de dados, segurança, testes, Docker e Git, com fluxo de trabalho de ponta a ponta e restrições de qualidade invioláveis.

## Instalação

Copie a pasta `.opencode/` deste repositório para a raiz do seu projeto. Depois, conforme a ferramenta que você usa:

- **Claude Code** → renomeie a pasta para `.claude`. Pronto.
- **opencode** → use como está (`.opencode/`). Curiosidade útil: o opencode também lê skills de `.claude/skills/`, então a pasta renomeada para `.claude` funciona **nas duas ferramentas ao mesmo tempo** — é o nome recomendado se você alterna entre elas.

Só skills, sem agentes ou configuração extra — tudo roda no agente da sua conversa.

## Como usar

**Ponto de entrada: `/squad`** (ou simplesmente peça qualquer trabalho de desenvolvimento — a orquestradora é acionada automaticamente).

- **Projeto vazio** → a squad conduz o kickoff: você escolhe arquitetura, backend, frontend, banco e fluxo de Git (sempre com opções + resposta livre). As decisões ficam em `.squad/config.yaml` e em ADRs (`docs/adr/`), o esqueleto é gerado e só é declarado pronto depois da **prova de vida**: banco de pé, migration aplicada, API respondendo e frontend testado.
- **Projeto existente** → a squad infere/lê a stack e, a cada demanda ("adicione um telefone no cadastro de usuário"), executa o fluxo completo: análise de impacto → migration → domínio → API → frontend → testes → revisão → commit convencional. Nada fica pela metade.
- **Interrompeu no meio?** A memória de execução (`.squad/memory.md`) guarda o plano em checklist — a próxima sessão retoma exatamente de onde parou.

## As skills

| Skill | Papel |
|---|---|
| `squad` ⭐ | Orquestradora (tech lead): roteia demandas, garante o fluxo completo, a memória e o portão de qualidade |
| `squad-arquitetura` | Decisão de arquitetura com trade-offs, ADRs, DDD estratégico |
| `squad-backend-csharp` | C# / .NET / EF Core / xUnit |
| `squad-backend-java` | Java / Spring Boot / JPA / Flyway / JUnit |
| `squad-backend-node` | Node / TypeScript / NestJS-Fastify / Prisma / Vitest |
| `squad-backend-python` | Python / FastAPI-Django / SQLAlchemy / Alembic / pytest |
| `squad-frontend-react` | React / TanStack Query / React Hook Form / Testing Library |
| `squad-frontend-angular` | Angular / standalone / signals / Reactive Forms |
| `squad-database` | Postgres, Oracle, SQL Server e MongoDB (uma skill, referência por engine) |
| `squad-seguranca` | Requisitos de segurança: auth, IDOR, injeção, segredos, dependências, LGPD |
| `squad-testes-backend` | Estratégia de testes: pirâmide, dados de teste, mocks, banco real (base réplica no Docker) |
| `squad-testes-frontend` | Estratégia de testes no frontend: comportamento do usuário, MSW, E2E |
| `squad-docker` | Ambiente local: sobe/para/diagnostica containers sem você saber comandos |
| `squad-docs` | Documentação viva em Mermaid: C4, MER/DER, fluxos, glossário e regras de negócio |
| `squad-code-review` | Revisão antes de todo commit: correção, segurança, testes, arquitetura, clean code |
| `squad-git` | Conventional commits, Git Flow / GitHub Flow / trunk-based, PRs |

## Como o squad executa

A `squad` atua como tech lead: planeja, **delega as fases a subagentes** (cada um lê a skill especialista da sua fase e devolve resumo com evidência), valida os retornos contra a memória e consolida — quem mantém backend e frontend de acordo é o contrato em arquivos (config, ADRs, OpenAPI), não o contexto da conversa. Em harnesses sem subagentes, o mesmo fluxo roda inline. Antes de cada commit, a `squad-code-review` revisa o diff completo como portão de qualidade — sempre com olhos frescos.

## Princípios da coleção

1. **Quem decide é você.** As skills recomendam com justificativa curta e apresentam opções — nunca escolhem stack, arquitetura ou banco sozinhas.
2. **Fluxo completo, sempre.** Toda mudança percorre migration → código → testes → docs → revisão → commit. Código sem teste ou sem migration é trabalho inacabado.
3. **Não declarar pronto o que não foi executado.** Servidor levantado para verificação é derrubado em seguida; checkbox de progresso só é marcado com evidência; teste de integração roda em banco real.
4. **Consistência vence preferência.** Em projeto existente, o padrão do seu código vence o padrão da skill.
5. **Base comum de qualidade**: Código Limpo (`skills/squad/references/clean-code.md`) e DDD (`skills/squad-arquitetura/references/ddd.md`), destilados das obras dos autores.
6. **Economia de contexto.** Conhecimento conquistado uma vez não se paga duas vezes: fatos estáveis vão para o cache da memória, exploração pesada vai para subagentes que devolvem só conclusões, e a retomada lê a memória em vez de re-derivar a conversa.
7. **Skills bem delimitadas, sem redundância** — cada conhecimento vive em UM lugar: segurança define a régua que os outros implementam e a review confere; as skills de teste definem a estratégia que as stacks executam; banco de dados usa arquivos de referência por engine em vez de 4 skills separadas.

## Arquivos que a squad mantém no seu projeto

- `.squad/config.yaml` — o contrato: stack, arquitetura, fluxo de git escolhidos.
- `.squad/memory.md` — memória de execução **e cache de conhecimento**: plano em checklist, decisões, próximo passo e uma seção estável com os fatos do projeto (comandos, portas, caminhos, pegadinhas). Se a sessão cair, a próxima retoma de onde parou — e nenhuma sessão paga duas vezes pelo mesmo aprendizado, o que reduz consumo de tokens tanto em modelos pagos quanto na janela curta de modelos open source. Fica no `.gitignore`.
- `docs/` — documentação viva mantida pela `squad-docs`: C4, MER/DER, fluxos e jornadas em Mermaid, glossário e regras de negócio — criada do zero se não existir, atualizada a cada mudança se existir. Os ADRs vivem em `docs/adr/` (nunca editados, só substituídos).
