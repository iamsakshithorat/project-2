# Centralized CI/CD Platform using Shared Jenkins Infrastructure

## 📌 Project Overview

This project implements a **centralized CI/CD platform** using a shared Jenkins server, designed to replace the earlier setup where every development team maintained its own Jenkins instance. The old approach caused duplicate infrastructure, inconsistent pipelines, and security misconfigurations.

The new platform provides:
- A single, shared Jenkins server for all applications
- A **shared pipeline library** that standardizes the Build → Test → Scan → Deploy workflow
- Onboarding of multiple applications using the exact same pipeline logic
- Role-Based Access Control (RBAC) to separate Developer and Admin permissions

---

## 🏗️ Architecture

```
                        ┌───────────────────────┐
                        │     Jenkins Master     │
                        │      (on AWS EC2)      │
                        └───────────┬───────────┘
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          │                         │                         │
┌─────────▼─────────┐   ┌───────────▼───────────┐   ┌─────────▼─────────┐
│  GitHub: App-1     │   │ GitHub: Shared Library │   │  GitHub: App-2     │
│  (Jenkinsfile)     │   │  (vars/ pipeline steps)│   │  (Jenkinsfile)     │
└────────────────────┘   └───────────┬───────────┘   └────────────────────┘
                                      │
                          ┌───────────▼───────────┐
                          │   Static Agent/Node    │
                          │  Build → Test → Scan   │
                          │        → Deploy        │
                          └───────────┬───────────┘
                                      │
                          ┌───────────▼───────────┐
                          │  Role-Based Access     │
                          │  Developer | Admin     │
                          └────────────────────────┘
```

**Components:**
- **Jenkins Master** — deployed on an AWS EC2 instance, acts as the single control point for all pipelines
- **Shared Library repo** — a standalone GitHub repository containing reusable pipeline logic (`vars/*.groovy`)
- **Application repos** — each app repo contains only a thin `Jenkinsfile` that imports and calls the shared library
- **Built-in Node/Agent** — executes the actual pipeline stages
- **RBAC** — restricts what Developers vs Admins can do on the platform

---

## 📚 Shared Library Design

Repository: `jenkins-shared-library`

The shared library follows Jenkins' standard `vars/` convention, where each file defines a reusable pipeline step:

| File | Purpose |
|---|---|
| `vars/buildStage.groovy` | Standardizes the build step (dependency install + packaging) |
| `vars/testStage.groovy` | Standardizes running unit and integration tests |
| `vars/scanStage.groovy` | Standardizes security/vulnerability scanning |
| `vars/deployStage.groovy` | Standardizes deployment to the target environment |

**Why this design?**
- Every application calls the same four functions (`buildStage()`, `testStage()`, `scanStage()`, `deployStage()`), so pipeline **behavior is guaranteed to be consistent** across teams.
- Any future change to build/test/scan/deploy logic needs to be updated in **one place** (the shared library) instead of in every application's Jenkinsfile.
- New teams can onboard by writing a 15-line Jenkinsfile instead of a full custom pipeline.

The library is registered in Jenkins under **Manage Jenkins → System → Global Trusted Pipeline Libraries** with the name `shared-lib`, pointing to the GitHub repository and using stored Git credentials.

---

## 🔗 Multi-Application Onboarding

Two sample applications were onboarded to prove the platform supports multiple projects using identical shared logic:

- **App-1** → repo: `app-1-repo`
- **App-2** → repo: `app-2-repo`

Each application's `Jenkinsfile` follows the same pattern:

```groovy
@Library('shared-lib') _

pipeline {
    agent any
    stages {
        stage('Build')  { steps { buildStage('AppName') } }
        stage('Test')   { steps { testStage('AppName') } }
        stage('Scan')   { steps { scanStage('AppName') } }
        stage('Deploy') { steps { deployStage('AppName') } }
    }
}
```

Both pipelines were run in Jenkins and completed with `Finished: SUCCESS`, confirming that the shared library correctly standardizes behavior across independent applications. Screenshots of both successful builds are included in the deliverables.

---

## 🔐 Access Control (RBAC)

Configured using the **Role-based Authorization Strategy** plugin.

| Role | Permissions |
|---|---|
| **Admin** | Full control — `Administer` (create/delete jobs, manage credentials, configure system, manage users) |
| **Developer** | Limited — `Overall/Read`, `Job/Build`, `Job/Cancel`, `Run/Read` (can view and trigger pipelines, cannot change system configuration or credentials) |

This enforces the **principle of least privilege**: developers can do their day-to-day work (triggering and viewing builds) without being able to alter security-sensitive settings.

---

## ⚠️ Risk Mitigation & Governance Benefits

| Problem (Before) | Solution (After) |
|---|---|
| Every team ran its own Jenkins → duplicate infra & cost | Single shared Jenkins server for all teams |
| Inconsistent pipeline steps across teams | Shared library enforces identical Build→Test→Scan→Deploy flow |
| Security misconfigurations per-server | Centralized credential management + RBAC |
| No standard onboarding process for new apps | New apps onboard with a ~15 line Jenkinsfile |
| No control over who can change pipeline/system config | Developer vs Admin roles restrict sensitive actions |

---

## 🛠️ Technologies & Tools

- **Jenkins** (2.568.2) — CI/CD orchestration
- **AWS EC2** — Jenkins host infrastructure
- **GitHub** — source control for shared library and application repos
- **Groovy** — shared library pipeline scripting
- **Role-based Authorization Strategy plugin** — access control

---

## 📁 Repository Links

- Shared Library: `https://github.com/iamsakshithorat/jenkins-shared-library`
- Sample App 1: `https://github.com/iamsakshithorat/app-1-repo`
- Sample App 2: `https://github.com/iamsakshithorat/app-2-repo`

---

## 📸 Deliverables Included

- [x] Jenkins shared library repository (4 groovy stage files)
- [x] Sample pipelines for 2 applications, both using the shared library
- [x] Screenshots of both pipelines running successfully (multi-project proof)
- [x] Role-based access control configuration (Developer & Admin roles)
- [x] This README explaining the platform design

---




## 👤 Author

**Sakshi Thorat**
