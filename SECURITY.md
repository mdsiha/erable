# Security Policy

> *Version française disponible plus bas — [Politique de sécurité](#politique-de-sécurité).*

## Reporting a vulnerability

If you discover a security vulnerability in Erable, please report it
privately. **Do not open a public GitHub issue.**

Send an email to `security@erable.app` with:

- A description of the vulnerability
- Steps to reproduce
- The potential impact (data exposure, account takeover, etc.)
- Optionally, a suggested fix

You will receive an acknowledgement within 48 hours. We will keep you
updated as we investigate and remediate.

## Scope

The following are in scope for security reports:

- The Erable web application (production and staging)
- The backend API
- Authentication and authorization flows
- Data handling (especially document analysis and PII)
- Affiliate redirection mechanisms

The following are **not** in scope:

- Third-party services we integrate with (Lemon Squeezy, Anthropic,
  Hetzner, Cloudflare, etc.) — please report directly to them
- Vulnerabilities requiring physical access to a user's device
- Social engineering attacks
- Denial-of-service attacks
- Issues already publicly known or already reported

## What we ask

- Give us reasonable time to fix the issue before any public
  disclosure (typically 90 days, less for critical issues)
- Do not access, modify, or delete data that does not belong to you
- Do not run automated scanners against production without first
  contacting us
- Act in good faith — we respond in kind

## What we offer

Erable is a solo-maintained project with limited resources. We do not
currently run a paid bug bounty program, but we will:

- Acknowledge your contribution publicly (with your permission) in
  the changelog and release notes
- Add your name to a `SECURITY-HALL-OF-FAME.md` file once we receive
  the first valid report
- Provide a written statement of the disclosure that you can use in
  professional contexts

## Maintained versions

Only the `main` branch and the latest production deployment receive
security fixes. Older deployments are not patched.

---
---

# Politique de sécurité

## Signaler une vulnérabilité

Si vous découvrez une faille de sécurité dans Érable, signalez-la en
privé. **N'ouvrez pas une issue GitHub publique.**

Envoyez un e-mail à `security@erable.app` avec :

- Une description de la vulnérabilité
- Les étapes pour la reproduire
- L'impact potentiel (exposition de données, compromission de
  compte, etc.)
- Optionnellement, une proposition de correctif

Vous recevrez un accusé de réception sous 48 heures. Nous vous
tiendrons informé pendant l'enquête et la correction.

## Périmètre

Les éléments suivants sont concernés par les signalements :

- L'application web Érable (production et staging)
- L'API backend
- Les flux d'authentification et d'autorisation
- Le traitement des données (en particulier l'analyse de documents
  et les renseignements personnels)
- Les mécanismes de redirection d'affiliation

Les éléments suivants ne sont **pas** concernés :

- Les services tiers que nous intégrons (Lemon Squeezy, Anthropic,
  Hetzner, Cloudflare, etc.) — signalez directement à ces services
- Les vulnérabilités qui exigent un accès physique à l'appareil de
  l'utilisateur
- Les attaques par ingénierie sociale
- Les attaques par déni de service
- Les problèmes déjà connus publiquement ou déjà signalés

## Ce qu'on vous demande

- Nous laisser un délai raisonnable pour corriger le problème avant
  toute divulgation publique (typiquement 90 jours, moins pour les
  failles critiques)
- Ne pas accéder, modifier ou supprimer des données qui ne vous
  appartiennent pas
- Ne pas lancer de scanners automatiques contre la production sans
  nous contacter au préalable
- Agir de bonne foi — nous répondons de la même manière

## Ce qu'on vous offre

Érable est un projet maintenu par une seule personne avec des
ressources limitées. Nous ne proposons pas de programme de bug bounty
rémunéré pour l'instant, mais nous nous engageons à :

- Reconnaître publiquement votre contribution (avec votre accord) dans
  le changelog et les notes de version
- Ajouter votre nom à un fichier `SECURITY-HALL-OF-FAME.md` dès
  réception du premier signalement valide
- Fournir une attestation écrite de la divulgation que vous pouvez
  utiliser dans un contexte professionnel

## Versions maintenues

Seules la branche `main` et le dernier déploiement de production
reçoivent les correctifs de sécurité. Les déploiements antérieurs ne
sont pas patchés.
