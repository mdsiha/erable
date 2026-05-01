# Érable

> Votre guide pour vous installer au Canada — sans stress, sans erreurs.

S'installer au Canada quand on arrive de l'étranger, c'est une suite
de démarches qu'on découvre au fur et à mesure : NAS, compte
bancaire, OHIP, garderie, permis, déclaration d'impôts, et une
dizaine d'autres choses qui prennent normalement des semaines à
comprendre seul, à grand renfort de fils Reddit contradictoires et de
sites gouvernementaux écrits en 2010.

Érable est une application web qui rassemble tout ça en un seul
endroit, en français, avec une IA qui répond à vos questions en
citant ses sources. La première phase se concentre sur Ottawa, en
Ontario. D'autres villes et provinces suivront.

Une version anglaise de ce README est disponible : [README.md](./README.md).

---

## État du projet

**En développement. Lancement public prévu environ 12 semaines après le démarrage.**

Ce dépôt est public depuis le premier jour. Le code source *est* le
produit — pas d'édition entreprise fermée, pas de modules premium
cachés. Le modèle économique repose sur des partenariats d'affiliation
transparents, des services IA à l'unité, et un abonnement Premium,
tous documentés ouvertement.

---

## Pourquoi ce projet existe

L'auteur est lui-même un arrivant francophone qui s'installe à Ottawa
fin 2026. Chaque problème qu'Érable cherche à résoudre est un problème
qu'il vit ou qu'il s'apprête à vivre : choisir une banque sans se
faire avoir sur les frais cachés, comprendre un bail Ontario en
anglais juridique, décider entre conseil scolaire francophone et
anglophone d'immersion, ne pas rater une deadline d'inscription en
garderie.

Les ressources existantes ont chacune leur problème : Settlement.org
a une UX figée en 2010, Moving2Canada et WelcomeAide sont en anglais
uniquement, l'application Arrive de RBC pousse les produits RBC, et
les consultants RCIC facturent 500 à 1 500 $ pour des informations
publiques. Érable cherche à combler ce vide avec une interface
moderne, une IA contextuelle, et une approche francophone par défaut,
tout en restant strictement dans le cadre légal de l'information
publique (pas de conseil en immigration — c'est réservé aux RCIC
agréés).

---

## Pile technique

**Backend** — Python 3.13, FastAPI, SQLAlchemy 2.0 async, PostgreSQL 16
avec pgvector, Redis 7, ARQ pour les tâches en arrière-plan, Pydantic v2.

**Frontend** — Next.js 14 (App Router), TypeScript strict, Tailwind CSS,
shadcn/ui, next-intl pour l'internationalisation FR/EN, PWA.

**IA** — Abstraction LLM agnostique du fournisseur, compatible
Anthropic Claude, OpenAI, Mistral et Ollama en local. Embeddings BGE-M3
pour le RAG multilingue. Tesseract pour l'OCR local privacy-first ;
Claude Vision uniquement comme repli avec consentement explicite.

**Infrastructure** — Docker, VPS Hetzner avec Coolify pour la
production, Cloudflare en bordure, GitHub Actions pour le CI/CD. Des
modèles Terraform AWS sont conservés dans `infrastructure/aws/` à
titre de référence, pas comme cible de production.

**Observabilité** — structlog, OpenTelemetry, Sentry, Grafana Cloud.

Les décisions d'architecture sont consignées dans
[`docs/adr/`](./docs/adr).

---

## Confidentialité

La confidentialité n'est pas une fonctionnalité, c'est un parti pris.
Les documents téléversés pour analyse sont traités en mémoire et ne
sont jamais persistés sur disque ou en base de données. L'OCR
fonctionne par défaut en local avec Tesseract ; envoyer un document à
Claude Vision exige un consentement explicite, document par document.
Le suivi des affiliations ne s'applique qu'aux utilisateurs
authentifiés — pas d'empreinte numérique anonyme, pas de cookies tiers.

La politique de confidentialité complète se trouve dans [PRIVACY.md](./PRIVACY.md),
rédigée en français simple, pas en jargon juridique.

---

## Cadre légal

Érable fournit de l'information générale basée sur des sources
canadiennes officielles publiques. **Érable n'est PAS un service de
conseil en immigration, juridique ou fiscal.** Pour tout conseil
personnalisé, consultez un consultant en immigration canadien
réglementé (RCIC) ou un avocat membre du barreau de votre province.

Cet avertissement est appliqué dans le produit lui-même : l'IA est
testée pour refuser les questions qui sortent du cadre de
l'information publique, et ces tests font partie du pipeline
d'intégration continue.

---

## Démarrage rapide

> Le bootstrap est en cours. Les instructions détaillées arriveront
> avec la première version exécutable de la stack. Pour l'instant,
> cette section documente l'expérience développeur cible.

**Prérequis**

- Docker 25+ et Docker Compose v2
- Node.js 20+ et pnpm 9+
- Python 3.13+ et [uv](https://github.com/astral-sh/uv)
- GNU Make
- Un environnement Linux ou macOS (Fedora 44 est l'environnement de référence)

**Installation**

```sh
git clone git@github.com:mdsiha/erable.git
cd erable
cp .env.example .env  # puis remplir les secrets locaux
make dev              # démarre postgres, redis, backend, frontend
```

Un `Makefile` à la racine du dépôt servira de point d'entrée unique
pour les tâches courantes (`make dev`, `make test`, `make lint`,
`make migrate`).

---

## Organisation du dépôt

```
erable/
├── backend/          Service FastAPI + workers ARQ
├── frontend/         Application Next.js
├── infrastructure/   Configs Coolify (prod) et Terraform AWS (référence)
├── docs/             Architecture, ADR, runbooks
└── .github/          Workflows CI, templates issues et PR
```

---

## Contribuer

Le projet est en développement intensif par un seul auteur durant les
10 à 12 premières semaines. Les contributions externes sont les
bienvenues, mais leur revue peut prendre du temps tant que le MVP
n'est pas livré.

Voir [CONTRIBUTING.md](./CONTRIBUTING.md) pour le processus de
contribution, et [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) pour les
attentes de la communauté.

Pour signaler une faille de sécurité, suivez la procédure indiquée
dans [SECURITY.md](./SECURITY.md).

---

## Feuille de route

Une feuille de route à haut niveau est tenue à jour dans [ROADMAP.md](./ROADMAP.md).
Le travail détaillé se déroule via les Issues et Projects de GitHub.

---

## Licence

[MIT](./LICENSE) © 2026 Michael Das

---

## Contact

Pour toute question générale : `hello@erable.app`

Pour le reste, les Issues GitHub sont le canal préféré.
