# Contributing to Erable

Thanks for considering a contribution. This document explains how to
work effectively in this repository.

The project is in heavy active development by a single author during
the first 10–12 weeks. External contributions are welcome but
review may take longer than usual until the MVP ships.

A French version of this guide is below — [Contribuer à Érable](#contribuer-à-érable).

---

## Quick links

- Code of conduct: [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
- Security disclosure: [SECURITY.md](./SECURITY.md)
- Privacy policy: [PRIVACY.md](./PRIVACY.md)
- Roadmap: [ROADMAP.md](./ROADMAP.md)

---

## Before you start

Before opening a pull request, please:

1. Open or comment on an issue describing what you intend to change.
   This avoids wasted work on something already in progress or
   outside the project's scope.
2. For non-trivial changes, wait for an acknowledgement before
   writing code.

If you have a question rather than a contribution, open a discussion
or send an e-mail to `hello@erable.app`.

---

## Development setup

See [README.md](./README.md#quickstart) for prerequisites and the
initial bootstrap. Once the stack is running, the day-to-day commands
are exposed through the `Makefile` at the repository root:

```sh
make dev          # start all services in development mode
make test         # run backend + frontend tests
make lint         # run ruff (Python) and eslint (TypeScript)
make typecheck    # run mypy and tsc --noEmit
make format       # auto-format with ruff format and prettier
make migrate      # apply pending Alembic migrations
```

A pre-commit hook is installed automatically when you run `make dev`
for the first time. It runs lint, typecheck, and conventional commit
validation before each commit.

---

## Branch and PR workflow

- The default branch is `main`. It is protected and only accepts
  changes through pull requests.
- Create a branch from `main` for your work. Suggested naming:
  `<type>/<short-slug>`, e.g. `feat/cv-builder-ai`,
  `fix/jwt-refresh-rotation`, `docs/adr-rag-pipeline`.
- Keep PRs focused. One concern per PR. If you find yourself
  describing two unrelated changes in the title, split them.
- Make sure CI is green before requesting review.
- Use the PR template that GitHub fills in automatically.

---

## Commit conventions

We follow [Conventional Commits](https://www.conventionalcommits.org/).
Subjects are in English, imperative mood, lowercase after the type,
72 characters or less.

Examples:

```
feat(auth): add jwt refresh token rotation
fix(rag): handle empty chunk arrays in retrieval
chore(deps): bump fastapi to 0.137.0
docs(adr): add ADR-002 on payment provider choice
test(cv-builder): cover french accent edge cases
ci: add ruff to pre-commit
```

Body and footer are optional. Add them only when the decision
benefits from being recorded — not for trivial changes.

---

## Code style

### Python (backend)

- Python 3.13, strict mypy, ruff for linting and formatting
- Type annotations are required on all public functions
- Use `async def` for I/O-bound code
- Pydantic v2 for input/output schemas
- SQLAlchemy 2.0 async for database access
- Tests with pytest + pytest-asyncio + factory-boy

### TypeScript (frontend)

- TypeScript strict mode, ESLint, Prettier
- Function components with hooks, no class components
- TanStack Query for server state, Zustand for client state
- shadcn/ui for primitives, Tailwind for layout and ad-hoc styling
- Tests with Vitest + Testing Library; Playwright for end-to-end

### Comments

Comments explain *why*, not *what*. Short by default, longer only
when the decision is non-obvious. No comments that simply restate the
function name or signature.

### Internationalization

User-facing strings must be wrapped in i18n calls. Both `fr.json` and
`en.json` are updated together; missing keys break CI.

---

## Tests

- New features must come with tests.
- Bug fixes should include a regression test that fails without the
  fix and passes with it.
- Coverage target is 80% across the project; PRs that lower the
  coverage materially will not be merged without justification.
- LLM-related changes must update the evaluation suite in
  `backend/tests/llm_evals/` if behavior is affected.

---

## Documentation

- Update the relevant README, ADR, or runbook when behavior or
  architecture changes.
- New architecture decisions deserve an ADR in `docs/adr/`. Use the
  template in that folder.
- Public API changes must be reflected in the OpenAPI schema and the
  generated TypeScript types.

---

## Legal scope reminder

Erable provides general information based on official Canadian public
sources. Contributions must not introduce features that constitute
immigration, legal, or tax advice — that scope is reserved for
licensed professionals. The CI runs a dedicated suite of tests that
verify the AI refuses out-of-scope questions; please do not weaken
those tests.

---

## License

By contributing, you agree that your contributions are licensed under
the [MIT License](./LICENSE) of this repository.

---
---

# Contribuer à Érable

Merci d'envisager une contribution. Ce document explique comment
travailler efficacement dans ce dépôt.

Le projet est en développement intensif par un seul auteur durant les
10 à 12 premières semaines. Les contributions externes sont les
bienvenues, mais la revue peut prendre plus de temps que d'habitude
tant que le MVP n'est pas livré.

---

## Liens rapides

- Code de conduite : [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
- Signalement de sécurité : [SECURITY.md](./SECURITY.md)
- Politique de confidentialité : [PRIVACY.md](./PRIVACY.md)
- Feuille de route : [ROADMAP.md](./ROADMAP.md)

---

## Avant de commencer

Avant d'ouvrir une pull request, merci de :

1. Ouvrir ou commenter une issue décrivant ce que vous voulez
   changer. Cela évite de gaspiller du temps sur quelque chose déjà
   en cours ou hors du périmètre du projet.
2. Pour les changements non triviaux, attendre une confirmation
   avant d'écrire du code.

Pour une question plutôt qu'une contribution, ouvrez une discussion
ou envoyez un e-mail à `hello@erable.app`.

---

## Mise en place du développement

Voir [README.fr.md](./README.fr.md#démarrage-rapide) pour les
prérequis et l'amorçage initial. Une fois la stack en place, les
commandes du quotidien sont exposées via le `Makefile` à la racine du
dépôt (voir la section anglaise ci-dessus pour la liste).

Un hook pre-commit est installé automatiquement au premier `make dev`.
Il lance le lint, le typecheck et la validation des messages de
commit avant chaque commit.

---

## Workflow de branches et de PR

- La branche par défaut est `main`. Elle est protégée et n'accepte
  les changements que via pull request.
- Créez une branche depuis `main` pour votre travail. Nommage
  suggéré : `<type>/<slug-court>`, par exemple `feat/cv-builder-ai`,
  `fix/jwt-refresh-rotation`, `docs/adr-rag-pipeline`.
- Gardez vos PR ciblées. Un sujet par PR. Si vous décrivez deux
  changements sans rapport dans le titre, séparez-les.
- Assurez-vous que la CI est verte avant de demander la revue.
- Utilisez le template de PR que GitHub remplit automatiquement.

---

## Conventions de commit

Nous suivons [Conventional Commits](https://www.conventionalcommits.org/).
Les sujets sont en anglais, à l'impératif, en minuscules après le
type, 72 caractères maximum.

Voir la section anglaise pour les exemples.

---

## Style de code

Voir la section anglaise. Les commentaires de code sont en anglais.

### Internationalisation

Les chaînes destinées aux utilisateurs doivent passer par des appels
i18n. Les fichiers `fr.json` et `en.json` sont mis à jour ensemble ;
toute clé manquante fait échouer la CI.

---

## Tests

- Toute nouvelle fonctionnalité doit être accompagnée de tests.
- Les corrections de bugs doivent inclure un test de régression qui
  échoue sans le correctif et passe avec.
- L'objectif de couverture est de 80 % sur l'ensemble du projet ;
  les PR qui font baisser la couverture significativement ne seront
  pas fusionnées sans justification.
- Les changements liés au LLM doivent mettre à jour la suite
  d'évaluation dans `backend/tests/llm_evals/` si le comportement est
  affecté.

---

## Documentation

- Mettez à jour le README, l'ADR ou le runbook concerné quand le
  comportement ou l'architecture change.
- Toute nouvelle décision d'architecture mérite un ADR dans
  `docs/adr/`. Utilisez le template dans ce dossier.
- Les changements d'API publique doivent être reflétés dans le
  schéma OpenAPI et les types TypeScript générés.

---

## Rappel sur le cadre légal

Érable fournit de l'information générale basée sur des sources
canadiennes officielles publiques. Les contributions ne doivent pas
introduire de fonctionnalités qui constituent du conseil en
immigration, juridique ou fiscal — ce périmètre est réservé aux
professionnels agréés. La CI exécute une suite de tests dédiée qui
vérifie que l'IA refuse les questions hors périmètre ; merci de ne
pas affaiblir ces tests.

---

## Licence

En contribuant, vous acceptez que vos contributions soient publiées
sous la [licence MIT](./LICENSE) de ce dépôt.
