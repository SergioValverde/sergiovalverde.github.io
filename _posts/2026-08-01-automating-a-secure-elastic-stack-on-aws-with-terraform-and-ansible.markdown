---
title: Automating a Secure Elastic Stack on AWS with Terraform and Ansible
date: 2026-08-01 18:16:00 +02:00
tags:
- terraform
- ansible
- aws
- elastic
license: false
show_edit_on_github: false
show_subscribe: false
pageview: true
mermaid: true
---

# From Infrastructure to Configuration: Automating a Secure Elastic Stack on AWS with Terraform and Ansible

Deploying an EC2 instance is an infrastructure problem. Installing Docker, generating certificates, configuring services, and validating an application are configuration-management problems.

In this project, I explored how **Terraform and Ansible can be combined while maintaining a clear separation between those responsibilities**.

The result is an automated workflow that provisions AWS infrastructure, configures an EC2 host, and deploys a secured Elasticsearch and Kibana environment. The project focuses on reproducibility, idempotency, security controls, and complete infrastructure lifecycle management.


## Architecture Overview

The environment is deployed in a custom AWS VPC and uses a single EC2 instance to run Elasticsearch and Kibana through Docker Compose.

<style> .mermaid-wrapper { width: 100%; display: flex; justify-content: center; margin: 24px 0; } .mermaid-wrapper svg { max-width: 100%; height: auto; } </style>

<div class="mermaid-wrapper"> <pre class="mermaid"> graph TD Operator["Operator workstation"] --> Terraform["Terraform provisions AWS infrastructure"] --> EC2["EC2 instance + Elastic IP"] --> Ansible["Ansible configures the host"] --> Docker["Docker Compose"] Docker --> ES["Elasticsearch"] Docker --> KB["Kibana"] </pre> </div>

<script src="https://cdn.jsdelivr.net/npm/mermaid@10.9.1/dist/mermaid.min.js"></script>

```text
Operator workstation
        ↓
Terraform provisions AWS infrastructure
        ↓
EC2 instance and Elastic IP
        ↓
Ansible configures the host
        ↓
Docker Compose
   ┌────┴─────┐
   ↓          ↓
Elasticsearch Kibana
```

The EC2 instance is located in a public subnet and receives an Elastic IP. Access to SSH, Elasticsearch, and Kibana is restricted to the public IPv4 address of the deployment workstation.

The deployment also enables TLS, Elastic authentication, encrypted storage, and IMDSv2.

This is a single-node laboratory architecture designed to demonstrate automation and security decisions. It is not intended to represent a highly available production Elastic platform.

## Why Terraform and Ansible?

Terraform and Ansible solve related but different problems.

Terraform is responsible for declaring and managing the AWS infrastructure:

```text
Terraform
├── VPC and subnet
├── Internet routing
├── Security Group
├── EC2 instance
├── Elastic IP
├── Encrypted storage
└── Infrastructure outputs
```

Ansible is responsible for transforming the provisioned EC2 instance into the required application platform:

```text
Ansible
├── Host preflight validation
├── Docker installation
├── Directory and file management
├── Configuration templates
├── TLS certificate generation
├── Elasticsearch deployment
├── Kibana deployment
└── Internal service validation
```

This separation keeps the workflow easier to understand and troubleshoot.

If the VPC, subnet, or EC2 instance cannot be created, the issue belongs to the Terraform layer. If the infrastructure exists but Docker or Elasticsearch cannot be configured, the issue belongs to the Ansible or application layer.

## Terraform: Provisioning the AWS Infrastructure

Terraform defines the AWS foundation required by the deployment.

Rather than creating resources manually through the AWS console, the infrastructure is described as code and managed through the standard Terraform lifecycle:

```bash
terraform init
terraform validate
terraform plan
terraform apply
```

The configuration provisions the network, compute instance, Elastic IP, encrypted EBS storage, and restricted Security Group rules.

A saved Terraform plan is created and reviewed before the infrastructure is applied. I retained this approval step intentionally because infrastructure changes may affect security, connectivity, and cost.

Terraform also produces the outputs required by the next automation stage. In particular, the EC2 Elastic IP becomes the connection point used by the orchestration and Ansible workflow.

This creates a clear boundary:

```text
Terraform creates the infrastructure
        ↓
Terraform exposes the required outputs
        ↓
The configuration stage consumes those outputs
```

Terraform remains responsible for the infrastructure lifecycle. It can create the environment, detect changes, and later remove the resources through a controlled destroy plan.

## Ansible: Configuring the EC2 Host

Once Terraform has created the infrastructure, Ansible configures the EC2 instance.

Before modifying the host, a preflight playbook validates that the instance meets the expected conditions. This includes checking the operating system, available memory and disk space, Python availability, SSH connectivity, and privilege escalation.

The main configuration then installs and manages Docker, prepares the Elastic Stack directories, renders the required configuration templates, generates the private certificate authority and service certificates, and starts the application services.

The deployment order is controlled:

```text
Validate the host
        ↓
Install and start Docker
        ↓
Generate TLS material
        ↓
Deploy Elasticsearch
        ↓
Configure the Kibana service account
        ↓
Deploy Kibana
        ↓
Validate both services over HTTPS
```

Ansible is not being used as a sequence of remote shell commands. The playbooks declare the required state of the host and converge the instance toward that state.

## Connecting Terraform and Ansible

The most relevant part of the project is the handoff between infrastructure provisioning and configuration management.

```text
Terraform provisions EC2
        ↓
Terraform returns the Elastic IP
        ↓
The deployment workflow waits for SSH
        ↓
The SSH host fingerprint is reviewed
        ↓
Ansible receives the instance as inventory
        ↓
Ansible configures the host
        ↓
The services are validated
```

The Bash scripts coordinate this workflow, but they do not replace Terraform or Ansible.

Terraform still performs infrastructure planning and state management. Ansible still performs host configuration. The scripts provide a consistent execution path between both tools.

The workflow also retains explicit approval for Terraform application, SSH host-key trust, and infrastructure destruction. These steps represent security and cost-control boundaries rather than missing automation.

## Idempotency

A major validation objective was Ansible idempotency.

During the first execution, Ansible installs packages, creates directories, generates configuration, and starts services. Changes are expected.

During the second execution, the host should already match the declared state.

```text
First execution:
changed > 0

Second execution:
changed = 0
```

The second configuration run completed with `changed=0`.

This demonstrates that the playbooks converge toward a stable state instead of repeating unnecessary actions every time they are executed.

Idempotency is important because configuration automation must support safe reapplication. It improves repeatability, recovery, troubleshooting, and confidence when applying future changes.

This result applies to the validated environment and current project versions. It does not prove compatibility with every operating system or future Elastic release.

## Security Built into Both Layers

Security controls are distributed between Terraform and Ansible.

Terraform manages infrastructure-level controls such as:

* Security Group access restricted to the operator’s IPv4 address
* Encrypted EBS storage
* Mandatory IMDSv2
* Controlled SSH key association
* Explicit infrastructure planning and destruction

Ansible manages host and application controls such as:

* TLS certificate generation
* HTTPS for Elasticsearch and Kibana
* Elastic authentication
* Certificate verification
* Dedicated service credentials
* Restricted local secret-file permissions

The deployment therefore does not rely on a single boundary.

A client must originate from an approved source, connect through the permitted network path, establish a valid TLS session, and authenticate to the Elastic service.

## Validation and Infrastructure Lifecycle

The project was tested across the complete workflow:

```text
Provision
    ↓
Configure
    ↓
Validate
    ↓
Reapply idempotently
    ↓
Destroy
    ↓
Recreate
```

Validation included:

* Clean Terraform provisioning
* Clean EC2 configuration through Ansible
* Docker installation and service management
* TLS certificate generation
* Secure Elasticsearch and Kibana deployment
* Internal HTTPS checks
* External HTTPS checks
* A second Ansible execution with `changed=0`
* Terraform destruction and recreation

Testing the complete lifecycle was important. A reproducible project should not only create infrastructure successfully; it should also be capable of removing and rebuilding it predictably.

## Project Limitations

This environment is designed as a laboratory and portfolio implementation.

It uses a single EC2 instance in a public subnet and does not provide high availability. Terraform state and application credentials are also managed locally.

A production architecture would require additional decisions around:

* Private networking
* Remote Terraform state and locking
* Managed secret storage
* Multi-node Elasticsearch clustering
* Multi-AZ resilience
* Backup and recovery
* Centralized monitoring
* Certificate lifecycle management
* Upgrade and rollback procedures
* CI/CD integration

Depending on the workload and operational requirements, a managed service such as Amazon OpenSearch Service or Elastic Cloud may also be more appropriate than operating Elasticsearch directly on EC2.

## Key Takeaway

The most important outcome of this project is not simply an EC2 instance running Elasticsearch and Kibana.

The project demonstrates how Terraform and Ansible can be combined while preserving clear ownership of each automation layer:

```text
Terraform manages infrastructure
Ansible manages configuration
Docker Compose manages services
Bash coordinates the workflow
```

Terraform provisions and manages the AWS resources. Ansible converts the resulting EC2 instance into the required application platform. The orchestration layer connects both stages without hiding their native workflows.

The resulting environment can be provisioned, configured, validated, reapplied idempotently, destroyed, and recreated through a documented process.

## Project Repository

The complete Terraform configuration, Ansible roles, orchestration scripts, security decisions, deployment instructions, and validation workflow are available in the GitHub repository:

[View the AWS Elastic Automation project on GitHub](https://github.com/SergioValverde/aws-elastic-automation)
