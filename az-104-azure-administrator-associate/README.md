# AZ-104 Certification Practice Repository

## Why take AZ-104 exam?
The AZ-104 (Microsoft Azure Administrator) exam validates practical, job-ready Azure administration skills across identity, governance, compute, storage, networking, and monitoring.
Preparing for AZ-104 helps you build a strong foundation for real-world cloud operations, not just pass an exam.

## Exam Preparation Resources
Microsoft Learn is the primary one-stop resource for AZ-104 preparation. 

It includes:
* Exam curriculum details
* Training content - both text, and complete (good quality) training videos (you don't need to hunt for AZ-104 training courses on YouTube, Udemy, etc)
* Practice-exams to test your knowledge
* Link to schedule your actual exam

Important links:
* Official Microsoft Learn certification page for Az-104: https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/?practice-assessment-type=certification
* Official Study guide: https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104 
* [Optional] Official Microsoft (fundamentals) training course AZ-900 (free) (~ 3 hours): 
  * https://youtube.com/playlist?list=PLahhVEj9XNTd6H2vpsvm_kBSMsMw9EMUL&si=XTRmVkUm41GRePSl
* Official Microsoft (full) training course videos AZ-104 (free) (~ 14 hours): 
  * https://learn.microsoft.com/en-us/training/course-videos-on-shows 
  * https://youtube.com/playlist?list=PLahhVEj9XNTcj4dwEwRHozO3xcxI_UHYG&si=zeN1tkjq0pMx7Qgk
* Official Exam readiness videos AZ-104 (shorter than the full course) - (free) (~1 hour): 
  * https://youtube.com/playlist?list=PLahhVEj9XNTcFnkx03XSK9TUkhzG3YTS_&si=1dxB0qhE3NHVRiQ_ 
* Practical exam-day tips/notes from this repo: [docs/about-actual-exam.md](docs/about-actual-exam.md)


## Repository Purpose
- Hands-on Azure AZ-104 practice labs aligned with curriculum study workflow.
- Portal-first approach (using click-path)
- PowerShell and Azure CLI instructions to perform the same tasks on the command line instead of Portal WebUI (click-path)
- Validation using PowerShell and Azure CLI
- ARM template to perform various tasks using IaC approach

## Scope and Constraints
- ARM templates are optional reinforcement and not required for every lab step.
- Some identity features (for example dynamic group membership) can require Microsoft Entra ID P1/P2 - which may require upgrading your payment plan.
- Corporate or sandbox subscriptions may block specific actions. When blocked, complete the closest equivalent task and make note of the limitations encountered.

## Estimated Cost to run the labs
- Cost is typically very low if resources are cleaned up (deleted) at the end of each lab. (I used a total of less than 10 NOK while preparing for this exam)
- Actual cost depends on subscription type, region, runtime duration, and any resources left running. 
- To control cost, use the smallest supported SKUs and delete temporary resources immediately after validation. 
- Keep a watchful eye on on going cost using:
  * Subscriptions -> [Your Subscription] -> Overview
  * Or, "Azure Cost Management"

## How to Use This Repository
- Clone this repository to your local computer.
- The `labs/` directory contains all labs.
- Labs are focused on Azure Portal execution through clear click-path instructions.
- Azure CLI commands are included for validation and troubleshooting.
- You can use `bash`, `PowerShell`, or Azure Cloud Shell.

## Prerequisites for Labs
- A Microsoft account that can sign in to Azure Portal.
- Azure CLI installed locally (`az`) if you plan to run CLI validation commands.
- If you are using a corporate account, then naturally you will be using Azure through your corporate (Microsoft) account. In that case, you will need access to some sort of "sandbox" environment, normally a separate Azure subscription. Your manager, or someone in the company will get this arranged for you. You may be limited by some restrictions against your user-account which will prevent you from doing few things in the IAM section. You should be able to do most of the labs.
- If you experience problems using corporate account, then simply use your personal account for your personal subscription in the Azure Portal, preferably in a different browser than what you use for your office work, or at least create a new browser profile, so you do not confuse between Azure Portal for work, and Azure portal being used for learning. This (using personal account) is much better, and will save you a lot of time.
- If you are using a personal Microsoft account, then you will most probably get a free tier in Azure. The free tier is sufficient to cover 98% of Microsoft exam objectives. There are a few edge cases such as experimenting with Entra ID P1/P2 plans, dynamic user/group subscriptions, etc. You should at least study those items even if you cannot practice them.
- Use one common/baseline resource group for consistency (for example `rg-az104`).
- Use one consistent region for repeatability (for example: `northeurope` / `North Europe`).

### Important note about free subscription:
If you have a personal subscription then you should know that the free credits (worth about 200 USD) will expire within 30 days. In any case, you will lose ability to use any Azure resources if your **Free** account expires. You may even be able to login to Azure Portal. This is important to understand. So if you created a MS account just to practice for Azure exam, and later you forget about it later, then you you will lose ability to use that account again - depending on how much time has passed, and if Microsoft wants to help you or not. 

Therefore, **upgrade** to **Basic** (pay-as-you-go) pricing plan before the 30 day period (or before the free credits are consumed), for continued access to Microsoft cloud.

## Learning Path
- Day 1: `labs/day1/day1.md` (Identity/RBAC baseline, storage baseline, governance starter)
- Day 2: `labs/day2/day2.md` (Compute baseline, segmentation, load balancing, compute/network gap closure)
- Day 3: `labs/day3/day3.md` (Storage security/lifecycle, Azure Files, backup starter)
- Day 4: `labs/day4/day4.md` (Policy/tags, monitoring/alerts, automation, governance controls)
- Day 5: `labs/day5/day5.md` (Timed simulation rounds and readiness scoring)

## AZ-104 Objectives Coverage Map
| AZ-104 Objective Domain | Core Labs | Optional Reinforcement |
| --- | --- | --- |
| Manage Azure identities and governance | `labs/day1/lab1.md`, `labs/day1/lab4.md`, `labs/day4/lab1.md`, `labs/day4/lab4.md` | `labs/optional-labs/lab-sspr-review.md` |
| Implement and manage storage | `labs/day1/lab2.md`, `labs/day3/lab1.md`, `labs/day3/lab2.md` | `labs/optional-labs/lab-storage-access-security-gaps.md` |
| Deploy and manage Azure compute resources | `labs/day2/lab1.md`, `labs/day2/lab4.md` | `labs/optional-labs/lab-compute-app-platform-gaps.md` |
| Implement and manage virtual networking | `labs/day2/lab2.md`, `labs/day2/lab3.md`, `labs/day2/lab5.md` | `labs/optional-labs/lab-hub-and-spoke.md`, `labs/optional-labs/lab-expressroute-design.md`, `labs/optional-labs/lab-advanced-networking-gaps.md`, `labs/optional-labs/lab-private-dns-zone-vnet-link.md` |
| Monitor and maintain Azure resources | `labs/day3/lab3.md`, `labs/day4/lab2.md`, `labs/day4/lab3.md` | `labs/optional-labs/lab-monitor-backup-dr-gaps.md` |

### Exam Simulation
- Exam readiness simulation flow: `labs/day5/lab1.md`, `labs/day5/lab2.md`, `labs/day5/lab3.md`

## Optional Labs
- Optional focused labs: `labs/optional-labs/`

## Quick Start
1. Clone this repository.
2. Sign in to Azure CLI: `az login`
3. Confirm your active subscription: `az account show --output table`
4. Create baseline resource group (if missing): `az group create --name rg-az104 --location northeurope --output table`
5. Open Day 1 overview: `labs/day1/day1.md`
6. Start first lab: `labs/day1/lab1.md`


---

## Do you want to use AI with this repository?

(A note about AGENTS.md and the AI workflow)

The `AGENTS.md` is the root level file, that has nothing to do (directly) with the contents of this repository, and can safely be ignored by the user merely using this repository for exam preparation. 

The `AGENTS.md` is used by the developer/author of this product/project to interact with any of the AI tools available - either inside the code editor (e.g. VSCode), or outside the editor directly on CLI.

These AI assistants could be any of the following:
* GitHub Copilot
* Gemini
* ChatGPT
* Claude
* DeepSeek
* ,etc

Yes, it is safe to commit the AGENTS.md file to git, as it does not contain anything sensitive. 

**NOTE:** You must make sure that you do not put anything sensitive in it!

If, you - the candidate preparing for the exam - want to use AI assistants while going through the labs in this repo, then it is do-able, and is very easy.

The steps would be:
* Clone this repository to your local computer (`git clone`)
* Create the following directory structure, which is already ignored in `.gitignore` , so you are safe! :)
* Lastly create `ai/ai-policy.md` file, which is mandatory.
* Adjust the root-level `AGENTS.md` to point to correct files.
* In the AI assistant (e.g. "Copilot chat", "Gemini CLI"), run the `/init` command, so the AI assistant can read all files and builds it's context about this repository, and creates necessary files under the `ai` directory.

Once you have setup like this, you can safely use any AI assistant of your choice with this repository, without the fear of leaking any personal thinking patterns, reasoning, etc, through this git repository.

### Directory structure for `ai/`

```
<repository-root-directory>
|
|-- ai
|    |-- ai-policy.md
|    |-- context.md
|    |-- next-steps.md
|    |-- progress.md
|    |-- daily-checkpoints
|        |-- 2026-03-13.md
|
...
```

Basically you need to create just one project-root-level directory named `ai`, and only one file inside it named `ai-policy.md` . Rest of it will be built and maintained by the AI assistant - following instructions in the root-level `AGENTS.md` file.
