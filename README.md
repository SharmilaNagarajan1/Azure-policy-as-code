# 🚀 Azure Policy as Code — Deny Non-EastUS Resources

This repository demonstrates how to **define, store, and deploy Azure Policy as code** using **GitHub Actions**.  
The policy enforces that resources can be created **only in the East US region**.

---

## 📂 Repository Structure

```bash
├── .github/
│   └── workflows/
│       └── deploy-policy.yml        # GitHub Actions workflow to deploy policy
├── policies/
│   ├── policy-definition.json       # Policy rule definition
│   └── policy-assignment.json       # Policy assignment parameters (optional)
└── README.md
```

---

## 🔐 Prerequisites

1. **Azure Subscription**
2. **GitHub Repository** with this code
3. **Service Principal** with `Contributor` role for authentication

---

## 🧩 Create Service Principal

Run the following command in Azure Cloud Shell or your local terminal:

```bash
az ad sp create-for-rbac --name "github-policy-sp" \
  --role Contributor \
  --scopes /subscriptions/<SUBSCRIPTION_ID> \
  --sdk-auth
```

Then, copy the JSON output and save it as a **GitHub repository secret**:

`Settings → Secrets and variables → Actions → New repository secret → AZURE_CREDENTIALS`

---

## 🌿 Policy Definition Example

**policies/policy-definition.json**

```json
{
  "if": {
    "not": {
      "field": "location",
      "equals": "eastus"
    }
  },
  "then": {
    "effect": "deny"
  }
}
```

This rule denies creation of any Azure resource whose `location` is not `eastus`.

---

## ⚙️ GitHub Actions Workflow

**.github/workflows/deploy-policy.yml**

```yaml
name: Deploy Azure Policy

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy-policy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Login to Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Deploy Policy Definition
        run: |
          az policy definition create \
            --name deny-non-eastus \
            --display-name "Deny resources not in East US" \
            --description "Deny creation of resources outside East US" \
            --rules policies/policy-definition.json \
            --mode All

      - name: Assign Policy
        run: |
          az policy assignment create \
            --name deny-non-eastus-assignment \
            --policy deny-non-eastus \
            --scope /subscriptions/${{ fromJson(secrets.AZURE_CREDENTIALS).subscriptionId }}
```

---

## 🧠 How It Works

- The **policy definition** checks each resource’s location.
- If the location is **not East US**, the request is **denied**.
- The **GitHub Action** automatically deploys and assigns this policy whenever:
  - You push to the `main` branch, or  
  - You trigger the workflow manually via **workflow_dispatch**.

---

## ✅ Testing the Policy

After the workflow runs successfully:

1. Try creating a resource in a region other than `eastus` — it should **fail**.  
2. Try again in `eastus` — it should **succeed**.

---

## 🧹 Cleanup

To remove the policy and its assignment:

```bash
az policy assignment delete --name deny-non-eastus-assignment
az policy definition delete --name deny-non-eastus
```

---

## 💡 Summary

This demo shows:
- How to implement **Policy as Code**
- How to use **GitHub Actions** for Azure policy deployment
- How to enforce region compliance automatically via CI/CD

---


