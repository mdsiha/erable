# Politique de confidentialité d'Érable

> *English version below — [Privacy Policy](#privacy-policy).*

**Dernière mise à jour : 1er mai 2026**
**Version : 1.0**

> **En une phrase : nous ne possédons pas vos documents, vous nous les
> prêtez le temps d'une analyse. Et on ne garde rien.**

Érable est un projet bâti par et pour des nouveaux arrivants
francophones. Nous savons que confier ses documents personnels — un
permis de conduire, une lettre d'IRCC, un contrat de bail — à une
application qu'on ne connaît pas demande de la confiance. Cette
politique explique ce qu'on fait avec vos données, en français simple,
sans jargon juridique inutile. Le principe directeur : **on garde le
strict minimum, on supprime tout le reste.**

---

## 1. Ce qu'on collecte (et ce qu'on ne collecte pas)

### Quand vous créez un compte

On stocke votre adresse e-mail, votre mot de passe (chiffré, jamais en
clair), votre langue préférée (FR ou EN), et la date de création de
votre compte. C'est tout.

### Quand vous remplissez votre profil

On stocke uniquement les informations nécessaires pour personnaliser
votre liste de démarches : votre province d'arrivée, votre statut
d'immigration (résident permanent, étudiant, etc.), votre pays
d'origine, votre date d'arrivée approximative, et la composition de
votre famille (conjoint, enfants).

On **ne demande pas** votre numéro d'assurance sociale, votre numéro
de passeport, votre adresse exacte ou votre statut financier.

### Quand vous téléversez un document pour analyse

C'est ici que notre engagement le plus fort s'applique. Quand vous
nous envoyez une photo ou un PDF d'un document à analyser :

1. Le fichier est traité en mémoire vive, dans un processus isolé.
2. Notre moteur OCR local (Tesseract) extrait le texte.
3. L'IA produit un résumé en français simple.
4. **Le fichier est immédiatement détruit.** Il n'est jamais écrit
   sur un disque, jamais copié dans une base de données, jamais
   sauvegardé.
5. Le contenu textuel du document n'est pas non plus conservé. Seul
   un compteur d'usage est gardé pour appliquer les limites du plan
   Premium.

Pour les cas où Tesseract ne suffit pas (écriture manuscrite, scans de
mauvaise qualité), on peut utiliser Claude Vision d'Anthropic. Dans
ce cas-là, on vous demande explicitement votre consentement, document
par document. Si vous refusez, on n'envoie rien à un service externe.

### Quand vous cliquez sur un lien d'affiliation

On enregistre que **vous** (en tant qu'utilisateur authentifié) avez
cliqué sur l'offre d'un partenaire (par exemple, le programme RBC pour
les nouveaux arrivants). On ne suit pas les visiteurs anonymes, on ne
plante pas de cookies tiers, on n'utilise pas d'empreinte numérique
("fingerprinting") pour vous reconnaître hors de votre session.

### Ce qu'on ne collecte jamais

- Vos numéros gouvernementaux (NAS, passeport, permis)
- Votre activité hors d'Érable (pas de tracker tiers, pas de pixels Facebook ou Google Ads)
- Vos contacts ou votre carnet d'adresses
- Vos coordonnées de paiement (gérées exclusivement par notre
  prestataire de paiement, voir section 3)

---

## 2. Pourquoi on collecte ces données

Trois raisons, et seulement trois :

1. **Vous fournir le service** : générer votre liste de démarches,
   répondre à vos questions, envoyer les rappels que vous avez
   demandés.
2. **Améliorer le produit** : comprendre quelles démarches sont les
   plus consultées, quels comparateurs sont utilisés, où les
   utilisateurs décrochent. Les données sont **agrégées et anonymisées**
   pour ces analyses.
3. **Respecter nos obligations légales** : si une autorité canadienne
   nous demande légalement de produire des données dans le cadre
   d'une enquête, on s'y conforme — et seulement dans ce cas.

On ne revend **jamais** vos données. On ne les partage pas avec des
courtiers en données. On n'envoie pas de pubs ciblées.

---

## 3. Avec qui on partage des données (sous-traitants)

Pour faire fonctionner Érable, on utilise quelques services tiers.
Chacun a accès au strict minimum nécessaire pour son rôle.

| Service | Rôle | Données partagées | Localisation | Note |
|---|---|---|---|---|
| Hetzner | Hébergement de l'application | Toutes les données utilisateur (sauf documents, jamais persistés) | Allemagne ou Canada (au choix) | — |
| Cloudflare | DNS, CDN, protection anti-attaque | Adresses IP, métadonnées de requête | Réseau mondial | — |
| Anthropic (Claude) | Génération de réponses IA | Texte de votre question, contexte récupéré du RAG | États-Unis | Anthropic n'utilise pas vos données pour entraîner ses modèles ([source](https://www.anthropic.com/legal/commercial-terms)) |
| Lemon Squeezy | Traitement des paiements | E-mail, montant, produit acheté | États-Unis (Merchant of Record) | — |
| Resend | E-mails transactionnels (rappels, confirmations) | E-mail, contenu du message | États-Unis | — |
| Sentry | Détection d'erreurs techniques | Traces d'erreurs, identifiant utilisateur anonymisé | États-Unis | — |

On choisit ces fournisseurs en fonction de leur sérieux sur la
confidentialité, et on documente publiquement nos choix dans nos ADR
(*Architecture Decision Records*).

---

## 4. Combien de temps on garde vos données

| Type de donnée | Durée de conservation |
|---|---|
| Compte utilisateur (e-mail, profil) | Tant que le compte est actif |
| Compte supprimé | Soft-delete pendant 30 jours, puis effacement définitif |
| Documents téléversés | **Aucun** — détruits immédiatement après analyse |
| Historique de chat IA | Tant que le compte est actif (vous pouvez supprimer une conversation à tout moment) |
| Logs techniques (Sentry, traces) | 30 jours maximum |
| Logs d'audit (sécurité) | 12 mois |
| Données de paiement | Conservées par Lemon Squeezy selon leur propre politique |

Si vous fermez votre compte, l'effacement complet a lieu sous 30
jours. Vous pouvez aussi demander une copie de toutes vos données ou
leur effacement immédiat à tout moment (voir section 6).

---

## 5. Comment vos données sont protégées

- **En transit** : toutes les communications entre votre navigateur
  et nos serveurs sont chiffrées avec TLS 1.3.
- **Au repos** : la base de données est chiffrée avec AES-256.
- **Mots de passe** : hachés avec Argon2id (algorithme recommandé en
  2026).
- **Code public** : le code source d'Érable est ouvert sur GitHub.
  Vous pouvez vérifier vous-même comment vos données sont traitées.
- **Sauvegardes** : chiffrées, conservées 30 jours, hébergées dans
  l'Union européenne.
- **Accès interne** : un seul mainteneur (Michael Das) a accès à
  l'infrastructure de production. Tous les accès sont journalisés.

---

## 6. Vos droits

En vertu de la loi canadienne sur la protection des renseignements
personnels et les documents électroniques (LPRPDE / PIPEDA) et de la
Loi 25 du Québec, vous avez le droit de :

1. **Accéder** à vos données : on vous fournit une copie complète
   sous 30 jours.
2. **Corriger** des données inexactes.
3. **Supprimer** votre compte et toutes les données associées.
4. **Exporter** vos données dans un format lisible (JSON).
5. **Vous opposer** à un traitement (par exemple, refuser les
   analyses agrégées).
6. **Retirer votre consentement** à tout moment pour les traitements
   basés sur le consentement.

Pour exercer ces droits, écrivez-nous à `privacy@erable.app`. On
répond sous 30 jours.

Vous pouvez aussi déposer une plainte auprès du Commissariat à la
protection de la vie privée du Canada
([priv.gc.ca](https://www.priv.gc.ca)) ou de la Commission d'accès à
l'information du Québec ([cai.gouv.qc.ca](https://www.cai.gouv.qc.ca))
si vous estimez que nous n'avons pas respecté vos droits.

---

## 7. Cookies et technologies similaires

Érable utilise un nombre minimal de cookies, tous fonctionnels :

- **Cookie de session** : indispensable pour vous garder connecté.
- **Cookie de préférence linguistique** : pour se souvenir de votre
  choix FR ou EN.
- **Cookie de protection CSRF** : pour empêcher certaines attaques.

Aucun cookie publicitaire, aucun cookie tiers, aucun pixel de
tracking. Pas besoin de bannière "acceptez-vous les cookies ?" parce
qu'on n'utilise que le strict nécessaire au bon fonctionnement.

---

## 8. Mineurs

Érable s'adresse à des adultes en âge de gérer leurs propres démarches
administratives. Si vous avez moins de 18 ans et avez créé un compte,
ou si vous êtes un parent qui découvre que votre enfant mineur a créé
un compte, écrivez à `privacy@erable.app` et on supprimera tout sous
48 heures.

---

## 9. Transferts internationaux

Certains de nos sous-traitants (Anthropic, Lemon Squeezy, Resend,
Sentry) sont basés aux États-Unis. Vos données peuvent donc transiter
hors du Canada lors de leur traitement. On choisit ces fournisseurs
parce qu'ils offrent des garanties contractuelles équivalentes aux
standards canadiens, et on documente cette analyse publiquement.

Si cette information est importante pour vous (par exemple si vous
êtes au Québec et que vous préférez que vos données restent au
Canada), écrivez-nous : on travaille à proposer des alternatives
canadiennes pour les fonctionnalités où c'est techniquement possible.

---

## 10. Modifications de cette politique

Si on modifie cette politique, on vous prévient :

- Par e-mail, 14 jours avant l'entrée en vigueur, pour les changements
  significatifs.
- Sur la page d'accueil, pour les changements mineurs.

L'historique complet des versions est disponible publiquement dans le
dépôt GitHub : chaque modification est un commit traçable.

---

## 11. Nous contacter

Pour toute question sur la confidentialité :

- **E-mail** : `privacy@erable.app`
- **Issue GitHub publique** (pour les questions générales) :
  [github.com/mdsiha/erable/issues](https://github.com/mdsiha/erable/issues)

Responsable du traitement : Michael Das, lui-même nouvel arrivant
francophone, opérant Érable depuis la Turquie en attendant son
installation à Ottawa fin 2026. À cette date, l'opération sera
transférée vers une entité canadienne basée à Ottawa, et cette
politique sera mise à jour pour refléter ce changement.

---
---

# Privacy Policy

**Last updated: May 1, 2026**
**Version: 1.0**

> **In one sentence: we don't own your documents — you lend them to us
> for the time of an analysis. And we keep nothing.**

Erable is a project built by and for francophone newcomers. We
understand that trusting an unknown app with your personal documents —
a driver's licence, an IRCC letter, a lease — requires confidence.
This policy explains what we do with your data, in plain language,
without unnecessary legalese. The guiding principle: **we keep the
strict minimum, we delete everything else.**

---

## 1. What we collect (and what we don't)

### When you create an account

We store your e-mail address, your password (hashed, never in clear),
your language preference (FR or EN), and the creation date of your
account. That's it.

### When you fill in your profile

We store only the information needed to personalize your checklist of
administrative steps: your arrival province, your immigration status
(permanent resident, student, etc.), your country of origin, your
approximate arrival date, and your family composition (spouse,
children).

We **do not ask** for your social insurance number, passport number,
exact address, or financial situation.

### When you upload a document for analysis

This is where our strongest commitment applies. When you send us a
photo or PDF of a document for analysis:

1. The file is processed in RAM, in an isolated process.
2. Our local OCR engine (Tesseract) extracts the text.
3. The AI produces a plain-language summary.
4. **The file is immediately destroyed.** It is never written to
   disk, never copied to a database, never backed up.
5. The text content of the document is also not retained. Only a
   usage counter is kept to enforce Premium plan limits.

For cases where Tesseract is not enough (handwriting, low-quality
scans), we may use Anthropic's Claude Vision. In that case, we ask
for your explicit per-document consent. If you decline, nothing is
sent to an external service.

### When you click an affiliate link

We record that **you** (as an authenticated user) clicked on a
partner's offer (for example, RBC's newcomer program). We do not
track anonymous visitors, do not set third-party cookies, do not use
fingerprinting to recognize you outside your session.

### What we never collect

- Government numbers (SIN, passport, licence)
- Your activity outside of Erable (no third-party trackers, no Facebook
  or Google Ads pixels)
- Your contacts or address book
- Your payment details (handled exclusively by our payment processor,
  see section 3)

---

## 2. Why we collect this data

Three reasons, and only three:

1. **To deliver the service**: generate your checklist, answer your
   questions, send the reminders you have requested.
2. **To improve the product**: understand which steps are most
   consulted, which comparators are used, where users drop off. This
   data is **aggregated and anonymized** for these analyses.
3. **To comply with legal obligations**: if a Canadian authority
   lawfully requests data as part of an investigation, we comply —
   and only in that case.

We **never** resell your data. We do not share it with data brokers.
We do not run targeted advertising.

---

## 3. Who we share data with (subprocessors)

To run Erable, we rely on a small number of third-party services.
Each has access only to the minimum required for its role.

| Service | Role | Data shared | Location | Note |
|---|---|---|---|---|
| Hetzner | Application hosting | All user data (except documents, never persisted) | Germany or Canada (selectable) | — |
| Cloudflare | DNS, CDN, attack protection | IP addresses, request metadata | Global network | — |
| Anthropic (Claude) | AI response generation | Question text, RAG-retrieved context | United States | Anthropic does not use your data to train its models ([source](https://www.anthropic.com/legal/commercial-terms)) |
| Lemon Squeezy | Payment processing | E-mail, amount, product purchased | United States (Merchant of Record) | — |
| Resend | Transactional e-mails (reminders, confirmations) | E-mail, message content | United States | — |
| Sentry | Technical error monitoring | Error traces, anonymized user identifier | United States | — |

We pick these vendors based on their privacy track record and we
document our choices publicly in our ADRs (*Architecture Decision
Records*).

---

## 4. How long we keep your data

| Data type | Retention period |
|---|---|
| User account (e-mail, profile) | While the account is active |
| Deleted account | Soft-delete for 30 days, then permanent erasure |
| Uploaded documents | **None** — destroyed immediately after analysis |
| AI chat history | While the account is active (you can delete any conversation at any time) |
| Technical logs (Sentry, traces) | 30 days maximum |
| Audit logs (security) | 12 months |
| Payment data | Retained by Lemon Squeezy under their own policy |

If you close your account, full erasure happens within 30 days. You
can also request a full data export or immediate erasure at any time
(see section 6).

---

## 5. How your data is protected

- **In transit**: all communication between your browser and our
  servers is encrypted with TLS 1.3.
- **At rest**: the database is encrypted with AES-256.
- **Passwords**: hashed with Argon2id (recommended algorithm in 2026).
- **Public source**: Erable's source code is open on GitHub. You can
  audit yourself how your data is handled.
- **Backups**: encrypted, retained for 30 days, hosted in the
  European Union.
- **Internal access**: a single maintainer (Michael Das) has access
  to production infrastructure. All access is logged.

---

## 6. Your rights

Under Canada's Personal Information Protection and Electronic
Documents Act (PIPEDA) and Quebec's Loi 25, you have the right to:

1. **Access** your data: we provide a complete copy within 30 days.
2. **Correct** inaccurate data.
3. **Delete** your account and all associated data.
4. **Export** your data in a readable format (JSON).
5. **Object** to processing (e.g., refuse aggregated analytics).
6. **Withdraw consent** at any time for consent-based processing.

To exercise these rights, write to `privacy@erable.app`. We respond
within 30 days.

You may also file a complaint with the Office of the Privacy
Commissioner of Canada ([priv.gc.ca](https://www.priv.gc.ca)) or the
Commission d'accès à l'information du Québec
([cai.gouv.qc.ca](https://www.cai.gouv.qc.ca)) if you believe we have
not respected your rights.

---

## 7. Cookies and similar technologies

Erable uses a minimal set of cookies, all functional:

- **Session cookie**: required to keep you logged in.
- **Language preference cookie**: to remember your FR or EN choice.
- **CSRF protection cookie**: to prevent certain attacks.

No advertising cookies, no third-party cookies, no tracking pixels.
No need for an "accept cookies?" banner because we only use what is
strictly necessary for the site to work.

---

## 8. Minors

Erable is intended for adults old enough to manage their own
administrative procedures. If you are under 18 and created an
account, or if you are a parent who discovers that your minor child
created an account, write to `privacy@erable.app` and we will delete
everything within 48 hours.

---

## 9. International transfers

Some of our subprocessors (Anthropic, Lemon Squeezy, Resend, Sentry)
are based in the United States. Your data may therefore transit
outside Canada during processing. We pick these vendors because they
offer contractual guarantees equivalent to Canadian standards, and we
document this analysis publicly.

If this matters to you (for example, if you are in Quebec and prefer
your data to stay in Canada), write to us: we are working on
Canadian-based alternatives where technically possible.

---

## 10. Changes to this policy

If we change this policy, we notify you:

- By e-mail, 14 days before the effective date, for significant
  changes.
- On the home page, for minor changes.

The full version history is publicly available in the GitHub
repository: every change is a traceable commit.

---

## 11. Contact us

For any privacy question:

- **E-mail**: `privacy@erable.app`
- **Public GitHub issue** (for general questions):
  [github.com/mdsiha/erable/issues](https://github.com/mdsiha/erable/issues)

Data controller: Michael Das, himself a francophone newcomer,
operating Erable from Turkey while preparing his move to Ottawa in
late 2026. At that point, operations will move to a Canadian entity
based in Ottawa, and this policy will be updated to reflect that
change.
