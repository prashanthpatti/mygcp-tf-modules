# 📦 Terraform Modules for GCP Infrastructure

This directory contains reusable Terraform modules to manage essential GCP services:

- ✅ VPC
- ✅ Cloud Function (Gen2)
- ✅ IAM Bindings
- ✅ Service Account
- ✅ Load Balancer

Each module is designed to be simple, composable, and production-ready.

---

## 📁 Modules Overview

### 1️⃣ VPC Module (`"git::https://github.com/prashanthpatti/mygcp-tf-modules.git//vpc/`)

#### 🔧 Resources Created:
- VPC Network
- Subnets (Regional)
- Optional Firewall Rules (customizable)

#### 📥 Inputs:

| Variable         | Type     | Description                     | Example              |
|------------------|----------|----------------------------------|----------------------|
| `vpc_name`       | string   | Name of the VPC                  | `"my-vpc"`           |
| `subnet_name`    | string   | Name of the subnet               | `"my-subnet"`        |
| `ip_cidr_range`  | string   | IP CIDR block for the subnet     | `"10.0.0.0/24"`      |
| `region`         | string   | GCP region                       | `"us-central1"`      |

#### 📤 Outputs:

| Name           | Description           |
|----------------|-----------------------|
| `vpc_id`       | ID of the VPC network |
| `subnet_id`    | ID of the subnet      |

---

### 2️⃣ Cloud Function Module (`"git::https://github.com/prashanthpatti/mygcp-tf-modules.git//cloud_function/`)

#### 🔧 Resources Created:
- Cloud Function (Gen2)
- Associated IAM roles
- Artifact storage (bucket)

#### 📥 Inputs:

| Variable           | Type   | Description                             | Example                            |
|--------------------|--------|-----------------------------------------|------------------------------------|
| `function_name`    | string | Name of the cloud function              | `"hello-function"`                 |
| `region`           | string | Region for deployment                   | `"us-central1"`                    |
| `entry_point`      | string | Name of the Python entry method         | `"hello_world"`                    |
| `runtime`          | string | Runtime environment                     | `"python311"`                      |
| `source_archive`   | string | Path to ZIP file with source code       | `"function-source.zip"`            |
| `is_public`        | bool   | Whether to make function public         | `true`                             |
| `public_member`    | string | IAM member to assign `invoker` role     | `"allUsers"` or custom SA          |

#### 📤 Outputs:

| Name             | Description                      |
|------------------|----------------------------------|
| `function_url`   | URL endpoint of the function     |
| `function_name`  | Name of the function             |

---

### 3️⃣ IAM Module (`"git::https://github.com/prashanthpatti/mygcp-tf-modules.git//iam/`)

#### 🔧 Resources Created:
- IAM policy bindings for users or service accounts

#### 📥 Inputs:

| Variable         | Type   | Description                         | Example                                      |
|------------------|--------|-------------------------------------|----------------------------------------------|
| `project_id`     | string | GCP project ID                      | `"my-project-123"`                            |
| `bindings`       | map    | Map of role → list of members       | `{ "roles/viewer" = ["user:john@example.com"] }` |

#### 📤 Outputs:
*None (IAM is a direct binding module)*

---

### 4️⃣ Service Account Module (`"git::https://github.com/prashanthpatti/mygcp-tf-modules.git//service_account/`)

#### 🔧 Resources Created:
- GCP Service Account
- (Optional) IAM roles

#### 📥 Inputs:

| Variable             | Type     | Description                           | Example                             |
|----------------------|----------|---------------------------------------|-------------------------------------|
| `account_id`         | string   | Unique service account ID             | `"my-app-sa"`                       |
| `display_name`       | string   | Human-readable name                   | `"App Service Account"`             |
| `project_id`         | string   | GCP project ID                        | `"my-project"`                      |
| `iam_roles`          | list     | List of IAM roles to assign           | `[ "roles/logging.logWriter" ]`     |

#### 📤 Outputs:

| Name                | Description                         |
|---------------------|-------------------------------------|
| `service_account_email` | Email address of the SA         |

---

### 5️⃣ Load Balancer Module (`"git::https://github.com/prashanthpatti/mygcp-tf-modules.git//load_balancer/`)

#### 🔧 Resources Created:
- Serverless Network Endpoint Group (NEG)
- HTTPS Load Balancer (Global)
- Backend Services, URL Map, Target Proxy, Forwarding Rule
- SSL certificate (self-managed or Google-managed)

#### 📥 Inputs:

| Variable                   | Type   | Description                               | Example                       |
|----------------------------|--------|-------------------------------------------|-------------------------------|
| `project_id`               | string | GCP project ID                            | `"my-project"`                |
| `region`                   | string | Region                                    | `"us-central1"`               |
| `function_name`            | string | Name of cloud function to expose          | `"hello-function"`            |
| `is_public`                | bool   | Whether load balancer is public           | `true`                        |
| `public_member`            | string | Member (e.g. `"allUsers"`, service account)| `"allUsers"`                  |

#### 📤 Outputs:

| Name                | Description                          |
|---------------------|--------------------------------------|
| `load_balancer_ip`  | Public IP of the HTTP(S) Load Balancer |

---

## 🧠 Usage Example

In your `main.tf`:

```hcl
module "cloud_function" {
  source          = "git::https://github.com/prashanthpatti/mygcp-tf-modules.git//cloud_function?ref=v1.0"
  function_name   = "hello-fn"
  region          = "us-central1"
  entry_point     = "hello_world"
  runtime         = "python311"
  source_archive  = "function-source.zip"
  is_public       = true
  public_member   = "allUsers"
}

module "vpc" {
  source         = "git::https://github.com/prashanthpatti/mygcp-tf-modules.git//vpc?ref=v1.0"
  vpc_name       = "demo-vpc"
  subnet_name    = "demo-subnet"
  ip_cidr_range  = "10.10.0.0/24"
  region         = "us-central1"
}
