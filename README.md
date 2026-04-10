# ACA-Container-AWS

**Container su AWS** - Complete course materials for deploying and managing containers on Amazon Web Services using ECS, EKS, and Fargate.

## About This Repository

This repository contains comprehensive educational materials for a 40-hour course on **containers on AWS**, including hands-on labs, presentation slides, and assessment materials designed for practical cloud computing learning. The course follows a project-based approach with a continuous mini-project ("hello-api") threaded through all lessons.

## Repository Structure

### **labs/** - Hands-on exercises (Italian)

| #   | Lab                                                                      | Topic                                          |
| --- | ------------------------------------------------------------------------ | ---------------------------------------------- |
| 00  | [Cheat Sheet](labs/00_cheat_sheet_docker_ecs_k8s.md)                     | Docker, ECS & K8s formats - **print and keep** |
| 01  | [Containers Fundamentals](labs/01_containers_fundamentals_warmup.md)     | Docker warmup - images vs containers           |
| 02  | [Docker Basics](labs/02_docker_basics_build_run.md)                      | Build, run, logs, debug                        |
| 03  | [AWS Setup](labs/03_aws_setup_console_tour.md)                           | Console tour and CLI                           |
| 04  | [ECS First Task](labs/04_ecs_first_task_fargate.md)                      | First task on Fargate                          |
| 05  | [ECR](labs/05_ecr_push_pull_scan.md)                                     | Push, pull, scan                               |
| 06  | [ECS Cluster & Task Def](labs/06_ecs_cluster_task_definition_service.md) | Cluster, task definition, service              |
| 07  | [ECS Service](labs/07_ecs_service_deploy_scaling_rollback.md)            | Deploy, scaling, rollback                      |
| 08  | [Networking](labs/08_networking_vpc_subnets_security_groups.md)          | VPC, subnets, Security Groups                  |
| 09  | [ALB + Health Checks](labs/09_ecs_alb_target_groups_healthchecks.md)     | Target groups, health checks                   |
| 10  | [IAM & Secrets](labs/10_iam_roles_and_secrets_ecs.md)                    | Roles and secrets for ECS                      |
| 11  | [CloudWatch](labs/11_cloudwatch_logs_metrics_alarms.md)                  | Logs, metrics, alarms                          |
| 12  | [Fargate Patterns](labs/12_fargate_patterns_worker_scheduled.md)         | Worker, scheduled tasks                        |
| 12b | [EKS Quick Check](labs/12b_eks_deploy_quick_check.md)                    | kubectl deploy + cleanup                       |
| 13  | [CI/CD](labs/13_cicd_codebuild_codepipeline.md)                          | CodeBuild + CodePipeline                       |
| 14  | [Versioning](labs/14_versioning_update_rollback.md)                      | Update and rollback                            |
| 15  | [Resilience](labs/15_resilience_failure_injection.md)                    | Failure injection                              |
| 16  | [Cost Governance](labs/16_cost_governance_quick_wins.md)                 | Quick wins                                     |

## Author

**Seyedhossein Javadizavieh**

seyedhossein.javadizavieh@its-ictpiemonte.it | [LinkedIn](https://www.linkedin.com/in/seyedhosseinjavadizavieh)

## Course Information

- **Duration**: 40 hours (17 lectures + labs)
- **Level**: Intermediate
- **Language**: Slides & demos in English, Labs in Italian
- **Institution**: ITS ICT Piemonte
- **Course**: Tecnico superiore System Administrator - AWS Cloud Architect (B.F. 2025/2027)

## Topics Covered

1. Container fundamentals and Docker
2. AWS container landscape (ECS, EKS, Fargate)
3. Amazon Elastic Container Service (ECS) - clusters, task definitions, services
4. Amazon Elastic Container Registry (ECR)
5. Networking for containers (VPC, ALB, Security Groups)
6. Identity and Access Management (IAM) for ECS
7. Monitoring with CloudWatch (logs, metrics, alarms)
8. Fargate patterns (web API, worker, scheduled tasks)
9. Amazon Elastic Kubernetes Service (EKS) - overview and comparison
10. CI/CD pipelines with CodeBuild and CodePipeline
11. Versioning, updates, and rollback strategies
12. Resilience and fault tolerance
13. Cost optimization and governance

## Mini-Project: hello-api

A continuous project threaded through all lectures:

- **Port**: 9090 (avoids conflicts with common local services)
- **Endpoints**: `/`, `/health`, `/version`
- **Target architecture**: ECR → ECS on Fargate → ALB
- **Skills demonstrated**: Docker build, ECR push, ECS deploy, IAM roles, CloudWatch observability, CI/CD pipeline, cleanup discipline

## License

Educational materials for ITS ICT Piemonte - Container su AWS course.

---

_Last Updated: March 2026_
