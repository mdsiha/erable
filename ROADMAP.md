# Roadmap

> *Version française disponible plus bas — [Feuille de route](#feuille-de-route).*

This roadmap describes the planned trajectory of Erable. It is a
living document — priorities and dates may shift based on what we
learn from real users.

**Status legend**

- ⬜ Planned
- 🟦 In progress
- ✅ Shipped
- ❄️ Frozen / on hold

---

## Phase 0 — Foundations (weeks 1–2)

🟦 **In progress.**

The infrastructure and developer experience that everything else
builds on.

- ✅ Domain `erable.app` registered
- ✅ Public GitHub repository, MIT licensed
- 🟦 Repository structure, governance docs, issue templates
- ⬜ Local development environment (uv, Ollama, Tesseract, Docker)
- ⬜ Bootstrap of the FastAPI backend and Next.js frontend
- ⬜ `staging.erable.app` live on Hetzner with Coolify
- ⬜ CI/CD pipeline (lint, type-check, test, security, build)
- ⬜ Initial ADRs covering the major technical choices

---

## Phase 1 — MVP for Ottawa (weeks 3–10)

⬜ **Planned.**

A first usable product, focused entirely on Ottawa, Ontario.

### Core platform

- ⬜ Database schema and Alembic migrations
- ⬜ Authentication: signup, login, JWT with refresh rotation, magic
  link, password reset
- ⬜ User onboarding (province, status, family, country of origin)
- ⬜ Personalized administrative checklist

### Revenue features (built in from day one)

- ⬜ Bank account comparator with affiliate links
  (RBC, Scotiabank, BMO, TD, Desjardins)
- ⬜ Mobile/internet plan comparator
  (Public Mobile, Fizz, Bell, Rogers)
- ⬜ Temporary health insurance comparator (Manulife, Sun Life)
- ⬜ Canadian-format CV builder (one-shot, $29–49)
- ⬜ Cover letter generator (likely bundled with CV builder)
- ⬜ ECA Concierge — automated guidance for credential evaluation
  (one-shot, $49)
- ⬜ Lemon Squeezy integration in production

### Content and SEO

- ⬜ Eight detailed Ottawa-specific pages: SIN, OHIP, driver's
  licence, banking, school enrolment, subsidized childcare, housing,
  first tax return
- ⬜ Structured data, sitemap, hreflang, OpenGraph
- ⬜ `<DataFreshnessIndicator />` component on every information
  page

### AI and freshness pipeline

- ⬜ Strict RAG with mandatory citations
- ⬜ Weekly re-crawl of official sources with chunk versioning
- ⬜ AI chat for complex questions
- ⬜ LLM abstraction supporting Claude, OpenAI, Mistral, Ollama

### Quality and launch

- ⬜ End-to-end tests for critical user journeys
- ⬜ Test coverage above 80%
- ⬜ Private beta with 50 testers
- ⬜ Public launch on Product Hunt, Hacker News, Reddit, Facebook

---

## Phase 2 — Premium and scale (weeks 13–26)

⬜ **Planned.**

Once the MVP has real users, we add the recurring revenue features
and start expanding geographically.

- ⬜ Document analysis (privacy-first OCR with Tesseract, Claude
  Vision as opt-in fallback)
- ⬜ Multi-member family tracking
- ⬜ Advanced reminders across e-mail and web push
- ⬜ Premium subscription "Erable Settlement Pro" ($9.99/month or
  $79/year)
- ⬜ Family plan ($14.99/month, up to 4 members)
- ⬜ Toronto, then the rest of Ontario

---

## Phase 3 — B2B and seasonal modules (months 6–9)

⬜ **Planned.**

- ⬜ White-label B2B for francophone settlement organizations
  (CESOC, Centre francophone de Toronto, La Cité collégiale, OCISO,
  OCASI). Pricing: $299–999/month.
- ⬜ Newcomer Tax module for the March–April 2027 filing season,
  with CPA partner validation ($99–149 per return)
- ⬜ Quebec
- ⬜ Referral program to certified RCICs (10–15% commission)

---

## Phase 4 — Beyond (months 9+)

⬜ **Future.**

- ⬜ British Columbia, Alberta, New Brunswick
- ⬜ Native mobile app (only if PWA proves insufficient)
- ⬜ Additional language support (Spanish, Arabic for newcomer
  communities)
- ⬜ Open data API for partner organizations

---

## Out of scope

Some things will never be in Erable, by design:

- ❄️ Personalized immigration advice (illegal without RCIC license)
- ❄️ IRCC application strategy or representation
- ❄️ Predicting acceptance probabilities
- ❄️ Filling IRCC forms on behalf of clients in exchange for direct
  payment

---
---

# Feuille de route

Cette feuille de route décrit la trajectoire planifiée d'Érable. C'est
un document vivant — les priorités et les dates peuvent évoluer en
fonction de ce qu'on apprend des utilisateurs réels.

**Légende**

- ⬜ Planifié
- 🟦 En cours
- ✅ Livré
- ❄️ Gelé / en pause

---

## Phase 0 — Fondations (semaines 1–2)

🟦 **En cours.**

L'infrastructure et l'expérience développeur sur lesquelles tout le
reste se construit.

- ✅ Nom de domaine `erable.app` réservé
- ✅ Dépôt GitHub public, licence MIT
- 🟦 Structure du dépôt, documents de gouvernance, templates d'issue
- ⬜ Environnement de développement local (uv, Ollama, Tesseract, Docker)
- ⬜ Bootstrap du backend FastAPI et du frontend Next.js
- ⬜ `staging.erable.app` en ligne sur Hetzner avec Coolify
- ⬜ Pipeline CI/CD (lint, type-check, tests, sécurité, build)
- ⬜ ADR initiaux couvrant les choix techniques majeurs

---

## Phase 1 — MVP pour Ottawa (semaines 3–10)

⬜ **Planifié.**

Un premier produit utilisable, entièrement focalisé sur Ottawa, Ontario.

### Plateforme

- ⬜ Schéma de base de données et migrations Alembic
- ⬜ Authentification : inscription, connexion, JWT avec rotation
  des refresh tokens, lien magique, réinitialisation de mot de passe
- ⬜ Onboarding utilisateur (province, statut, famille, pays
  d'origine)
- ⬜ Liste de démarches personnalisée

### Fonctionnalités générant du revenu (intégrées dès le début)

- ⬜ Comparateur de comptes bancaires avec liens d'affiliation
  (RBC, Scotiabank, BMO, TD, Desjardins)
- ⬜ Comparateur de forfaits mobile/internet
  (Public Mobile, Fizz, Bell, Rogers)
- ⬜ Comparateur d'assurance santé temporaire (Manulife, Sun Life)
- ⬜ Générateur de CV format canadien (à l'unité, 29–49 $)
- ⬜ Générateur de lettre de motivation (probablement combiné avec le
  CV builder)
- ⬜ ECA Concierge — accompagnement automatisé pour la
  reconnaissance des diplômes (à l'unité, 49 $)
- ⬜ Intégration Lemon Squeezy en production

### Contenu et SEO

- ⬜ Huit pages détaillées spécifiques à Ottawa : NAS, OHIP, permis
  de conduire, compte bancaire, inscription scolaire, garderie
  subventionnée, logement, première déclaration d'impôts
- ⬜ Données structurées, sitemap, hreflang, OpenGraph
- ⬜ Composant `<DataFreshnessIndicator />` sur chaque page
  d'information

### IA et pipeline de fraîcheur

- ⬜ RAG strict avec citations obligatoires
- ⬜ Re-crawl hebdomadaire des sources officielles avec versioning
  des chunks
- ⬜ Chat IA pour les questions complexes
- ⬜ Abstraction LLM compatible Claude, OpenAI, Mistral, Ollama

### Qualité et lancement

- ⬜ Tests end-to-end des parcours critiques
- ⬜ Couverture de tests supérieure à 80 %
- ⬜ Bêta privée avec 50 testeurs
- ⬜ Lancement public sur Product Hunt, Hacker News, Reddit, Facebook

---

## Phase 2 — Premium et scaling (semaines 13–26)

⬜ **Planifié.**

Une fois le MVP livré et avec des utilisateurs réels, on ajoute les
fonctionnalités à revenu récurrent et on commence à étendre
géographiquement.

- ⬜ Analyse de documents (OCR privacy-first avec Tesseract, Claude
  Vision en repli avec consentement)
- ⬜ Suivi multi-membres famille
- ⬜ Rappels avancés multi-canal (e-mail et push web)
- ⬜ Abonnement Premium « Érable Settlement Pro » (9,99 $/mois ou
  79 $/an)
- ⬜ Plan famille (14,99 $/mois, jusqu'à 4 membres)
- ⬜ Toronto, puis le reste de l'Ontario

---

## Phase 3 — B2B et modules saisonniers (mois 6–9)

⬜ **Planifié.**

- ⬜ Label blanc B2B pour les organismes d'accueil francophones
  (CESOC, Centre francophone de Toronto, La Cité collégiale, OCISO,
  OCASI). Tarifs : 299–999 $/mois.
- ⬜ Module fiscal pour la saison de déclaration mars–avril 2027,
  avec validation par CPA partenaire (99–149 $ par déclaration)
- ⬜ Québec
- ⬜ Programme de référencement vers RCIC certifiés (commission
  10–15 %)

---

## Phase 4 — Au-delà (mois 9 et plus)

⬜ **Futur.**

- ⬜ Colombie-Britannique, Alberta, Nouveau-Brunswick
- ⬜ Application mobile native (uniquement si la PWA s'avère
  insuffisante)
- ⬜ Support de langues additionnelles (espagnol, arabe pour les
  communautés de nouveaux arrivants)
- ⬜ API de données ouvertes pour les organismes partenaires

---

## Hors périmètre

Certaines choses ne seront jamais dans Érable, par construction :

- ❄️ Conseil personnalisé en immigration (illégal sans licence RCIC)
- ❄️ Stratégie ou représentation devant IRCC
- ❄️ Prédiction des chances d'acceptation
- ❄️ Remplir des formulaires IRCC pour le compte d'un client contre
  paiement direct
