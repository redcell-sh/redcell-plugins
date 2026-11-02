---
name: cloud-attack-paths
description: Cloud attack-path playbook for authorized AWS and Azure/Entra ID environments — account and metadata enumeration, IAM role abuse, SSRF to instance metadata, privilege-escalation chains, hybrid-identity pivots (Azure AD Connect, gMSA), storage account abuse, and automation/runbook abuse. Use when the engagement includes cloud assets, cloud-connected infrastructure, or hybrid identity.
metadata:
  short-description: AWS/Azure attack-path playbook
  category: cloud
---

# Cloud Attack Paths (AWS / Azure / Entra ID)

> Authorized engagements only (see `authorized-use`): confirm the exact cloud accounts,
> subscriptions, and regions in scope — cloud tenants are shared infrastructure, so asset-level
> scope discipline is stricter than on-prem. Cloud APIs change fast; verify every technique against
> current provider documentation before use.

## 1. Enumerate the cloud surface first
- Determine provider and tenant boundaries: account IDs, subscriptions, regions, SSO endpoints,
  and any public buckets/console endpoints.
- With a foothold: environment variables, instance metadata, CLI config files, and shell history
  enumerate *which* cloud and *which* identity you already hold.

## 2. AWS patterns
- **SSRF → instance metadata**: from any SSRF (web or app), pull the instance role credentials —
  then enumerate what that role can actually do (policy simulation / `GetCallerIdentity`-style
  checks) before touching anything.
- **Privilege-escalation chains**: look for `iam:PassRole` + compute (Lambda/EC2 user-data),
  secrets-manager reads, role assumption chains, and cross-account trust policies; role chaining is
  the cloud equivalent of credential pivoting.
- **Storage**: open buckets/objects are both findings and credential sources (backups, configs,
  key material).
- CloudTrail logs everything — work minimally and document as you go.

## 3. Azure / Entra ID patterns
- Enumerate with any valid token: users, groups, role assignments (Global Administrator,
  Privileged Role holders), app registrations, and consent grants.
- **Managed identities**: instance-level IMDS yields tokens for the attached identity — check its
  role assignments before moving.
- **Automation**: runbooks/functions/apps with wide rights are the classic Azure escalation;
  storage account keys and classic credentials appear in automation assets and backups.
- **Device/consent abuse**: illicit consent grants and device-code flows are social-adjacent — only
  with explicit authorization covering user interaction.

## 4. Hybrid identity (the bridge both ways)
- **Azure AD Connect on-prem**: the sync service's credentials can write directory changes —
  equivalent to directory dominance; protect and treat as tier-0.
- **gMSA and sync service accounts**: cloud-connected services frequently run as gMSA accounts;
  their readable passwords are domain-wide pivots (see `windows-ad-attack-paths`).
- Cloud groups that grant on-prem rights (and vice versa) are the shortest chains — enumerate both
  sides of every identity.

## 5. Containers and serverless in cloud context
- Metadata → role → registry/console: a pod or function with an attached role is a cloud
  credential store (see `container-k8s-escape`).
- Exposed dashboards and control planes (with anonymous or weak auth) remain common initial
  footholds.

## 6. Validate and document
- Cloud engagements are fully audited by design (CloudTrail / Entra sign-in and audit logs):
  reproduce exactly what you did, minimize reads of sensitive data, and surface every permission
  gap as a finding, not just the final takeover.

---
*Patterns reflect standard, publicly documented cloud attack paths (AWS/Azure) as exercised in
authorized labs and public cloud-security research. Verify against current provider docs.*
