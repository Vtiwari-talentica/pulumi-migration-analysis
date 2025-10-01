# Migration Analysis: central-ops-n8n

## Executive Summary

**DISCOVERY**: The central-ops-n8n repository is a **Pulumi infrastructure project** that deploys n8n workflow automation platform and Iris security operations platform to a central operations Kubernetes cluster. This repository contains TypeScript infrastructure code with AWS RDS, Kubernetes, and Helm chart deployments.

**Migration Impact**: **DIRECT PULUMI MIGRATION REQUIRED** - This is a Pulumi project that uses Pulumi Cloud backend and requires state migration.

**Relationship to Pulumi Migration**: This repository represents **central operations infrastructure** that provides workflow automation and security operations capabilities to the overall platform.

## Repository Overview

**Repository**: central-ops-n8n  
**Purpose**: Central operations n8n workflow automation and Iris security platform  
**Technology**: Pulumi + TypeScript + Kubernetes + AWS RDS + Helm Charts  
**Components**: n8n workflow automation + Iris security operations + PostgreSQL database  
**Architecture**: Centralized operations platform with multi-environment support (dev/prod)

### Repository Structure
```
central-ops-n8n/
├── src/
│   ├── index.ts              # Main Pulumi program entry point
│   ├── config.ts             # Configuration and environment setup
│   ├── n8n.ts                # n8n workflow platform deployment
│   ├── iris.ts               # Iris security operations platform
│   └── namespaceAccess.ts    # Kubernetes RBAC configuration
├── Pulumi.yaml               # Pulumi project configuration
├── Pulumi.dev.yaml           # Development environment configuration
├── Pulumi.prod.yaml          # Production environment configuration
├── package.json              # Node.js dependencies
└── tsconfig.json             # TypeScript configuration
```

## Technical Architecture Analysis

### Pulumi Infrastructure Pattern
This repository implements a **centralized operations platform pattern**:

1. **Central Operations Cluster**: Deploys to a shared Kubernetes cluster via StackReference
2. **Multi-Component Architecture**: n8n + Iris + PostgreSQL + Redis
3. **Environment Separation**: Separate dev/prod configurations with different resource sizing
4. **Security Integration**: SAML authentication, IP allowlisting, encrypted secrets

### Key Components

#### 1. **n8n Workflow Automation Platform**
- **Purpose**: Workflow automation and integration platform
- **Deployment**: Helm chart from `oci://8gears.container-registry.com/library/n8n`
- **Features**: Webhook support, scaling with Redis, PostgreSQL backend
- **Security**: IP allowlisting, SSL certificates, encrypted secrets
- **Resources**: Configurable CPU/memory (250m-500m CPU, 1500Mi memory)

#### 2. **Iris Security Operations Platform**
- **Purpose**: Security operations and incident response platform
- **Deployment**: Custom Helm chart `iris-web/iris-web`
- **Features**: SAML authentication, AWS Bedrock integration, Datadog monitoring
- **Database**: Shared PostgreSQL instance with separate database
- **Integrations**: Slack, Vitally, threat intelligence APIs

#### 3. **PostgreSQL Database Infrastructure**
- **Engine**: AWS RDS PostgreSQL 16.4
- **Instance**: db.t4g.large (configurable)
- **Storage**: 20GB GP3 encrypted storage
- **Security**: VPC security groups, encrypted at rest
- **Backup**: 7-day retention, automated backups

#### 4. **Redis Caching Layer**
- **Purpose**: n8n workflow scaling and caching
- **Deployment**: Embedded Redis with persistence
- **Storage**: 4Gi persistent volume
- **Security**: Password-protected with random credentials

### Environment Configuration Analysis

#### Development Environment (dev)
- **Resource Sizing**: Standard CPU/memory allocation
- **Logging**: Debug level logging for Iris
- **User Groups**: 'Analysts' group for Iris
- **Features**: Full feature set enabled

#### Production Environment (prod)
- **Resource Sizing**: Same as dev (500m CPU, 1500Mi memory)
- **Logging**: Error level logging for Iris
- **User Groups**: 'CS-Read' group for Iris (more restrictive)
- **Features**: Production-optimized configuration

### Dependencies on Other Pulumi Infrastructure
This repository has **critical dependencies** on the central-ops cluster:

1. **Central Operations Cluster**: Referenced via `cequence/central-ops/${env}` StackReference
2. **Kubernetes Configuration**: Uses cluster kubeconfig from central-ops
3. **VPC Infrastructure**: Uses VPC ID, security groups, and subnet IDs from central-ops
4. **Network Security**: Depends on cluster security group for database access

## Migration Impact Assessment

### Direct Migration Impact: HIGH
- **Pulumi Cloud dependency**: Repository uses Pulumi Cloud backend for state management
- **State migration required**: All infrastructure state must be migrated to new backend
- **Stack references**: Dependencies on central-ops cluster must be updated
- **Secret management**: Encrypted configuration values must be migrated

### Infrastructure Components Requiring Migration
1. **AWS RDS Instance**: PostgreSQL database with 7-day backup retention
2. **Kubernetes Resources**: Namespaces, secrets, ingress, RBAC
3. **Helm Releases**: n8n and Iris application deployments
4. **IAM Resources**: Secrets Manager access keys and policies
5. **Security Groups**: Database and network security configurations

### Indirect Migration Impact: MEDIUM
- **Central Operations Dependency**: Requires central-ops cluster to be migrated first
- **Service Disruption Risk**: Workflow automation and security operations may be affected
- **Configuration Updates Required**: Stack references and cluster configurations must be updated
- **Timing Coordination**: Must be migrated after central-ops cluster migration

## Migration Strategy and Recommendations

### Pre-Migration Preparation
1. **State Backup**: Export current Pulumi state for rollback capability
2. **Configuration Audit**: Document all encrypted configuration values
3. **Dependency Validation**: Ensure central-ops cluster migration is complete
4. **Service Impact Assessment**: Identify workflows and security operations that may be affected

### Migration Execution Steps
1. **Update Stack References**: Modify central-ops cluster references to new backend
2. **State Migration**: Migrate Pulumi state to new backend
3. **Configuration Migration**: Transfer encrypted configuration values
4. **Infrastructure Validation**: Verify all components are operational
5. **Service Testing**: Validate n8n workflows and Iris functionality

### Post-Migration Validation
1. **Database Connectivity**: Verify PostgreSQL and Redis connections
2. **Application Health**: Check n8n and Iris application status
3. **Authentication**: Validate SAML and user access
4. **Integration Testing**: Test external service integrations (Slack, Datadog, etc.)
5. **Workflow Testing**: Execute sample n8n workflows

## Risk Assessment and Mitigation

### High-Risk Areas
1. **Database Migration**: RDS instance with 7-day backup retention
   - **Mitigation**: Create final snapshot before migration
   - **Rollback**: Restore from snapshot if migration fails

2. **Secret Management**: Multiple encrypted configuration values
   - **Mitigation**: Document all secrets before migration
   - **Rollback**: Re-encrypt secrets in original backend

3. **Service Dependencies**: Central-ops cluster dependency
   - **Mitigation**: Ensure central-ops migration is complete and stable
   - **Rollback**: Revert to original cluster references

### Medium-Risk Areas
1. **Helm Chart Updates**: Custom Iris chart version management
   - **Mitigation**: Test chart compatibility before migration
   - **Rollback**: Revert to previous chart version

2. **Network Configuration**: Security groups and ingress rules
   - **Mitigation**: Document current network configuration
   - **Rollback**: Restore original security group rules

## Timeline Integration with Overall Migration

### Phase Alignment
- **Pre-Migration (Week 1)**: Central-ops cluster migration completion
- **Migration Execution (Week 2)**: central-ops-n8n state and configuration migration
- **Post-Migration (Week 3)**: Service validation and testing
- **Ongoing**: Monitor workflow automation and security operations

### Dependencies
- **Prerequisite**: central-ops cluster migration must be completed first
- **Coordination**: Align with central operations team for service windows
- **Validation**: Include central operations team in post-migration testing

## Key Insights for Overall Migration Strategy

### Central Operations Platform Complexity
The central-ops-n8n repository reveals additional operational complexity:
- **Workflow Automation**: n8n platform for business process automation
- **Security Operations**: Iris platform for incident response and security management
- **Multi-Environment Support**: Separate dev/prod configurations with different access controls
- **External Integrations**: Slack, Datadog, AWS Bedrock, threat intelligence APIs

### Operational Impact Considerations
- **Business Process Dependencies**: Workflow automation may be critical for business operations
- **Security Operations**: Incident response capabilities must remain available
- **User Access Management**: SAML authentication and user group management
- **External Service Dependencies**: Multiple third-party service integrations

### Resource Requirements
- **Database Resources**: PostgreSQL RDS instance with backup requirements
- **Compute Resources**: Configurable CPU/memory for n8n and Iris workloads
- **Storage Resources**: Persistent volumes for Redis and application data
- **Network Resources**: Security groups, ingress rules, and SSL certificates

## Strategic Recommendations

### 1. Treat as Critical Operations Infrastructure
- central-ops-n8n provides **essential business operations capabilities**
- **High priority** for migration due to operational dependencies
- Ensure **minimal service disruption** during migration

### 2. Coordinate with Central Operations Team
- **Include central operations team** in migration planning
- **Schedule migration windows** during low-activity periods
- **Prepare rollback procedures** for critical workflows

### 3. Enhanced Testing and Validation
- **Comprehensive workflow testing** post-migration
- **Security operations validation** for incident response capabilities
- **Integration testing** for all external service connections

### 4. Long-term Operational Considerations
- **Document operational dependencies** on this infrastructure
- **Establish monitoring** for workflow automation and security operations
- **Create procedures** for coordinated infrastructure and application changes

## Conclusion

The central-ops-n8n repository represents **critical operational infrastructure** that provides workflow automation and security operations capabilities to the overall platform. This repository requires **direct Pulumi migration** and significantly impacts the **operational continuity** of the platform by providing:

- **Workflow automation capabilities** for business process management
- **Security operations platform** for incident response and threat management
- **Centralized operations infrastructure** with multi-environment support
- **External service integrations** for monitoring, communication, and intelligence

This discovery reinforces that the Pulumi migration affects not just customer-facing applications but also **internal operational capabilities** that are essential for platform management and security operations. The migration strategy must account for both the technical infrastructure migration and the **operational continuity requirements** of these critical business systems.
