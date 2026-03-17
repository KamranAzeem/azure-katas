# SC-300 Certification Practice Repository

## Why take SC-300 exam?
The SC-300 (Identity and Access Administrator Associate) exam checks real Microsoft Entra identity and access administration skills across identity lifecycle, authentication and access controls, workload identities, and governance.

## Exam Preparation Resources
Microsoft Learn is the primary one-stop resource for SC-300 preparation.

It includes:
* Exam curriculum details
* Training content (text and official videos)
* Practice assessments
* Exam scheduling links

Important links:
* Official Microsoft Learn certification page for SC-300: https://learn.microsoft.com/en-us/credentials/certifications/identity-and-access-administrator/?practice-assessment-type=certification
* Official Study guide: https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/SC-300
* Official Microsoft training course videos: https://learn.microsoft.com/en-us/training/course-videos-on-shows
* Practical exam-day tips/notes from this repo: [docs/about-actual-exam.md](docs/about-actual-exam.md)

## Repository Purpose
- Hands-on SC-300 practice labs matched to the exam topics.
- Portal-first approach (click-path) as the primary learning path.
- PowerShell path as method #2.
- Azure CLI path as method #3.
- ARM templates where possible for supporting Azure resources.

## Scope and Limits
- Many SC-300 tasks are tenant identity tasks in Microsoft Graph and are not ARM-native.
- ARM templates in `labs/arm/` are reinforcement for supporting Azure resources.
- Some identity governance features require Microsoft Entra ID P1/P2.
- Corporate/sandbox tenants may block specific features; do the closest equivalent and note blockers.

## Estimated Cost to run the labs
- Entra-only tasks are typically low/no direct Azure resource cost.
- Workload identity and logging labs may have low cost if resources are left running.
- Use smallest SKUs and delete temporary resources immediately after validation.
- Monitor spend via **Subscriptions -> Overview** or **Azure Cost Management**.

## How to Use This Repository
- Clone this repository.
- Use `labs/` for daily labs.
- Follow each lab in this order:
  1. Direct Portal GUI Steps (Primary)
  2. PowerShell Path (Method #2)
  3. Azure CLI Path (Method #3)
  4. Optional ARM reinforcement where applicable

## Prerequisites for Labs
- A tenant where you have enough Entra admin rights.
- Azure CLI installed (`az`) and signed in.
- PowerShell 7+ with modules:
  - `Microsoft.Graph`
  - `Az.Accounts`, `Az.Resources`
- A basic Azure resource group for Azure-side practice resources (for example `rg-sc300` in `northeurope`).

### Recommended baseline setup
```bash
az login
az account show --output table
az group create --name rg-sc300 --location northeurope --output table
```

## Learning Path
- Day 1: `labs/day1/day1.md` (Identity basics: users, groups, AUs, guest access, RBAC mapping)
- Day 2: `labs/day2/day2.md` (Authentication methods, SSPR, Conditional Access, password protection)
- Day 3: `labs/day3/day3.md` (App registrations, service principals, API permissions, managed identities)
- Day 4: `labs/day4/day4.md` (PIM, access reviews, entitlement management, lifecycle workflows)
- Day 5: `labs/day5/day5.md` (Timed simulation, troubleshooting sprint, readiness and cleanup)

## SC-300 Objectives Coverage Map
| SC-300 Objective Domain | Core Labs | Optional Practice |
| --- | --- | --- |
| Implement identities in Microsoft Entra ID | `labs/day1/lab1.md`, `labs/day1/lab2.md`, `labs/day1/lab3.md`, `labs/day1/lab4.md` | `labs/optional-labs/README.md` |
| Implement authentication and access management | `labs/day2/lab1.md`, `labs/day2/lab2.md`, `labs/day2/lab3.md`, `labs/day2/lab4.md` | `labs/optional-labs/README.md` |
| Plan and implement workload identities | `labs/day3/lab1.md`, `labs/day3/lab2.md`, `labs/day3/lab3.md`, `labs/day3/lab4.md` | `labs/arm/day3/scenario01/main.json` |
| Plan and automate identity governance | `labs/day4/lab1.md`, `labs/day4/lab2.md`, `labs/day4/lab3.md`, `labs/day4/lab4.md` | `labs/optional-labs/README.md` |

### Exam Simulation
- Exam simulation flow: `labs/day5/lab1.md`, `labs/day5/lab2.md`, `labs/day5/lab3.md`

## Optional Labs
- Optional focused labs index: `labs/optional-labs/README.md`

## Quick Start
1. Clone this repository.
2. Sign in to Azure CLI: `az login`
3. Confirm active subscription: `az account show --output table`
4. Create baseline resource group (if missing): `az group create --name rg-sc300 --location northeurope --output table`
5. Open Day 1 overview: `labs/day1/day1.md`
6. Start first lab: `labs/day1/lab1.md`

## ARM Notes for SC-300
- ARM scenarios are under `labs/arm/day*/scenario*/main.json`.
- Use them for Azure resource scaffolding that supports identity exercises.
- For Entra tenant objects (users, groups, CA, governance), use Portal/Graph PowerShell/`az rest`.
