# 🚀 GCP Project Bootstrap & Terraform Automation

This repository automates the **creation and management of Google Cloud Projects** and key resources like **Cloud Functions**, **Load Balancers**, **Service Accounts**, and **IAM configurations** — all through **Bash scripts** and **Terraform**.

---

## 📁 Repository Structure

# 🚀 GCP Project Bootstrap & Terraform Automation

This repository automates the **creation and management of Google Cloud Projects** and key resources like **Cloud Functions**, **Load Balancers**, **Service Accounts**, and **IAM configurations** — all through **Bash scripts** and **Terraform**.

---

## 📁 Repository Structure

```text
.
├── execute.sh               # Bootstrap project, APIs, and infra
├── terminate.sh             # Destroy Terraform-managed resources
├── tf-project/
│   ├── backend.tf           # GCS backend for Terraform state
│   ├── config.env           # Configuration file
│   ├── main.tf              # Main Terraform logic (uses modules)
│   ├── outputs.tf           # Resource outputs
│   ├── variables.tf         # Input variables
│   ├── vars.tfvars          # Auto-filled config values (e.g., project_id)
│   └── versions.tf          # Terraform/provider versions
├── tf-modules/
│   ├── vpc/                 # VPC Module
│   ├── cloud_function/      # Cloud Function Module
│   ├── service_account/     # Service Account Module
│   ├── iam/                 # IAM Role Bindings
│   └── load_balancer/       # Load Balancer Module
├── function/                
│   ├── main.py              # Cloud Function source code
│   └── function-source.zip  # Zipped deployment artifact
```

---

## ⚙️ Prerequisites

* [Terraform](https://developer.hashicorp.com/terraform/downloads)
* [gcloud CLI](https://cloud.google.com/sdk/docs/install)
* A GCP billing account
* GCP IAM user/service account with appropriate permissions

### 🔧 Required APIs

Enable the following APIs manually or through the script:

* Cloud Resource Manager
* IAM
* Cloud Functions
* Cloud Run
* VPC Access
* Cloud Build
* Artifact Registry

---

## ☁️ Setup GCS Backend (One-time)

```bash
gsutil mb -l us-central1 gs://<your-unique-bucket-name>
```

> ⚠️ Required once per project. Terraform cannot create its own backend.

---

## 🔐 Required Permissions

Ensure the account running the scripts has:

* `resourcemanager.projects.create`
* `billing.resourceAssociations.create`
* `serviceusage.services.enable`
* IAM admin rights
* VPC access rights

---

## 🧾 Naming Conventions

### ✅ Format

```
<env>-<project_name>-<resource_type>
```

| Component       | Example                    | Description                       |
| --------------- | -------------------------- | --------------------------------- |
| `env`           | `dev`                      | Environment (dev, prod, etc.)     |
| `project_name`  | `pkumar-gcp`               | Short, lowercase, hyphenated name |
| `resource_type` | `sa`, `subnet`,`connector` | Suffix for resource type          |

### ⚠️ Special Naming Rules

* Max 25 characters for VPC Access Connectors
* Must match: `^[a-z][-a-z0-9]{0,23}[a-z0-9]$`

```hcl
connector_name = "${var.env}-${substr(var.project_name, 0, 10)}-conn"  #Extracts the first 10 characters from the project_name variable. This helps keep the total name within GCP's 25-character limit
```

---

## 📥 Clone the Repository

```bash
git clone https://github.com/prashanthpatti/mygcprepo.git
cd mygcprepo
```

---

## 🔧 Configuration Steps

### 1️⃣ Edit `backend.tf` (Once)

```hcl
backend "gcs" {
  bucket = "<your-unique-bucket-name>"
  prefix = "gcp-cloud-function/state"
}
```

### 2️⃣ Fill in `config.env`

```hcl
project_name     = "myproject"
billing_account  = "YOUR-BILLING-ID"
region           = "us-central1"
```

> 💡 Do **not** add `project_id` manually – it’s auto-populated.

---

## 🚀 Deploy Infrastructure

```bash
./execute.sh
```

### 🛠 What it Does:

* Creates or reuses GCP project
* Associates billing account
* Enables required APIs
* Updates `vars.tfvars` with `project_id`
* Applies all Terraform modules

⏳ **Note**: VPC Access Connector creation may take several minutes.

---

## 🌐 Access the Application

Check output for:

```hcl
load_balancer_ip = "34.xxx.xxx.xxx"
```

Open in your browser:

```url
https://<load_balancer_ip>
```

> ⚠️ Works only if `is_public = true`
> ℹ️ Initial delay expected due to Cloud Function cold start

---

## 🧹 Cleanup

```bash
./terminate.sh
```

* Tears down all Terraform-managed resources
* Prompts whether to delete the project (optional)

---

## ✅ Managed Resources

* GCP Project (with billing)
* Cloud Function (Gen2)
* VPC & Subnets
* VPC Access Connector
* HTTP Load Balancer
* IAM Bindings
* Source Code Storage Bucket

---

## 🔐 Security: Public vs Private Access

### Public (Default)

```hcl
is_public     = true
public_member = "allUsers"
```

Allows Load Balancer to invoke unauthenticated Cloud Function.

### Private (Recommended for Production)

```hcl
is_public     = true
public_member = "serviceAccount:internal-sa@project.iam.gserviceaccount.com"
```

Use:

* IAP (Identity-Aware Proxy)
* Authenticated clients with identity tokens

> ❌ Load Balancer → Private Function with NEG won’t work due to lack of token propagation

---

## 📌 Deployment Summary

| Use Case       | IAM Policy Setting         | Works? |
| -------------- | -------------------------- | ------ |
| Public demo    | `run.invoker` + `allUsers` | ✅ Yes  |
| Secure prod    | Identity token / IAP       | ✅ Yes  |
| Custom SA only | NEG lacks token forwarding | ❌ No   |

---

## 🧠 Notes

* Scripts are tested in **Windows Git Bash**
* Supports re-running `execute.sh` safely
* Uses suffix-based unique project IDs

---

## 🚧 TODO / Improvements

* Org folder support
* Destroy specific modules
* Logging enhancements
* Toggle between public/private setup

---

## 📞 Support

Need help customizing or extending this setup?
Open an issue or contact the maintainer.

---

```

Let me know if you’d like this output also saved as a downloadable `README.md` file.
```
