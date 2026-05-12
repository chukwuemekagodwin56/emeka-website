# Emeka Website — Azure Deployment Guide

This folder contains everything needed to deploy the **Emeka** single-page website
to **Azure App Service** using an ARM template.

---

## 📁 Files

| File | Purpose |
|---|---|
| `index.html` | The Emeka website |
| `azuredeploy.json` | ARM template — creates App Service Plan + Web App |
| `azuredeploy.parameters.json` | Parameter values (edit before deploying) |
| `README.md` | This guide |

---

## ✅ Prerequisites

- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) installed
- An active Azure subscription
- VS Code with the **Azure App Service** extension (optional but recommended)

---

## 🚀 Deployment Steps

### Step 1 — Login to Azure

```bash
az login
```

### Step 2 — Create a Resource Group

```bash
az group create \
  --name emeka-rg \
  --location westeurope
```

### Step 3 — Deploy the ARM Template

```bash
az deployment group create \
  --resource-group emeka-rg \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json
```

The deployment takes ~1–2 minutes. At the end, the output will show your **Web App URL**
(e.g. `https://emeka-website-abc123.azurewebsites.net`).

### Step 4 — Deploy the Website Files

**Option A — Azure CLI (zip deploy):**

```bash
# Zip the site files
zip site.zip index.html

# Deploy the zip
az webapp deploy \
  --resource-group emeka-rg \
  --name <YOUR_WEB_APP_NAME> \
  --src-path site.zip \
  --type zip
```

> Replace `<YOUR_WEB_APP_NAME>` with the `webAppName` output from Step 3.

**Option B — VS Code Azure Extension:**
1. Open this folder in VS Code
2. Install the **Azure App Service** extension
3. Sign in → right-click your App Service → **Deploy to Web App**
4. Select this folder → confirm

---

## ⚙️ Parameters Reference

| Parameter | Default | Options | Notes |
|---|---|---|---|
| `webAppName` | `emeka-website` | Any unique string | Becomes part of the URL |
| `sku` | `F1` | `F1`, `B1`, `B2`, `S1`, `P1v3` | F1 is free; B1+ needed for custom domains |
| `location` | Resource group region | Any Azure region | e.g. `westeurope`, `eastus` |
| `linuxFxVersion` | `NODE\|20-lts` | Any supported runtime | Keep as-is for static HTML |

---

## 🔒 Security Features Included

- HTTPS-only enforced
- TLS 1.2 minimum
- FTPS disabled
- System-assigned Managed Identity
- HTTP/2 enabled

---

## 💡 Upgrade to a Custom Domain (Optional)

After deploying with B1 or higher SKU:

```bash
# Map your custom domain
az webapp config hostname add \
  --resource-group emeka-rg \
  --webapp-name <YOUR_WEB_APP_NAME> \
  --hostname www.yourdomain.com

# Create a free managed SSL certificate
az webapp config ssl create \
  --resource-group emeka-rg \
  --name <YOUR_WEB_APP_NAME> \
  --hostname www.yourdomain.com
```

---

## 🗑️ Clean Up

To delete all resources when you're done:

```bash
az group delete --name emeka-rg --yes --no-wait
```
