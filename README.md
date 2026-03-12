# 🚀 Production-Ready WordPress on Kubernetes (Zero Trust)

Managed and Automated by **[Mohamed Adel](https://github.com/mohamedadel-devops)**

A highly secure, automated, and scalable WordPress deployment leveraging **Kubernetes (K3s)**, **Cloudflare Tunnels**, and **GitHub Actions**. This project demonstrates a "Zero Trust" architecture where the application is accessible to the world without ever opening a single inbound port on the host firewall.

## 🏗️ Architecture Overview

The infrastructure is designed with security and automation at its core:
* **Orchestration:** K3s (Lightweight Kubernetes) running on AWS EC2.
* **Zero Trust Access:** Cloudflare Tunnels (`cloudflared`) create an encrypted outbound-only connection, eliminating the need for public-facing LoadBalancers or open SSH/HTTP ports.
* **Ingress Management:** NGINX Ingress Controller handles internal routing and SSL termination.
* **CI/CD Pipeline:** GitHub Actions automates the entire lifecycle, including manifest validation (**Kubeconform**) and dynamic firewall whitelisting for deployments.
* **Stateful Storage:** Persistent Volume Claims (PVC) backed by local storage for MySQL and WordPress uploads.
* **Automated Backups:** Daily CronJobs to dump the database and sync to **AWS S3** using the AWS CLI.

## 🛡️ Best Practices Implemented

* **Infrastructure as Code (IaC):** Every resource is defined in declarative YAML manifests.
* **GitOps Workflow:** Pushing to the `main` branch triggers an automated deployment.
* **Secret Management:** Sensitive data (passwords, AWS keys) are managed via GitHub Secrets and injected at runtime using `envsubst` to avoid hardcoding.
* **Dynamic Security Groups:** The CI/CD pipeline dynamically authorizes the GitHub Runner's IP for the K3s API port (6443) only during the deployment window, revoking it immediately after.
* **Least Privilege:** Dedicated IAM users with scoped permissions for S3 storage and EC2 security group management.
* **Health & Availability:** Implemented `rollout status` checks in the pipeline to ensure database readiness before application deployment.

## 📂 Project Structure

```text
├── .github/workflows/
│   └── deploy.yml          # The CI/CD engine (Node.js 24 optimized)
├── k8s/
│   ├── namespace.yaml      # Multi-namespace isolation (app vs infra)
│   ├── secrets.yaml        # Template for env-injected secrets
│   ├── cloudflared.yaml    # Cloudflare Tunnel deployment
│   ├── nginx-service.yaml  # Internal Ingress controller (ClusterIP)
│   ├── mysql-deployment.yaml
│   ├── wordpress-deployment.yaml
│   ├── ingress.yaml        # L7 routing rules for brandrv.com
│   └── backup/
│       └── backup-cronjob.yml # Automated S3 offsite backups

================================================================================
## 🚀 DEPLOYMENT PROCESS & ARCHITECTURE
================================================================================

This project utilizes a fully automated GitOps CI/CD pipeline. 
Follow the steps below to replicate the production environment.

1. INFRASTRUCTURE PREPARATION
--------------------------------------------------------------------------------
* COMPUTE: Provision an Ubuntu EC2 (t3.medium recommended) in eu-north-1.
* K8S:     Install K3s and verify cluster health (kubectl get nodes).
* IAM:     Create a user with S3FullAccess and EC2 Security Group permissions.

2. GITHUB SECRETS CONFIGURATION
--------------------------------------------------------------------------------
Add the following keys under Settings > Secrets and variables > Actions:

[ KUBECONFIG ]
-> Content of ~/.kube/config (Set 'server' to Public EC2 IP)

[ AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY ]
-> Credentials for the IAM user managing S3 and Security Groups.

[ AWS_SG_ID ]
-> The ID of the Security Group (sg-xxxx) attached to your EC2.

[ CLOUDFLARE_TUNNEL_TOKEN ]
-> Authentication token from the Cloudflare Zero Trust Dashboard.

[ DB CREDENTIALS ]
-> MYSQL_ROOT_PASSWORD, MYSQL_PASSWORD, MYSQL_USER, MYSQL_DATABASE.

3. AUTOMATED WORKFLOW STAGES
--------------------------------------------------------------------------------
The pipeline (deploy.yml) executes these steps on every push to 'main':

STEP A: LINTING
   Validates YAML syntax using Kubeconform against OpenAPI schemas.

STEP B: DYNAMIC WHITELISTING
   The Runner detects its IP and authorizes port 6443 in AWS temporarily.

STEP C: SECRET INJECTION
   Uses 'envsubst' to safely populate k8s/secrets.yaml templates.

STEP D: ORCHESTRATION
   Applies manifests in order: Namespace -> Secrets -> Storage -> Apps.

STEP E: CLEANUP
   Automatically revokes Runner IP access from AWS, regardless of success.

================================================================================
Managed by: Mohamed Adel (https://github.com/mohamedadel-devops)
================================================================================
