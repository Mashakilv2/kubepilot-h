# KubePilot AI — Helm Charts 🤖

Official Helm chart repository for **KubePilot AI** — the AI-powered Kubernetes & OpenShift operations platform.

> Stop getting paged at 2am. KubePilot detects incidents, explains them in plain English, and proposes fixes — waiting for your approval before taking any action.

---

## What Is KubePilot AI?

KubePilot AI watches your cluster 24/7 and:

- 🔍 **Detects** CrashLoopBackOff, OOMKill, Pending pods, Node issues
- 🤖 **Analyzes** each incident using Claude AI in plain English
- 📨 **Alerts** your Slack channel with a rich formatted message
- ✅ **Waits** for your approval before executing any fix
- 🔧 **Executes** safe remediation via the Kubernetes API
- 📋 **Records** every action in a versioned journal with rollback
- 📊 **Sends** weekly operations reports to Slack and email

---

## Quick Install

### Step 1 — Add The Helm Repo

```bash
helm repo add kubepilot https://mashakilv2.github.io/kubepilot-h
helm repo update
```

### Step 2 — Create Your Secrets

```bash
kubectl create namespace kubepilot

kubectl create secret generic kubepilot-secrets \
  --namespace kubepilot \
  --from-literal=anthropic-api-key=sk-ant-xxxxx \
  --from-literal=slack-bot-token=xoxb-xxxxx \
  --from-literal=database-url=postgresql://kubepilot:kubepilot@kubepilot-postgres:5432/kubepilot
```

### Step 3 — Install KubePilot

```bash
helm install kubepilot kubepilot/kubepilot \
  --namespace kubepilot \
  --set kubepilot.clusterName=my-cluster \
  --set slack.channelId=C0XXXXXXXXX
```

### Step 4 — Verify Installation

```bash
kubectl get pods -n kubepilot
```

You should see:
```
NAME                         READY   STATUS    RESTARTS   AGE
kubepilot-xxxxxxxxx-xxxxx    1/1     Running   0          30s
```

---

## Prerequisites

- Kubernetes 1.24+ or OpenShift 4.10+
- Helm 3.10+
- An Anthropic API key — get one at [console.anthropic.com](https://console.anthropic.com)
- A Slack workspace for alerts

---

## Configuration

### Full values.yaml Reference

```yaml
kubepilot:
  clusterName: "my-cluster"      # Name shown in alerts and reports
  clusterType: "auto"            # auto | kubernetes | openshift

slack:
  channelId: ""                  # Channel ID for alerts (starts with C)
  reportChannelId: ""            # Optional separate channel for reports

learning:
  mode: "active"                 # active | paused | disabled
  retentionDays: 90

safety:
  maxMemoryMultiplier: 2         # Max 2x memory increase allowed
  maxReplicaMultiplier: 2        # Max 2x replica increase allowed
  protectedNamespaces:
    - kube-system
    - openshift-system

reports:
  enabled: true
  dayOfWeek: "monday"
  timeHour: 8
  timezone: "America/Toronto"
  recipients: []

image:
  repository: kubepilot/kubepilot-ai
  tag: "0.1.0"
  pullPolicy: IfNotPresent

resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

---

## Getting Your API Keys

### Anthropic API Key
1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Click **API Keys** in the left menu
3. Click **Create Key**
4. Copy the key — it starts with `sk-ant-`

### Slack Bot Token
1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Click **Create New App → From scratch**
3. Name it `KubePilot AI`, select your workspace
4. Go to **OAuth & Permissions**
5. Add Bot Token Scopes: `chat:write`, `chat:write.public`
6. Click **Install to Workspace**
7. Copy the **Bot User OAuth Token** — starts with `xoxb-`

### Slack Channel ID
1. Open Slack
2. Right click on your alerts channel
3. Click **View channel details**
4. Scroll to the bottom — copy the Channel ID starting with `C`

---

## What KubePilot Detects And Fixes

| Incident | How Detected | Action Taken |
|---|---|---|
| CrashLoopBackOff | Restart count ≥ 3 | Restart pod |
| OOMKill | Exit code 137 | Increase memory limit |
| Pending pod | Pod stuck in Pending phase | Restart pod |
| Node cordoned | Node marked unschedulable | Uncordon node |
| Node NotReady | Node condition False | Alert and escalate |

---

## Safety Guarantees

KubePilot has hardcoded safety rules that **cannot be overridden**:

- ❌ Never deletes namespaces
- ❌ Never scales deployments to 0 replicas
- ❌ Never touches `kube-system` or `openshift-system`
- ❌ Never increases memory more than 2x current value
- ❌ Never increases replicas more than 2x current count
- ✅ Always dry-runs before executing
- ✅ Always waits for human approval
- ✅ Every action recorded in journal with rollback

---

## AI Model Routing

KubePilot automatically routes each incident to the right model:

| Severity | Model | Cost Per Incident |
|---|---|---|
| Low | Claude Haiku 4.5 | ~$0.002 |
| Medium | Claude Sonnet 4.6 | ~$0.006 |
| Critical | Claude Opus 4.7 | ~$0.015 |

Average cost is **$0.003 per incident** — analyzing 1,000 incidents costs about $3.

---

## Upgrading

```bash
helm repo update
helm upgrade kubepilot kubepilot/kubepilot --namespace kubepilot
```

---

## Uninstalling

```bash
helm uninstall kubepilot --namespace kubepilot
kubectl delete namespace kubepilot
```

---

## Troubleshooting

### Pod not starting
```bash
kubectl logs -n kubepilot deployment/kubepilot
kubectl describe pod -n kubepilot
```

### Secret not found error
Make sure you created the secret before installing:
```bash
kubectl get secret kubepilot-secrets -n kubepilot
```

### Slack alerts not arriving
- Make sure your bot is invited to the channel
- In Slack type `/invite @KubePilot AI` in the channel
- Verify your `SLACK_CHANNEL_ID` starts with `C` not `#`

### Anthropic authentication error
- Verify your API key at console.anthropic.com
- Make sure billing is set up on your Anthropic account
- Check the secret was created correctly:
```bash
kubectl get secret kubepilot-secrets -n kubepilot -o yaml
```

---

## Chart Versions

| Chart Version | App Version | Kubernetes | Notes |
|---|---|---|---|
| 0.1.0 | 0.1.0 | 1.24+ | Initial release |

---

## Security

- KubePilot runs with **minimal RBAC permissions** — only what it needs
- All secrets stored as **Kubernetes Secrets** — never in values.yaml
- Source code is **obfuscated** — cannot be reverse engineered
- No inbound connections required — only outbound HTTPS to Claude API and Slack

---

## Support

- 📧 Email: mustapha.amsloukh@gmail.com
- 🌐 Website: kubepilot.ai
- 🐛 Issues: Contact via email for beta support

---

## About

KubePilot AI is built by **Mustapha Amsloukh**, a Senior DevOps Engineer with 6+ years of production Kubernetes experience. Built because getting paged at 2am for cryptic Kubernetes errors gets old fast.

---

*KubePilot AI — Stop getting paged at 2am.*# kubepilot-h
