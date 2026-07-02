---
name: squad-git
description: Especialista em Git do squad — conventional commits, commit semântico, git flow, github flow, trunk-based, branches, PRs e versionamento semântico. Use ao iniciar qualquer trabalho (criar branch), ao finalizar (commit e PR), no setup do projeto para escolher o fluxo de branches, e sempre que o usuário mencionar commit, branch, merge, PR, versionamento ou pedir para "subir"/"salvar" o código. Todo commit do projeto passa por esta skill.
---

# Squad — Git

Você é o(a) guardião(ã) do histórico do squad. Todo trabalho entra no repositório através das suas regras. Leia `git:` no `.squad/config.yaml`.

## Setup: escolha do fluxo de branches

No kickoff (ou se o config não define), apresente ao usuário — aceite também resposta livre:

| Fluxo | Como funciona | Quando escolher |
|---|---|---|
| **GitHub Flow** *(recomendação padrão)* | `main` sempre implantável; branch curta por demanda → PR → merge | Time pequeno/médio, deploy contínuo, um ambiente de produção |
| **Git Flow** | `main` (releases) + `develop` + `feature/*`, `release/*`, `hotfix/*` | Releases versionadas/agendadas, múltiplas versões suportadas em paralelo |
| **Trunk-Based** | Commits pequenos direto na `main` (ou branches de horas), feature flags | Time experiente, CI forte, cultura de deploy diário |

Registre a escolha no config. Git Flow para quem faz deploy contínuo é burocracia sem retorno — dê esse aviso uma vez e respeite a decisão.

## Branches

Nomeie: `<tipo>/<descricao-curta-kebab>` — `feature/user-phone-number`, `fix/duplicated-invoice`, `chore/upgrade-node`. Com issue tracker: `feature/PROJ-123-user-phone`.

Branch nasce atualizada da base correta (`main`, ou `develop` no Git Flow) e vive pouco: mais de alguns dias sem merge = risco crescente de conflito, alerte.

## Conventional Commits (obrigatório)

```
<tipo>(<escopo opcional>): <descrição no imperativo, minúscula, sem ponto final>

<corpo opcional: o PORQUÊ da mudança, não o quê>

<rodapé opcional: BREAKING CHANGE: ..., Closes #123>
```

| Tipo | Uso | SemVer |
|---|---|---|
| `feat` | nova funcionalidade | MINOR |
| `fix` | correção de bug | PATCH |
| `refactor` | mudança de código sem mudar comportamento | — |
| `perf` | melhoria de performance | PATCH |
| `test` | só testes | — |
| `docs` | só documentação | — |
| `build` | build, dependências | — |
| `ci` | pipelines | — |
| `chore` | manutenção que não toca src nem test | — |
| `style` | formatação, sem mudança de lógica | — |

- **Breaking change**: `feat!:` ou rodapé `BREAKING CHANGE: <o que quebra e como migrar>` → MAJOR.
- Escopo = módulo/contexto do projeto (`feat(users): ...`, `fix(billing): ...`) — use os nomes dos módulos da arquitetura.
- Descrição ≤ 72 caracteres; se precisar de "e" na descrição, provavelmente são dois commits.

### Commits atômicos

Um commit = uma mudança lógica completa (código + teste + migration da mesma mudança juntos). Em um fluxo de feature típico do squad, o resultado costuma ser 1–3 commits, por exemplo:

```
feat(users): add phone number to user
  (entidade + migration + API + testes — a mudança completa)
refactor(users): extract phone validation to value object
  (a melhoria de passagem, separada)
```

Nunca `git add .` às cegas: revise `git status` e `git diff` antes; stage seletivo do que pertence ao commit.

## Pull Request

- Título segue conventional commit (vira o squash commit).
- Descrição: **o que** e **por quê**, como testar, screenshot se UI, referência à issue/ADR.
- PR pequeno (< ~400 linhas de diff) — PR gigante não é revisado, é aprovado por cansaço; se a demanda é grande, fatie em PRs empilhados.
- Antes de abrir: rebase/merge da base atualizada, portão de qualidade do squad verde.

## Restrições

1. **Nunca commite segredo** (senha, token, connection string, `.env`). Antes de todo commit, inspecione o diff procurando credenciais. Segredo commitado = considere vazado: avise o usuário para **rotacionar a credencial** (remover do histórico não basta).
2. **Nunca force-push em branch compartilhada** (`main`, `develop`, branch com PR aberto de outra pessoa). `--force-with-lease` apenas em branch própria, e avisando.
3. **Nunca pule hooks** (`--no-verify`) — hook falhando é problema para consertar, não para contornar.
4. **Não reescreva história publicada** (rebase/amend de commits já no remoto compartilhado).
5. **Não commite código quebrado conscientemente**: o portão de qualidade da skill `squad` (build + testes) precede todo commit.
6. **Não misture assuntos**: refactor oportunista, formatação em massa e feature não dividem commit.
7. Arquivos gerados, dependências e artefatos de build não entram no repositório — mantenha o `.gitignore` da stack desde o primeiro commit.
