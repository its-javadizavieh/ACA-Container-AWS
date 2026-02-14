# ACA-Container-AWS

**Container su AWS** — Complete course materials for deploying and managing containers on Amazon Web Services using ECS, EKS, and Fargate.

## 📚 About This Repository

This repository contains comprehensive educational materials for a 40-hour course on **containers on AWS**, including hands-on labs, presentation slides, and assessment materials designed for practical cloud computing learning. The course follows a project-based approach with a continuous mini-project ("hello-api") threaded through all lessons.

## 📂 Repository Structure

### **labs/**
Hands-on laboratory exercises (Italian) covering:

| Lab | Topic |
|---|---|
| [01](labs/01_containers_fundamentals_warmup.md) | Containers fundamentals — Docker warmup |
| [02](labs/02_docker_basics_build_run.md) | Docker basics — build, run, logs |
| [03](labs/03_aws_setup_console_tour.md) | AWS setup — console tour and CLI |
| [04](labs/04_ecs_first_task_fargate.md) | ECS first task on Fargate |
| [05](labs/05_ecr_push_pull_scan.md) | ECR — push, pull, scan |
| [06](labs/06_ecs_cluster_task_definition_service.md) | ECS cluster, task definition, service |
| [07](labs/07_ecs_service_deploy_scaling_rollback.md) | ECS service — deploy, scaling, rollback |
| [08](labs/08_networking_vpc_subnets_security_groups.md) | Networking — VPC, subnets, Security Groups |
| [09](labs/09_ecs_alb_target_groups_healthchecks.md) | ECS + ALB — target groups, health checks |
| [10](labs/10_iam_roles_and_secrets_ecs.md) | IAM roles and secrets for ECS |
| [11](labs/11_cloudwatch_logs_metrics_alarms.md) | CloudWatch — logs, metrics, alarms |
| [12](labs/12_fargate_patterns_worker_scheduled.md) | Fargate patterns — worker, scheduled tasks |
| [12b](labs/12b_eks_deploy_quick_check.md) | EKS deploy quick check (kubectl + cleanup) |
| [13](labs/13_cicd_codebuild_codepipeline.md) | CI/CD — CodeBuild + CodePipeline |
| [14](labs/14_versioning_update_rollback.md) | Versioning, update, rollback |
| [15](labs/15_resilience_failure_injection.md) | Resilience and failure injection |
| [16](labs/16_cost_governance_quick_wins.md) | Cost governance — quick wins |

## 👨‍💻 Author

**Seyedhossein Javadizavieh**

📧 seyedhossein.javadizavieh@its-ictpiemonte.it

🔗 [LinkedIn Profile](https://www.linkedin.com/in/seyedhosseinjavadizavieh)

## 🎓 Course Information

- **Duration**: 40 hours (17 lectures + labs)
- **Level**: Intermediate
- **Language**: Slides in English, Labs in Italian
- **Institution**: ITS ICT Piemonte
- **Course**: Tecnico superiore System Administrator — AWS Cloud Architect (B.F. 2025/2027)

## 📖 Topics Covered

1. Container fundamentals and Docker
2. AWS container landscape (ECS, EKS, Fargate)
3. Amazon Elastic Container Service (ECS) — clusters, task definitions, services
4. Amazon Elastic Container Registry (ECR)
5. Networking for containers (VPC, ALB, Security Groups)
6. Identity and Access Management (IAM) for ECS
7. Monitoring with CloudWatch (logs, metrics, alarms)
8. Fargate patterns (web API, worker, scheduled tasks)
9. Amazon Elastic Kubernetes Service (EKS) — overview and comparison
10. CI/CD pipelines with CodeBuild and CodePipeline
11. Versioning, updates, and rollback strategies
12. Resilience and fault tolerance
13. Cost optimization and governance

## 🛠️ Mini-Project: hello-api

A continuous project threaded through all lectures:
- **Endpoints**: `/`, `/health`, `/version`
- **Target architecture**: ECR ──► ECS on Fargate ──► ALB
- **Skills demonstrated**: Docker build, ECR push, ECS deploy, IAM roles, CloudWatch observability, CI/CD pipeline, cleanup discipline

## 📄 License

Educational materials for ITS ICT Piemonte — Container su AWS course.

---

*Last Updated: February 2026*
