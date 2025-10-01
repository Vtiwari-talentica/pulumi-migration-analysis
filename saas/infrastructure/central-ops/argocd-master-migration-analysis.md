# Migration Analysis: central-ops-argocd (Pulumi Infrastructure Repository)

## Executive Summary

**DISCOVERY**: The central-ops-argocd repository is a **Pulumi infrastructure project** that deploys ArgoCD GitOps platform to a central operations Kubernetes cluster. This repository contains TypeScript infrastructure code with Kubernetes, Helm charts, and OIDC authentication for managing GitOps deployments across tenant clusters.

**Migration Impact**: **DIRECT PULUMI MIGRATION REQUIRED** - This is a Pulumi project that uses Pulumi Cloud backend and requires state migration.

**Relationship to Pulumi Migration**: This repository represents **central GitOps infrastructure** that orchestrates application deployments across all tenant clusters and is critical for the overall platform's deployment automation.

## Repository Overview

**Repository**: central-ops-argocd  
**Purpose**: Central ArgoCD GitOps platform for orchestrating tenant application deployments  
**Technology**: Pulumi + TypeScript + Kubernetes + Helm Charts + OIDC Authentication  
**Components**: ArgoCD server + ApplicationSet + Repository credentials + RBAC  
**Architecture**: Centralized GitOps platform with multi-tenant cluster management

### Repository Structure
```
central-ops-argocd/
├── src/
│   ├── index.ts              # Main Pulumi program entry point
│   ├── config.ts             # Configuration and environment setup
│   ├── argocd.ts             # ArgoCD platform deployment
│   └── serviceAccount.ts     # Kubernetes RBAC and service accounts
├── Pulumi.yaml               # Pulumi project configuration
├── Pulumi.dev.yaml           # Development environment configuration
├── Pulumi.prod.yaml          # Production environment configuration
├── package.json              # Node.js dependencies
└── tsconfig.json             # TypeScript configuration
```

## Technical Architecture Analysis

### Pulumi Infrastructure Pattern
This repository implements a **centralized GitOps orchestration pattern**:

1. **Central ArgoCD Instance**: Deploys ArgoCD to central operations cluster
2. **Multi-Tenant Management**: ApplicationSet for automatic tenant cluster registration
3. **Repository Integration**: GitLab repository credentials for GitOps source
4. **Authentication**: OIDC integration with Dex for user management

### Key Components

#### 1. **ArgoCD GitOps Platform**
- **Purpose**: Central GitOps platform for managing application deployments
- **Deployment**: Helm chart from `https://argoproj.github.io/argo-helm` (version 6.2.3)
- **Features**: ApplicationSet, notifications, Redis HA, autoscaling
- **Security**: OIDC authentication, RBAC policies, IP allowlisting
- **Resources**: Configurable CPU/memory (varies by environment)

#### 2. **ApplicationSet for Tenant Management**
- **Purpose**: Automatically creates ArgoCD applications for each tenant cluster
- **Generator**: Cluster generator that discovers registered tenant clusters
- **Template**: Creates tenant-specific applications from Helm charts
- **Exclusions**: Excludes GCP clusters (managed separately)
- **Configuration**: Uses tenant-specific values from GitLab repository

#### 3. **Repository Credentials Management**
- **GitLab Integration**: Credentials for `gitlab.com/cequence/saas/infrastructure/tenants`
- **Multi-Repository Support**: Separate credentials for different repository paths
- **Security**: Secure token-based authentication
- **Scope**: Access to tenant application configurations

#### 4. **OIDC Authentication Integration**
- **Identity Provider**: Integration with Dex (saas-dex stack)
- **User Groups**: Different groups for admin and read-only access
- **Environment Separation**: Different group IDs for dev/prod environments
- **RBAC Policies**: Custom policies for ops-portal access levels

### Environment Configuration Analysis

#### Development Environment (dev)
- **Resource Sizing**: Standard CPU/memory allocation (250m-500m CPU, 1500Mi memory)
- **Account ID**: 552447114887 (dev AWS account)
- **User Groups**: SaaS Developer group for admin access
- **Features**: Full development feature set

#### Production Environment (prod)
- **Resource Sizing**: High-performance allocation (2000m-4000m CPU, 8Gi-12Gi memory)
- **Account ID**: 545681961293 (prod AWS account)
- **User Groups**: SaaS Operations group for admin access
- **Features**: Production-optimized with enhanced monitoring

### Dependencies on Other Pulumi Infrastructure
This repository has **critical dependencies** on multiple Pulumi stacks:

1. **Central Operations Cluster**: Referenced via `cequence/central-ops/${env}` StackReference
2. **Dex Authentication**: Referenced via `cequence/saas-dex/${authStack}` StackReference
3. **Kubernetes Configuration**: Uses cluster kubeconfig from central-ops
4. **OIDC Configuration**: Uses issuer host and static clients from Dex stack

## Migration Impact Assessment

### Direct Migration Impact: HIGH
- **Pulumi Cloud dependency**: Repository uses Pulumi Cloud backend for state management
- **State migration required**: All infrastructure state must be migrated to new backend
- **Stack references**: Dependencies on central-ops and saas-dex stacks must be updated
- **Secret management**: Encrypted configuration values must be migrated

### Infrastructure Components Requiring Migration
1. **ArgoCD Helm Release**: Complete ArgoCD platform deployment
2. **Kubernetes Resources**: Namespaces, secrets, RBAC, service accounts
3. **ApplicationSet**: Multi-tenant cluster management configuration
4. **Repository Credentials**: GitLab integration secrets
5. **OIDC Configuration**: Authentication and authorization setup

### Indirect Migration Impact: CRITICAL
- **GitOps Platform Dependency**: All tenant application deployments depend on this ArgoCD instance
- **Service Disruption Risk**: GitOps operations may be affected during migration
- **Configuration Updates Required**: Stack references and authentication configurations must be updated
- **Timing Coordination**: Must be migrated after central-ops and saas-dex migrations

## Migration Strategy and Recommendations

### Pre-Migration Preparation
1. **State Backup**: Export current Pulumi state for rollback capability
2. **Configuration Audit**: Document all encrypted configuration values and environment variables
3. **Dependency Validation**: Ensure central-ops and saas-dex cluster migrations are complete
4. **GitOps State Documentation**: Document current application deployment state
5. **Repository Access Validation**: Verify GitLab repository credentials are still valid

### Migration Execution Steps
1. **Update Stack References**: Modify central-ops and saas-dex cluster references to new backend
2. **State Migration**: Migrate Pulumi state to new backend
3. **Configuration Migration**: Transfer encrypted configuration values and environment variables
4. **Infrastructure Validation**: Verify all ArgoCD components are operational
5. **GitOps Testing**: Validate ApplicationSet and repository connectivity

### Post-Migration Validation
1. **ArgoCD Health**: Verify ArgoCD server and all components are healthy
2. **ApplicationSet Functionality**: Test automatic tenant application creation
3. **Repository Connectivity**: Validate GitLab repository access
4. **Authentication**: Test OIDC login and user group assignments
5. **Tenant Cluster Management**: Verify tenant cluster registration and management

## Risk Assessment and Mitigation

### Critical-Risk Areas
1. **GitOps Platform Disruption**: ArgoCD is critical for all application deployments
   - **Mitigation**: Plan migration during low-activity maintenance windows
   - **Rollback**: Prepare rapid restoration procedures for GitOps operations

2. **ApplicationSet Configuration**: Complex multi-tenant management setup
   - **Mitigation**: Document current ApplicationSet configuration before migration
   - **Rollback**: Restore ApplicationSet configuration from backup

3. **OIDC Authentication**: User access and authorization management
   - **Mitigation**: Test authentication flows before migration
   - **Rollback**: Revert to original OIDC configuration

### High-Risk Areas
1. **Repository Credentials**: GitLab integration for GitOps source
   - **Mitigation**: Verify repository access before migration
   - **Rollback**: Restore original repository credentials

2. **Stack Dependencies**: Dependencies on central-ops and saas-dex
   - **Mitigation**: Ensure prerequisite migrations are complete and stable
   - **Rollback**: Revert to original stack references

### Medium-Risk Areas
1. **Helm Chart Updates**: ArgoCD Helm chart version management
   - **Mitigation**: Test chart compatibility before migration
   - **Rollback**: Revert to previous chart version

2. **RBAC Configuration**: User permissions and access control
   - **Mitigation**: Document current RBAC policies
   - **Rollback**: Restore original RBAC configuration

## Timeline Integration with Overall Migration

### Phase Alignment
- **Pre-Migration (Week 1)**: Central-ops and saas-dex cluster migrations completion
- **Migration Execution (Week 2)**: central-ops-argocd state and configuration migration
- **Post-Migration (Week 3)**: GitOps platform validation and tenant application testing
- **Ongoing**: Monitor GitOps operations and tenant cluster management

### Dependencies
- **Prerequisites**: central-ops and saas-dex cluster migrations must be completed first
- **Coordination**: Align with platform operations team for GitOps service windows
- **Validation**: Include platform operations team in post-migration testing

## Key Insights for Overall Migration Strategy

### Central GitOps Platform Complexity
The central-ops-argocd repository reveals the **critical role of GitOps** in the platform:
- **Multi-Tenant Orchestration**: Manages application deployments across 100+ tenant clusters
- **Automated Cluster Management**: ApplicationSet automatically discovers and manages tenant clusters
- **Repository Integration**: Complex GitLab integration for GitOps source management
- **Authentication Integration**: OIDC integration with centralized user management

### Platform-Wide Impact Considerations
- **Application Deployment Dependencies**: All tenant applications depend on this ArgoCD instance
- **GitOps Operations**: Continuous deployment operations must remain available
- **User Access Management**: OIDC authentication affects all platform users
- **Tenant Cluster Management**: Automated tenant cluster registration and management

### Resource Requirements
- **High-Performance Requirements**: Production environment uses significant resources (4 CPU, 12Gi memory)
- **Autoscaling Configuration**: Repo server and server components with autoscaling
- **Redis HA**: High-availability Redis for ArgoCD state management
- **Network Resources**: Ingress rules, SSL certificates, and IP allowlisting

## Strategic Recommendations

### 1. Treat as Critical Platform Infrastructure
- central-ops-argocd is **essential for all application deployments**
- **Highest priority** for migration due to platform-wide dependencies
- Ensure **minimal GitOps disruption** during migration

### 2. Coordinate with Platform Operations Team
- **Include platform operations team** in migration planning
- **Schedule migration windows** during low GitOps activity periods
- **Prepare rollback procedures** for GitOps operations

### 3. Enhanced Testing and Validation
- **Comprehensive GitOps testing** post-migration
- **Tenant application validation** for all managed clusters
- **Authentication testing** for all user groups and access levels

### 4. Long-term Operational Considerations
- **Document GitOps dependencies** on this infrastructure
- **Establish monitoring** for GitOps operations and tenant cluster management
- **Create procedures** for coordinated infrastructure and GitOps changes

## Conclusion

The central-ops-argocd repository represents **critical platform infrastructure** that orchestrates all application deployments across the entire platform. This repository requires **direct Pulumi migration** and has **platform-wide impact** by providing:

- **Central GitOps platform** for managing all tenant application deployments
- **Multi-tenant cluster management** with automated discovery and registration
- **Repository integration** for GitOps source management
- **Authentication and authorization** for platform users and operations

This discovery reinforces that the Pulumi migration affects not just individual applications but the **entire platform's deployment automation**. The migration strategy must account for both the technical infrastructure migration and the **platform-wide operational continuity** of GitOps operations that are essential for all customer deployments.

The central-ops-argocd repository is the **orchestration layer** that makes the entire platform's GitOps strategy possible, making it one of the most critical components in the migration project.

