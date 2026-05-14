---
name: git-workflow
description: >
  Boas práticas de gestão de repositórios Git: branching strategy, commits semânticos,
  versionamento semântico, fluxo de Pull Requests, proteção de branches, pipeline de
  ambientes (main/homolog/develop) e uso do GitHub CLI (gh) para gerenciar repositórios,
  PRs, issues, releases e configurações remotas diretamente pelo terminal. Use esta skill
  sempre que o usuário perguntar sobre como organizar um repositório Git, como nomear
  branches, como escrever commits, como fazer versionamento, como configurar regras de
  PR/merge, como separar ambientes de deploy, como usar o comando gh, como criar repos
  pelo terminal, como gerenciar PRs e issues via CLI, ou qualquer dúvida sobre fluxo de
  trabalho com Git e GitHub em equipes. Também acione quando o usuário mencionar Gitflow,
  trunk-based development, conventional commits, semantic versioning, squash merge,
  rebase, proteção de branch, gh cli, github cli ou automação de repositórios GitHub.
---

# Git Workflow — Boas Práticas

Guia completo de gestão de repositórios Git para equipes, cobrindo branching strategy,
commits, versionamento, ambientes e regras de Pull Request.

---

## 1. Branching Strategy

### Modelo recomendado: GitHub Flow com ambientes explícitos

Para a maioria dos projetos com ambientes distintos (dev → homolog → produção):

```
main          ← produção (sempre estável, sempre deployável)
homolog       ← staging / pré-produção (espelho da main, integração final)
develop       ← integração contínua (base para features)
feature/*     ← funcionalidades novas
fix/*         ← correções de bugs
hotfix/*      ← correções urgentes em produção
release/*     ← preparação de release (bump de versão, changelog)
chore/*       ← tarefas técnicas (atualização de deps, configs, CI)
docs/*        ← documentação
refactor/*    ← refatoração sem mudança de comportamento
```

### Regras de fluxo entre branches

```
feature/* ──► develop ──► homolog ──► main
fix/*     ──► develop
hotfix/*  ──► main  (e backport para develop)
release/* ──► main + develop (tag de versão)
```

> **Importante:** Nunca commitar diretamente em `main` ou `homolog`. Todo código entra
> via Pull Request com revisão obrigatória.

---

## 2. Nomenclatura de Branches

### Padrão

```
<tipo>/<identificador>-<descricao-curta>
```

### Exemplos

```bash
feature/123-autenticacao-oauth
feature/456-dashboard-relatorios
fix/789-corrige-calculo-imposto
hotfix/001-sql-injection-login
release/2.4.0
chore/atualiza-dependencias-outubro
docs/api-endpoints-v2
refactor/extrair-servico-pagamento
```

### Regras de nomenclatura

- Sempre **lowercase** com hífens (sem underscores, sem espaços)
- Incluir o número do ticket/issue quando existir (`feature/PROJ-123-descricao`)
- Máximo de 50 caracteres após o prefixo
- Sem caracteres especiais (`@`, `#`, `!`, etc.)
- Sem acentos ou cedilha

---

## 3. Commits Semânticos (Conventional Commits)

Seguir a especificação [Conventional Commits](https://www.conventionalcommits.org/).

### Formato

```
<tipo>[escopo opcional]: <descrição curta>

[corpo opcional]

[rodapé(s) opcional(is)]
```

### Tipos principais

| Tipo       | Uso                                                              |
|------------|------------------------------------------------------------------|
| `feat`     | Nova funcionalidade para o usuário                               |
| `fix`      | Correção de bug                                                  |
| `docs`     | Apenas documentação                                              |
| `style`    | Formatação, ponto-vírgula, etc. (sem mudança de lógica)          |
| `refactor` | Refatoração (sem nova feature, sem fix)                          |
| `perf`     | Melhoria de performance                                          |
| `test`     | Adição ou correção de testes                                     |
| `build`    | Mudanças no sistema de build ou dependências externas            |
| `ci`       | Mudanças nos arquivos e scripts de CI/CD                         |
| `chore`    | Outras tarefas que não alteram código de produção                |
| `revert`   | Reverte um commit anterior                                       |

### Exemplos de commits

```bash
# Feature simples
feat(auth): adiciona login com Google OAuth

# Fix com escopo
fix(checkout): corrige cálculo de frete para regiões Norte

# Breaking change (! indica breaking, BREAKING CHANGE no rodapé detalha)
feat(api)!: remove endpoint v1/usuarios

BREAKING CHANGE: o endpoint /v1/usuarios foi removido.
Migre para /v2/users conforme documentação em docs/migration-v2.md

# Commit com referência ao issue
fix(pagamento): trata timeout na integração com gateway

Closes #456
Refs PROJ-789

# Revert
revert: feat(auth): adiciona login com Google OAuth

This reverts commit a1b2c3d.
```

### Regras de commit

- **Assunto:** máximo 72 caracteres, imperativo, sem ponto final
- **Corpo:** separado por linha em branco, explica *o quê* e *por quê* (não *como*)
- **Rodapé:** referências a issues, breaking changes
- Um commit = uma mudança lógica (commits atômicos)
- Nunca commitar código comentado, arquivos de debug ou `.env` com segredos
- Nunca usar mensagens vagas: ~~`fix`~~, ~~`ajustes`~~, ~~`wip`~~, ~~`teste`~~

### Configuração do commitlint (recomendado)

```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional
echo "module.exports = { extends: ['@commitlint/config-conventional'] };" > commitlint.config.js
```

```bash
# .husky/commit-msg
npx --no -- commitlint --edit "$1"
```

---

## 4. Versionamento Semântico (SemVer)

Seguir [semver.org](https://semver.org/lang/pt-BR/): **MAJOR.MINOR.PATCH**

| Segmento | Quando incrementar                                        |
|----------|-----------------------------------------------------------|
| `MAJOR`  | Mudanças incompatíveis com versões anteriores (breaking)  |
| `MINOR`  | Nova funcionalidade compatível com versões anteriores     |
| `PATCH`  | Correção de bug compatível com versões anteriores         |

### Exemplos

```
1.0.0  → versão inicial estável
1.1.0  → nova feature adicionada
1.1.1  → bugfix
2.0.0  → breaking change (API incompatível)
```

### Versões de pré-lançamento

```
1.0.0-alpha.1
1.0.0-beta.3
1.0.0-rc.1
```

### Tags Git

Sempre criar tags anotadas para releases:

```bash
git tag -a v2.1.0 -m "Release 2.1.0: adiciona módulo de relatórios"
git push origin v2.1.0
```

### Changelog automatizado

Usar **standard-version** ou **release-please** para gerar CHANGELOG.md automaticamente
a partir dos commits convencionais:

```bash
npx standard-version          # bump automático + tag + changelog
npx standard-version --minor  # forçar minor
npx standard-version --major  # forçar major
```

---

## 5. Ambientes e Branches Protegidas

### Mapeamento branch → ambiente

| Branch    | Ambiente       | Deploy         | Quem pode mergear       |
|-----------|----------------|----------------|-------------------------|
| `main`    | Produção       | Automático/Tag | Tech Lead + aprovação   |
| `homolog` | Staging/UAT    | Automático     | Desenvolvedores sênior  |
| `develop` | Desenvolvimento| Automático     | Qualquer dev via PR     |

### Regras de proteção (Branch Protection Rules)

Configurar no GitHub/GitLab para `main`, `homolog` e `develop`:

```yaml
# Exemplo equivalente para GitHub Branch Protection
main:
  require_pull_request: true
  required_approving_reviews: 2          # mínimo 2 aprovações
  dismiss_stale_reviews: true            # invalida aprovação após novo push
  require_code_owner_review: true        # CODEOWNERS obrigatório
  require_status_checks: true
    contexts:
      - ci/tests
      - ci/lint
      - ci/build
  require_branches_up_to_date: true      # branch deve estar atualizada
  restrict_pushes: true                  # bloqueia push direto
  require_signed_commits: true           # commits assinados (opcional)
  include_administrators: true           # regras valem para admins também

homolog:
  require_pull_request: true
  required_approving_reviews: 1
  require_status_checks: true
  restrict_pushes: true

develop:
  require_pull_request: true
  required_approving_reviews: 1
  require_status_checks: true
    contexts:
      - ci/tests
      - ci/lint
```

### Arquivo CODEOWNERS

```
# .github/CODEOWNERS

# Time responsável por todo o repositório
*                   @org/core-team

# Backend: engenheiros backend
/src/api/**         @org/backend-team
/src/services/**    @org/backend-team

# Frontend
/src/components/**  @org/frontend-team
/src/pages/**       @org/frontend-team

# Infraestrutura
/.github/**         @org/devops-team
/docker/**          @org/devops-team
/terraform/**       @org/devops-team

# Segurança — qualquer mudança em auth requer revisão de segurança
/src/auth/**        @org/security-team
```

---

## 6. Pull Requests — Boas Práticas

### Template de PR

Criar `.github/pull_request_template.md`:

```markdown
## 📋 Descrição

<!-- Descreva o que foi feito e por quê -->

## 🎯 Tipo de mudança

- [ ] `feat` — Nova funcionalidade
- [ ] `fix` — Correção de bug
- [ ] `refactor` — Refatoração
- [ ] `docs` — Documentação
- [ ] `chore` — Tarefa técnica

## 🔗 Issues relacionadas

Closes #

## ✅ Checklist

- [ ] Código segue os padrões do projeto (lint passou)
- [ ] Testes foram adicionados/atualizados
- [ ] Todos os testes passam localmente
- [ ] Documentação atualizada (se aplicável)
- [ ] Não há segredos, tokens ou dados sensíveis no código
- [ ] Revisão de performance considerada
- [ ] Breaking changes documentadas

## 🧪 Como testar

<!-- Passos para reproduzir / testar a mudança -->

1.
2.
3.

## 📸 Screenshots (se aplicável)

<!-- Antes / Depois -->
```

### Regras para PRs

**Abertura:**
- PR pequeno e focado: idealmente < 400 linhas alteradas
- Título segue o padrão de commit semântico: `feat(auth): adiciona SSO`
- Descrição completa preenchida
- Labels aplicadas (`feature`, `bug`, `breaking-change`, etc.)
- Milestone associada (quando aplicável)
- Reviewer(s) atribuídos explicitamente

**Durante revisão:**
- Responder todos os comentários antes de solicitar re-review
- Não fazer force push após aprovação
- Manter o PR atualizado com a base (`git rebase develop`)

**Merge:**
- Estratégia padrão: **Squash and Merge** para `develop` (mantém histórico limpo)
- **Merge Commit** para `develop → homolog` e `homolog → main` (preserva contexto)
- Deletar a branch após o merge (configurar no repositório)
- Nunca mergear PR próprio sem aprovação de outra pessoa

### Estratégias de merge recomendadas

```
feature/* → develop      : Squash and Merge
                           (1 PR = 1 commit limpo no histórico)

develop → homolog         : Merge Commit
                           (preserva contexto completo do sprint)

homolog → main            : Merge Commit + Tag de versão

hotfix/* → main           : Merge Commit + backport para develop
```

---

## 7. Fluxo Completo — Passo a Passo

### Desenvolver uma feature

```bash
# 1. Atualizar develop local
git checkout develop
git pull origin develop

# 2. Criar branch de feature
git checkout -b feature/PROJ-123-autenticacao-oauth

# 3. Desenvolver com commits atômicos e semânticos
git add src/auth/oauth.ts
git commit -m "feat(auth): adiciona integração com provider OAuth"

git add tests/auth/oauth.test.ts
git commit -m "test(auth): adiciona testes de integração OAuth"

# 4. Manter branch atualizada com develop (rebase preferível ao merge)
git fetch origin
git rebase origin/develop

# 5. Push e abrir PR
git push origin feature/PROJ-123-autenticacao-oauth
# → Abrir PR: feature/PROJ-123-autenticacao-oauth → develop
```

### Promover para homolog

```bash
# Após PR aprovado e mergeado em develop via CI
# O deploy para homolog ocorre automaticamente via pipeline
# Se manual:
git checkout homolog
git pull origin homolog
git merge --no-ff develop -m "merge: promote develop to homolog [sprint-42]"
git push origin homolog
```

### Release para produção

```bash
# 1. Criar branch de release a partir de homolog
git checkout -b release/2.4.0 origin/homolog

# 2. Bump de versão e changelog
npx standard-version --release-as 2.4.0

# 3. Ajustes finais (se necessário): apenas bugfixes, sem novas features
git commit -m "fix(checkout): corrige edge case no cálculo de desconto"

# 4. PR: release/2.4.0 → main
# Após aprovação e merge:
git tag -a v2.4.0 -m "Release 2.4.0"
git push origin v2.4.0

# 5. Backport para develop
git checkout develop
git merge --no-ff release/2.4.0 -m "merge: backport release/2.4.0 into develop"
git push origin develop
```

### Hotfix em produção

```bash
# 1. Criar hotfix a partir da main
git checkout main
git pull origin main
git checkout -b hotfix/001-corrige-vulnerabilidade-xss

# 2. Corrigir e commitar
git commit -m "fix(seguranca): sanitiza input do campo de busca - CVE-2024-XXXX"

# 3. PR: hotfix/001 → main (aprovação prioritária)
# Após merge:
git tag -a v2.3.1 -m "Hotfix 2.3.1: corrige XSS no campo de busca"
git push origin v2.3.1

# 4. Backport obrigatório para develop
git checkout develop
git cherry-pick <hash-do-commit-do-fix>
git push origin develop
```

---

## 8. .gitignore e .gitattributes

### .gitignore mínimo recomendado

```gitignore
# Dependências
node_modules/
vendor/
.venv/
__pycache__/

# Builds
dist/
build/
*.egg-info/

# Variáveis de ambiente — NUNCA versionar
.env
.env.local
.env.*.local
*.env

# Editores
.vscode/settings.json
.idea/
*.swp
*.swo
.DS_Store
Thumbs.db

# Logs e temporários
*.log
tmp/
temp/
coverage/
.nyc_output/

# Cache
.cache/
.parcel-cache/
.next/
.nuxt/
```

### .gitattributes para consistência cross-platform

```gitattributes
# Normaliza line endings
* text=auto eol=lf

# Arquivos binários — sem diff de texto
*.png binary
*.jpg binary
*.gif binary
*.ico binary
*.pdf binary
*.zip binary
*.gz binary
*.jar binary

# Diffs customizados
*.md diff=markdown
```

---

## 9. Configuração Inicial Recomendada do Repositório

```bash
# Git config global recomendado
git config --global core.autocrlf false
git config --global core.eol lf
git config --global pull.rebase true          # rebase ao invés de merge no pull
git config --global fetch.prune true          # remove branches remotas deletadas
git config --global rerere.enabled true       # reutiliza resoluções de conflito
git config --global push.default current      # push na branch atual por padrão

# Alias úteis
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.st "status -sb"
git config --global alias.undo "reset HEAD~1 --mixed"
git config --global alias.aliases "config --get-regexp alias"
```

---

## 10. Checklist de Qualidade do Repositório

Antes de considerar um repositório bem configurado, verificar:

- [ ] `README.md` com descrição, como rodar localmente, variáveis de ambiente e links
- [ ] `CONTRIBUTING.md` com guia de contribuição
- [ ] `CHANGELOG.md` atualizado a cada release
- [ ] `.gitignore` configurado e revisado
- [ ] `.gitattributes` configurado
- [ ] `.github/pull_request_template.md` criado
- [ ] `.github/CODEOWNERS` configurado
- [ ] Branch protection rules ativas em `main`, `homolog` e `develop`
- [ ] CI pipeline obrigatório (lint + testes) antes do merge
- [ ] Squash merge configurado como padrão para PRs de feature
- [ ] Delete branch on merge ativo
- [ ] Tags de versão semântica em todos os releases
- [ ] Commits semânticos enforçados via commitlint + husky
- [ ] Secrets/tokens fora do repositório (usar vault, CI secrets ou .env não versionado)

---

## 11. GitHub CLI (`gh`) — Gestão do Repositório Remoto

O **GitHub CLI** (`gh`) permite gerenciar todo o ciclo de vida do repositório no GitHub
diretamente pelo terminal, sem abrir o navegador. É a forma recomendada para automatizar
e agilizar operações do dia a dia.

### Instalação e autenticação

```bash
# macOS
brew install gh

# Linux (Debian/Ubuntu)
sudo apt install gh

# Windows
winget install --id GitHub.cli

# Autenticar (abre navegador ou aceita token)
gh auth login

# Verificar autenticação
gh auth status
```

### Configuração inicial

```bash
# Definir editor padrão
gh config set editor "code --wait"          # VS Code
gh config set editor "vim"                  # Vim

# Definir protocolo padrão (SSH recomendado para equipes)
gh config set git_protocol ssh

# Ver todas as configurações
gh config list
```

---

### Repositórios

```bash
# Criar repositório público na organização
gh repo create minha-org/meu-projeto \
  --public \
  --description "Descrição do projeto" \
  --clone                                   # já clona localmente

# Criar repositório privado com template
gh repo create minha-org/meu-projeto \
  --private \
  --template minha-org/template-base

# Clonar repositório
gh repo clone minha-org/meu-projeto

# Listar repositórios da organização
gh repo list minha-org --limit 50

# Ver detalhes de um repositório
gh repo view minha-org/meu-projeto

# Abrir repositório no navegador
gh repo view --web

# Arquivar repositório
gh repo archive minha-org/meu-projeto

# Deletar repositório (requer confirmação)
gh repo delete minha-org/meu-projeto --confirm

# Fazer fork
gh repo fork minha-org/meu-projeto --clone

# Sincronizar fork com upstream
gh repo sync minha-org/meu-projeto --source upstream/main
```

---

### Pull Requests

```bash
# Criar PR da branch atual para develop (interativo)
gh pr create \
  --base develop \
  --title "feat(auth): adiciona login com OAuth" \
  --body "Implementa autenticação via Google OAuth 2.0. Closes #123" \
  --reviewer usuario1,usuario2 \
  --label "feature,needs-review" \
  --assignee "@me"

# Criar PR abrindo editor para preencher descrição
gh pr create --base develop --web

# Criar PR como draft
gh pr create --base develop --draft

# Listar PRs abertos
gh pr list

# Listar PRs com filtros
gh pr list --state open --assignee "@me"
gh pr list --state merged --base main --limit 20
gh pr list --label "breaking-change"

# Ver detalhes de um PR
gh pr view 42
gh pr view 42 --web                         # abre no navegador

# Verificar status dos checks de um PR
gh pr checks 42

# Fazer checkout de um PR para revisar localmente
gh pr checkout 42

# Aprovar um PR
gh pr review 42 --approve
gh pr review 42 --approve --body "LGTM! Ótima implementação."

# Solicitar mudanças
gh pr review 42 --request-changes --body "Por favor, adicione testes para o edge case."

# Comentar em um PR
gh pr comment 42 --body "Dependência do PR #40, aguardar merge antes."

# Mergear PR
gh pr merge 42 --squash --delete-branch    # squash (feature → develop)
gh pr merge 42 --merge --delete-branch     # merge commit (develop → main)
gh pr merge 42 --rebase                    # rebase

# Fechar PR sem mergear
gh pr close 42

# Reabrir PR
gh pr reopen 42

# Ver diff de um PR
gh pr diff 42

# Editar título, body ou reviewers de um PR
gh pr edit 42 --title "feat(auth): adiciona login com OAuth e 2FA"
gh pr edit 42 --add-reviewer usuario3
gh pr edit 42 --add-label "urgent"
```

---

### Issues

```bash
# Criar issue
gh issue create \
  --title "Bug: cálculo de frete incorreto para região Norte" \
  --body "Reprodução: ..." \
  --label "bug,priority-high" \
  --assignee usuario1 \
  --milestone "Sprint 12"

# Listar issues abertas
gh issue list
gh issue list --label "bug" --assignee "@me"
gh issue list --state closed --limit 30

# Ver uma issue
gh issue view 123
gh issue view 123 --web

# Fechar issue
gh issue close 123
gh issue close 123 --comment "Resolvido no PR #456."

# Reabrir issue
gh issue reopen 123

# Comentar em issue
gh issue comment 123 --body "Confirmado. Investigando causa raiz."

# Editar issue
gh issue edit 123 --title "Novo título"
gh issue edit 123 --add-label "regression"
gh issue edit 123 --add-assignee usuario2
gh issue edit 123 --milestone "Sprint 13"

# Transferir issue para outro repositório
gh issue transfer 123 minha-org/outro-repo
```

---

### Releases e Tags

```bash
# Criar release a partir de uma tag existente
gh release create v2.4.0 \
  --title "Release 2.4.0 — Módulo de Relatórios" \
  --notes "## O que há de novo
- feat: dashboard de relatórios (#89)
- fix: cálculo de frete para Norte (#102)
- chore: atualiza dependências" \
  --target main

# Criar release com notas geradas automaticamente a partir dos PRs mergeados
gh release create v2.4.0 \
  --generate-notes \
  --title "Release 2.4.0"

# Criar pre-release (alpha, beta, rc)
gh release create v3.0.0-rc.1 \
  --title "Release Candidate 3.0.0-rc.1" \
  --prerelease

# Fazer upload de artefatos junto com a release
gh release create v2.4.0 \
  --generate-notes \
  ./dist/app-linux-amd64 \
  ./dist/app-darwin-arm64 \
  ./dist/app-windows.exe

# Listar releases
gh release list

# Ver detalhes de uma release
gh release view v2.4.0

# Editar release existente
gh release edit v2.4.0 --notes "Notas atualizadas"

# Deletar release (mantém a tag)
gh release delete v2.4.0

# Deletar tag remota
git tag -d v2.4.0
git push origin :refs/tags/v2.4.0
```

---

### Branch Protection via CLI

O `gh` permite configurar regras de proteção de branch usando a API do GitHub:

```bash
# Proteger a branch main
gh api \
  --method PUT \
  repos/minha-org/meu-projeto/branches/main/protection \
  --input - <<'EOF'
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["ci/tests", "ci/lint", "ci/build"]
  },
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "required_approving_review_count": 2,
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF

# Proteger develop com 1 reviewer
gh api \
  --method PUT \
  repos/minha-org/meu-projeto/branches/develop/protection \
  --input - <<'EOF'
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["ci/tests", "ci/lint"]
  },
  "enforce_admins": false,
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF

# Ver proteções atuais de uma branch
gh api repos/minha-org/meu-projeto/branches/main/protection

# Remover proteção de uma branch
gh api \
  --method DELETE \
  repos/minha-org/meu-projeto/branches/main/protection
```

---

### Secrets e Variáveis de Ambiente

```bash
# Adicionar secret (valor lido do stdin — nunca fica no shell history)
gh secret set DATABASE_URL
# → Prompt: Cole o valor e pressione Enter

# Adicionar secret a partir de arquivo
gh secret set PRIVATE_KEY < chave-privada.pem

# Listar secrets (apenas nomes, nunca valores)
gh secret list

# Deletar secret
gh secret delete DATABASE_URL

# Adicionar variável de ambiente do repositório (não sensível)
gh variable set APP_ENV --body "production"
gh variable set API_URL --body "https://api.meuapp.com"

# Listar variáveis
gh variable list

# Secret de organização com visibilidade restrita a repos selecionados
gh secret set SHARED_API_KEY \
  --org minha-org \
  --visibility selected \
  --repos repo1,repo2,repo3
```

---

### Actions e Workflows

```bash
# Listar workflows
gh workflow list

# Disparar workflow manualmente (workflow_dispatch)
gh workflow run ci.yml
gh workflow run deploy.yml --field environment=production

# Listar execuções de um workflow
gh run list --workflow ci.yml
gh run list --branch develop --status failure

# Ver logs de uma execução
gh run view 12345678 --log

# Ver logs de uma job específica
gh run view 12345678 --job "test-unit"

# Cancelar execução em andamento
gh run cancel 12345678

# Re-executar apenas os jobs que falharam
gh run rerun 12345678 --failed

# Baixar artefatos de uma execução
gh run download 12345678 --name "coverage-report" --dir ./coverage
```

---

### Labels e Milestones

```bash
# Criar labels padrão do projeto
gh label create "feature"         --color "0075ca" --description "Nova funcionalidade"
gh label create "bug"             --color "d73a4a" --description "Algo não funciona"
gh label create "hotfix"          --color "e4e669" --description "Correção urgente em produção"
gh label create "breaking-change" --color "b60205" --description "Quebra compatibilidade"
gh label create "needs-review"    --color "fbca04" --description "Aguardando revisão"
gh label create "blocked"         --color "000000" --description "Bloqueado por dependência"

# Clonar labels de outro repositório (ótimo para padronizar a organização)
gh label clone minha-org/repo-referencia

# Listar labels
gh label list

# Criar milestone via API
gh api \
  --method POST \
  repos/minha-org/meu-projeto/milestones \
  -f title="Sprint 12" \
  -f due_on="2025-06-30T00:00:00Z" \
  -f description="Metas do sprint 12"
```

---

### Configurações do Repositório

```bash
# Configurar opções de merge e comportamentos recomendados
gh api \
  --method PATCH \
  repos/minha-org/meu-projeto \
  -F delete_branch_on_merge=true \
  -F allow_squash_merge=true \
  -F allow_merge_commit=true \
  -F allow_rebase_merge=false \
  -F squash_merge_commit_title="PR_TITLE" \
  -F squash_merge_commit_message="PR_BODY" \
  -F has_issues=true \
  -F has_wiki=false

# Ver configurações atuais
gh api repos/minha-org/meu-projeto \
  | jq '{default_branch,delete_branch_on_merge,allow_squash_merge,allow_merge_commit}'
```

---

### Fluxo Completo com `gh` — Do Zero ao PR Mergeado

```bash
# 1. Criar e clonar repositório
gh repo create minha-org/meu-projeto --private --clone
cd meu-projeto

# 2. Criar branches de ambiente e proteger
git checkout -b develop && git push -u origin develop
git checkout -b homolog && git push -u origin homolog
git checkout main
# → aplicar branch protection rules com gh api (ver seção acima)

# 3. Padronizar labels
gh label clone minha-org/template-labels

# 4. Criar feature branch e desenvolver
git checkout develop
git checkout -b feature/PROJ-42-exportacao-csv
# ... commits semânticos ...
git push -u origin feature/PROJ-42-exportacao-csv

# 5. Abrir PR
gh pr create \
  --base develop \
  --title "feat(exportacao): adiciona exportação em CSV" \
  --body "$(cat .github/pull_request_template.md)" \
  --reviewer colega1,colega2 \
  --label "feature,needs-review"

# 6. Monitorar checks
gh pr checks --watch                        # aguarda até todos passarem

# 7. Após aprovação, mergear e publicar
gh pr merge --squash --delete-branch

# 8. Criar release
gh release create v1.2.0 \
  --generate-notes \
  --title "Release 1.2.0"
```

---

## Referências

- [GitHub CLI — Documentação oficial](https://cli.github.com/manual/)
- [Conventional Commits](https://www.conventionalcommits.org/pt-br/)
- [Semantic Versioning](https://semver.org/lang/pt-BR/)
- [GitHub Flow](https://docs.github.com/en/get-started/quickstart/github-flow)
- [Gitflow Workflow — Atlassian](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
- [Google Engineering Practices — Code Review](https://google.github.io/eng-practices/review/)
- [Trunk Based Development](https://trunkbaseddevelopment.com/)
