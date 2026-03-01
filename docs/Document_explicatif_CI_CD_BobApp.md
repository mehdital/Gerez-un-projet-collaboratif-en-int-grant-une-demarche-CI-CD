# BobApp - Document explicatif CI/CD

## 1. Objet du document

Ce document formalise la mise en place de la CI/CD sur BobApp (back Spring Boot Java 11 + front Angular), propose des KPI de qualite, presente les metriques obtenues, et analyse les retours utilisateurs afin de prioriser les prochaines actions.

Repository analyse: <https://github.com/mehdital/Gerez-un-projet-collaboratif-en-int-grant-une-demarche-CI-CD>

## 2. Workflows GitHub Actions mis en place

### 2.1 Workflow Tests (`.github/workflows/tests.yml`)

Declenchement:
- `pull_request` vers `main`
- `workflow_dispatch`

Objectif:
- Valider automatiquement les tests avant validation d'une PR.

Etapes principales:
- Back-end: `mvn -B clean verify` (tests + rapport JaCoCo)
- Front-end: `npm ci` puis `ng test --watch=false --code-coverage --browsers=ChromeHeadlessCI`
- Publication des artefacts de test/couverture:
  - `back/target/surefire-reports/**`
  - `back/target/site/jacoco/**`
  - `front/coverage/**`

### 2.2 Workflow Qualite Sonar (`.github/workflows/quality.yml`)

Declenchement:
- `pull_request` vers `main`
- `workflow_dispatch`

Objectif:
- Evaluer la qualite du code et bloquer la PR si le quality gate est en echec.

Etapes principales:
- Regeneration de la couverture back/front
- Scan Sonar: `SonarSource/sonarqube-scan-action`
- Verification gate: `SonarSource/sonarqube-quality-gate-action`

Pre-requis:
- Secret GitHub `SONAR_TOKEN`

### 2.3 Workflow CI/CD (`.github/workflows/ci-cd.yml`)

Declenchement:
- `push` sur `main`
- `workflow_dispatch`

Objectif:
- tests, qualite, build, puis publication Docker.

Enchainement des jobs:
- `tests` > `quality` > `build` > `docker`
- Chaque etape est bloquante pour la suivante via `needs`.

Etapes Docker:
- Login Docker Hub (`DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`)
- Build et push des images:
  - `bobapp-back`
  - `bobapp-front`

Condition de publication Docker:
- Job `docker` execute uniquement sur `push` sur `main`.

## 3. KPI proposes

Les KPI suivants sont proposes pour stabiliser la qualite et reduire les regressions:

1. Couverture de code minimale (KPI obligatoire):
- Couverture globale minimale: **70%** 
- Cible a moyen terme: **80%**

2. Qualite securite:
- **New Blocker Issues = 0**
- **New Vulnerabilities = 0**


3. Stabilite pipeline:
- **Taux de succes des PR >= 95%** (a analyser sur 30 jours)

## 4. Metriques observees

### 4.1 GitHub Actions

- Workflow `Tests` sur PR: **SUCCESS**
- Workflow `Quality (SonarQube Cloud)` sur PR: **SUCCESS** 

### 4.2 SonarCloud

Etat quality gate: **OK**

Mesures disponibles (nouveau code):
- `new_bugs`: **0**
- `new_vulnerabilities`: **0**
- `new_code_smells`: **0**
- `new_duplicated_lines_density`: **0.0%**
- `new_security_hotspots_reviewed`: **100.0%**
- `new_reliability_rating`: **A (1.0)**
- `new_security_rating`: **A (1.0)**
- `new_maintainability_rating`: **A (1.0)**

### 4.3 Couverture de tests (rapports generes localement)

Back-end (JaCoCo):
- Couverture lignes: **37.78%** (17 couvertes / 45)
- Couverture instructions: **32.84%**

Front-end (Karma/lcov):
- Couverture lignes: **83.33%** (10 / 12)
- Couverture fonctions: **57.14%** (4 / 7)

Conclusion couverture:
- Le front est correct sur les lignes.
- Le back est insuffisant par rapport au KPI cible et doit etre prioritaire.

## 5. Analyse des retours utilisateurs ("Notes et avis")

Retours identifies:
- "Impossible de poster une suggestion de blague, le navigateur plante."
- "Bug sur le post de video toujours present apres 2 semaines."
- "Pas de retour support apres plusieurs jours."
- "Suppression du site des favoris."

Lecture produit/ops:
- Presence de bugs fonctionnels visibles en production.
- Delai de correction trop long.
- Manque de communication utilisateur et de suivi incident.
- Risque de churn (perte d'utilisateurs actifs).

Priorites recommandees:
1. Corriger en urgence le bug bloquant de suggestion de blague (P1).
2. Corriger le bug video non resolu (P1/P2 selon impact).
3. Mettre en place un triage bugs hebdomadaire et SLA de reponse.
4. Ajouter des tests de non-regression sur les parcours critiques.

## 6. Plan d'actions recommande

- Ajouter tests unitaires/service back sur les endpoints critiques.
- Objectif initial: +15 a +20 points de couverture back.
- Ajouter verifications de non-regression front (component/service tests).

## 7. Liens utiles

- Repo: <https://github.com/mehdital/Gerez-un-projet-collaboratif-en-int-grant-une-demarche-CI-CD>
- PR #1: <https://github.com/mehdital/Gerez-un-projet-collaboratif-en-int-grant-une-demarche-CI-CD/pull/1>
- Run Quality: <https://github.com/mehdital/Gerez-un-projet-collaboratif-en-int-grant-une-demarche-CI-CD/actions/runs/22537507058>
- SonarCloud projet: <https://sonarcloud.io/project/overview?id=mehdital_Gerez-un-projet-collaboratif-en-int-grant-une-demarche-CI-CD>
