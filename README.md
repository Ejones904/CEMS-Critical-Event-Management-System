# CEMS — Critical Event Management System

CEMS is an AI-assisted, event-driven Critical Event Management System designed to correlate enterprise events into managed incidents, coordinate stakeholder communication, and maintain a unified operational timeline.

The project was designed around a problem I encountered while supporting business-critical transportation technology: during critical events, information can become fragmented across systems, teams, and communication channels. CEMS establishes a coordinated path for managing that information throughout the incident lifecycle.

## What CEMS Does

CEMS receives events from enterprise systems and:

- Validates and normalizes incoming event data
- Correlates related events into managed incidents
- Maintains incident severity and lifecycle state
- Stores operational data in PostgreSQL
- Processes incident activity asynchronously through Amazon SQS
- Isolates failed processing through a Dead-Letter Queue (DLQ)
- Determines stakeholder escalation using deterministic business rules
- Publishes stakeholder notifications through Amazon SNS
- Accepts and correlates inbound stakeholder responses
- Maintains a unified incident timeline
- Uses a bounded Amazon Bedrock AI operations agent for incident investigation and system health monitoring

## Architecture

CEMS follows a deliberately simplified architecture:

**One API → One Database → One Queue → One Worker → One Notification Service → One Bounded AI Agent**

Core technologies include:

- Python
- FastAPI
- PostgreSQL
- Docker
- Amazon SQS
- Amazon SNS
- Amazon ECR
- Amazon Bedrock
- AWS IAM
- Amazon CloudWatch
- Terraform

## Event Processing

Incoming events are validated and normalized before being persisted.

CEMS distinguishes between a **source event** and an **incident**. Multiple related source events can belong to the same managed incident.

Incident correlation uses deterministic criteria including event type, location, incident state, and occurrence time. Higher-severity events can promote an existing incident's severity, while resolved or closed incidents are excluded from new correlation.

This keeps critical incident decisions deterministic rather than relying on AI to determine whether events belong together.

## Asynchronous Processing and Resilience

Amazon SQS decouples event ingestion from downstream incident processing.

The worker consumes typed messages and performs incident processing and stakeholder escalation. Failed messages remain available for retry and are eventually isolated in a DLQ after exceeding the configured retry threshold.

The DLQ provides failure isolation without allowing poison messages to continuously disrupt normal processing.

## Stakeholder Communication

Escalation rules determine which stakeholders should receive communications based on event type and severity.

Amazon SNS provides outbound notification capability.

Inbound stakeholder responses are correlated back to the appropriate incident and stored as part of the operational record.

## Unified Incident Timeline

CEMS exposes an incident timeline that combines operational events and stakeholder communications into chronological order.

This creates a single operational view of what occurred during an incident and provides context for both human operators and AI-assisted investigation.

## Bounded AI Operations Agent

CEMS integrates Amazon Bedrock using Amazon Nova Lite.

The AI operations agent is intentionally bounded rather than being given unrestricted system access.

Its available tools allow it to:

- Retrieve incident context
- Add investigation notes
- Check system health
- Request a controlled worker restart action

The agent cannot arbitrarily execute shell commands, modify IAM, change incident severity, close incidents, alter infrastructure, perform destructive database operations, or send unrestricted communications.

This demonstrates an AI design where reasoning capability is separated from operational authority.

## AWS and Infrastructure as Code

The application was containerized with Docker and successfully published to Amazon ECR.

Terraform defines the target AWS architecture, including:

- VPC
- Private application and database subnets
- Security groups
- Internal Application Load Balancer
- EC2 application hosts
- Amazon RDS PostgreSQL
- SQS and DLQ
- SNS
- IAM
- CloudWatch

The Terraform configuration was formatted and validated successfully and produced:

**Plan: 33 to add, 0 to change, 0 to destroy**

The complete EC2/RDS/ALB environment was intentionally not provisioned. The Terraform configuration represents the target cloud architecture while avoiding unnecessary ongoing infrastructure cost for a portfolio project.

## Security Approach

CEMS was designed around several security principles:

- Least-privilege IAM
- Private application and database networking
- No public database access
- Restricted security-group communication
- IMDSv2 for EC2
- Encrypted database and storage resources
- AI tool allowlisting
- Bounded remediation authority
- Auditable agent activity
- Secrets excluded from source control

## Engineering Decisions

Several features were intentionally kept deterministic rather than AI-driven.

AI assists with investigation and operational context, while incident correlation, severity handling, escalation rules, persistence, and message processing remain controlled by application logic.

The architecture also deliberately avoids unnecessary complexity such as Kubernetes, Kafka, multi-region deployment, multiple agents, and additional microservices.

The goal was not to use as many technologies as possible. The goal was to design a system whose complexity could be justified.

## Implementation Status

### Implemented and Tested

- FastAPI event ingestion
- PostgreSQL persistence
- Incident correlation
- Severity promotion
- Amazon SQS processing
- Dead-Letter Queue configuration
- Worker processing
- Amazon SNS publishing
- Inbound stakeholder responses
- Unified incident timeline
- Amazon Bedrock AI operations agent
- AI tool guardrails
- Docker containerization
- Amazon ECR image publishing
- Terraform validation and infrastructure planning

### Target Architecture / Future Production Work

A production deployment would additionally require areas such as:

- Complete private AWS service connectivity
- Production secret management
- TLS termination
- Authentication and authorization
- Automated application bootstrap/deployment
- Expanded monitoring and alerting
- Production-scale high availability

## AI-Assisted Development

AI was used throughout the project as an engineering accelerator to translate architecture decisions into implementation, challenge design assumptions, troubleshoot issues, and accelerate development.

Architecture, requirements, system behavior, security boundaries, testing, troubleshooting, and engineering tradeoffs remained deliberate parts of the development process.

## Repository Scope

This repository contains the **public technical documentation and evidence for CEMS**.

The full application source code, database migrations, worker implementation, Terraform source, and internal configuration are maintained separately in a private repository.

## Project Evidence

Architecture diagrams, API demonstrations, AWS integration evidence, Terraform validation, and additional technical documentation are included in this repository.
