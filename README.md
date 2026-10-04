# Cloud Web App — AWS Infrastructure as Code (Terraform)

A production-style, 3-tier web application architecture on AWS, fully provisioned and managed with Terraform. Built as a learning project to demonstrate real-world cloud infrastructure skills: networking, compute auto-scaling, load balancing, a managed database, and a static frontend — all wired together and version-controlled as code.

## Architecture

Visitor Browser
  -> S3 (static website, index.html frontend)
  -> JS fetch() call -> Application Load Balancer (public)
       -> Auto Scaling Group (EC2 + nginx, self-heals)
            -> RDS PostgreSQL (private)

All EC2/RDS/ALB components live inside a custom VPC with public + private
subnets across 2 Availability Zones.

## What this project demonstrates

- **Infrastructure as Code** — every resource is defined in Terraform (`main.tf`), not clicked together manually. The whole environment can be destroyed and rebuilt from the code alone.
- **Secure network design** — a custom VPC with public and private subnets across 2 AZs. The database sits in private subnets with no route to the internet at all.
- **Layered security groups** — each tier only trusts the tier directly in front of it (ALB <- internet, EC2 <- ALB only, RDS <- EC2 only), rather than broad open access.
- **Self-healing compute** — EC2 instances run inside an Auto Scaling Group via a Launch Template. If an instance fails, it's automatically replaced.
- **Load balancing & health checks** — an Application Load Balancer distributes traffic and only routes to instances that pass real HTTP health checks.
- **Managed database** — PostgreSQL via RDS, isolated in private subnets, provisioned entirely through code.
- **Decoupled frontend** — a static frontend hosted separately from the backend, calling the backend's public API over the internet.
- **Secrets hygiene** — sensitive values (IP, DB password) are kept out of version control via Terraform variables and a gitignored `terraform.tfvars` file.

## Tech stack

- Terraform — infrastructure provisioning
- AWS VPC — custom networking, 2 AZs, public/private subnets, Internet Gateway, route tables
- AWS EC2 + Auto Scaling Group + Launch Template — backend compute
- AWS Application Load Balancer — traffic distribution & health checks
- AWS RDS (PostgreSQL) — managed database
- AWS S3 — static frontend hosting
- nginx — installed automatically via instance `user_data` on boot

## Project structure

├── main.tf # All infrastructure resources
├── index.html # Static frontend (calls the backend ALB)
├── terraform.tfvars.example # Template for required variables (no real secrets)
├── .gitignore
└── .terraform.lock.hcl


## Setup

1. Clone the repo and install Terraform and the AWS CLI.
2. Configure AWS credentials (`aws configure`).
3. Copy `terraform.tfvars.example` to `terraform.tfvars` and fill in your own values.
4. Run:
```bash
   terraform init
   terraform plan
   terraform apply
```
5. Grab the ALB's DNS name (EC2 -> Load Balancers) and the S3 website endpoint (S3 -> bucket -> Properties -> Static website hosting) to see it live.

## Notes on current state

- The frontend currently runs on S3 static website hosting directly. A CloudFront distribution (with Origin Access Control, so the S3 bucket stays fully private) is built into the configuration but pending AWS account verification for CloudFront.
- CORS is currently open (`Access-Control-Allow-Origin: *`) for development simplicity; a production version would restrict this to the specific frontend origin.

## What I'd do differently in production

- Restrict CORS to the exact frontend domain rather than `*`
- Store the DB password in AWS Secrets Manager instead of a `.tfvars` file
- Use Terraform remote state (e.g. an S3 backend) instead of local state
- Add a CI/CD pipeline to run `terraform plan` on every pull request

---

Built as part of a hands-on cloud engineering learning path — every resource was written, debugged, and understood line by line.
