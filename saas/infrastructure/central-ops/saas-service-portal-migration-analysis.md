# Migration Analysis: central-ops-saas-service-portal 

## Executive Summary

**DISCOVERY**: The saas-service-portal repository is a **Pulumi infrastructure project** that deploys a comprehensive service portal application to the central operations Kubernetes cluster. This repository contains TypeScript infrastructure code for a Next.js application, DocumentDB database, and extensive third-party integrations.

**Migration Impact**: **DIRECT PULUMI MIGRATION REQUIRED** - This is a Pulumi project that uses Pulumi Cloud backend and requires state migration.

**Relationship to Pulumi Migration**: This repository represents **customer-facing service portal infrastructure** that provides operational dashboards, monitoring, and management capabilities for the platform.

## Repository Overview

**Repository**: saas-service-portal  
**Purpose**: Customer-facing service portal with operational dashboards and integrations  
**Technology**: Pulumi + TypeScript + Kubernetes + DocumentDB + Next.js + Multiple Integrations  
**Components**: Service Portal App + DocumentDB + Ingress + Service Monitor + 15+ Integrations  
**Architecture**: Centralized service portal with multi-environment support

### Repository Structure
```
saas-service-portal/
├── src/
│   ├── index.ts              # Main Pulumi program entry point
│   ├── config.ts             # Configuration and environment setup
│   └── servicePortalDB.ts    # DocumentDB database infrastructure
├── Pulumi.yaml               # Pulumi project configuration
├── Pulumi.dev.yaml           # Development environment configuration
├── Pulumi.prod.yaml          # Production environment configuration
├── package.json              # Node.js dependencies
└── tsconfig.json             # TypeScript configuration
```

## Technical Architecture Analysis

### Pulumi Infrastructure Pattern
This repository implements a **centralized service portal pattern**:

1. **Next.js Application**: Customer-facing service portal with operational dashboards
2. **DocumentDB Database**: MongoDB-compatible database for portal data
3. **Extensive Integrations**: 15+ third-party service integrations
4. **Environment Separation**: Different configurations for dev/prod environments

### Key Components

#### 1. **Service Portal Application**
- **Purpose**: Customer-facing operational dashboard and management interface
- **Technology**: Next.js application with TypeScript
- **Deployment**: Kubernetes deployment with configurable image tags
- **Resources**: 1000m-2000m CPU, 2Gi-4Gi memory
- **Features**: Operational dashboards, monitoring, cost analysis, tenant management

#### 2. **DocumentDB Database Infrastructure**
- **Engine**: AWS DocumentDB (MongoDB-compatible)
- **Instance**: db.r6g.large (configurable)
- **Storage**: DocumentDB cluster with 1 instance
- **Security**: VPC security groups, encrypted at rest
- **Backup**: 2-day retention (configurable)

#### 3. **Extensive Third-Party Integrations**
- **ArgoCD Integration**: Dev/prod ArgoCD access with read/write tokens
- **Monitoring**: Datadog, Grafana Cloud integration
- **Authentication**: Dex OIDC integration
- **Development Tools**: GitLab, Pulumi access tokens
- **AWS Services**: Cost Explorer, tenant workloads access
- **Communication**: Slack notifications
- **Customer Success**: Vitally integration
- **API Testing**: API Fury integration

#### 4. **Kubernetes Resources**
- **Deployment**: Service portal application deployment
- **Service**: ClusterIP service for internal communication
- **Ingress**: SSL-enabled ingress with IP allowlisting
- **ServiceMonitor**: Prometheus monitoring integration
- **Secrets**: Comprehensive secret management for all integrations

### Environment Configuration Analysis

#### Development Environment (dev)
- **Domain**: ops-dev.int.cequence.ai
- **Image**: registry.gitlab.com/cequence/orcx-team/portal/main:latest
- **Database Backup**: 2-day retention
- **ArgoCD URL**: https://argocd.central-ops-dev.int.cequence.ai
- **API Testing URL**: https://updates-dev.apifury.cequence.ai
- **Slack Channel**: C071RKJV3F1

#### Production Environment (prod)
- **Domain**: ops.int.cequence.ai
- **Image**: registry.gitlab.com/cequence/orcx-team/portal:{tag}
- **Database Backup**: 2-day retention
- **ArgoCD URL**: https://argocd.central-ops.int.cequence.ai
- **API Testing URL**: https://updates.apifury.cequence.ai
- **Slack Channel**: C072394VC0M

### Integration Dependencies
This repository has **extensive dependencies** on external services:

1. **Central Operations Cluster**: Referenced via `cequence/central-ops/${env}` StackReference
2. **Dex Authentication**: Referenced via `cequence/saas-dex/${authStack}` StackReference
3. **ArgoCD Instances**: Dev and prod ArgoCD access
4. **Third-Party Services**: 15+ external service integrations

## Migration Impact Assessment

### Direct Migration Impact: HIGH
- **Pulumi Cloud dependency**: Repository uses Pulumi Cloud backend for state management
- **State migration required**: All infrastructure state must be migrated to new backend
- **Stack references**: Dependencies on central-ops and saas-dex stacks must be updated
- **Secret management**: 15+ encrypted configuration values must be migrated

### Infrastructure Components Requiring Migration
1. **DocumentDB Cluster**: MongoDB-compatible database with 2-day backup retention
2. **Kubernetes Resources**: Deployment, service, ingress, secrets, service monitor
3. **IAM Roles**: AWS service access roles and policies
4. **Security Groups**: Database and network security configurations
5. **Container Registry Secrets**: GitLab registry authentication

### Indirect Migration Impact: HIGH
- **Customer-Facing Service**: Service portal is customer-facing
- **Service Disruption Risk**: Customer access to operational dashboards may be affected
- **Configuration Updates Required**: Stack references and integration configurations must be updated
- **Timing Coordination**: Must be migrated after central-ops and saas-dex migrations

## Migration Strategy and Recommendations

### Pre-Migration Preparation
1. **State Backup**: Export current Pulumi state for rollback capability
2. **Configuration Audit**: Document all 15+ encrypted configuration values
3. **Dependency Validation**: Ensure central-ops and saas-dex migrations are complete
4. **Integration Testing**: Verify all third-party integrations are working
5. **Data Backup**: Backup DocumentDB data and portal configurations

### Migration Execution Steps
1. **Update Stack References**: Modify central-ops and saas-dex references to new backend
2. **State Migration**: Migrate Pulumi state to new backend
3. **Configuration Migration**: Transfer all encrypted configuration values
4. **Infrastructure Validation**: Verify all components are operational
5. **Integration Testing**: Test all third-party service integrations

### Post-Migration Validation
1. **Database Connectivity**: Verify DocumentDB connection and data integrity
2. **Portal Functionality**: Test all portal features and dashboards
3. **Integration Testing**: Test all 15+ third-party service integrations
4. **Authentication**: Verify OIDC login and user access
5. **Customer Access**: Test customer-facing portal functionality

## Risk Assessment and Mitigation

### High-Risk Areas
1. **Database Migration**: DocumentDB cluster with 2-day backup retention
   - **Mitigation**: Create final snapshot before migration
   - **Rollback**: Restore from snapshot if migration fails

2. **Integration Dependencies**: 15+ third-party service integrations
   - **Mitigation**: Test all integrations before migration
   - **Rollback**: Restore original integration configurations

3. **Customer Impact**: Customer-facing service portal
   - **Mitigation**: Plan migration during low-usage periods
   - **Rollback**: Prepare rapid restoration procedures

### Medium-Risk Areas
1. **Secret Management**: Multiple encrypted configuration values
   - **Mitigation**: Document all secrets before migration
   - **Rollback**: Re-encrypt secrets in original backend

2. **ArgoCD Integration**: Complex ArgoCD access configuration
   - **Mitigation**: Verify ArgoCD connectivity before migration
   - **Rollback**: Restore original ArgoCD configuration

3. **Authentication Integration**: Dex OIDC integration
   - **Mitigation**: Test authentication flows before migration
   - **Rollback**: Restore original OIDC configuration

## Timeline Integration with Overall Migration

### Phase Alignment
- **Pre-Migration (Week 1)**: Central-ops and saas-dex cluster migrations completion
- **Migration Execution (Week 2)**: saas-service-portal state and configuration migration
- **Post-Migration (Week 3)**: Portal validation and integration testing
- **Ongoing**: Monitor portal operations and customer access

### Dependencies
- **Prerequisites**: central-ops and saas-dex cluster migrations must be completed first
- **Coordination**: Align with customer success team for portal service windows
- **Validation**: Include customer success team in post-migration testing

## Key Insights for Overall Migration Strategy

### Service Portal Platform Complexity
The saas-service-portal repository reveals **comprehensive operational platform** requirements:
- **Customer-Facing Interface**: Direct customer access to operational dashboards
- **Multi-Integration Platform**: Complex integration with 15+ third-party services
- **Operational Dashboards**: Cost analysis, monitoring, tenant management
- **Multi-Environment Support**: Different configurations for dev/prod

### Customer Impact Considerations
- **Customer Access Dependencies**: Customers depend on portal for operational insights
- **Integration Continuity**: Multiple third-party service integrations must remain functional
- **Data Integrity**: Portal data and configurations must be preserved
- **User Experience**: Portal functionality must remain seamless

### Resource Requirements
- **Database Resources**: DocumentDB cluster with backup requirements
- **Compute Resources**: High-performance Next.js application (2-4 CPU, 2-4Gi memory)
- **Integration Resources**: API tokens and credentials for 15+ services
- **Network Resources**: Ingress rules, SSL certificates, and IP allowlisting

## Strategic Recommendations

### 1. Treat as Critical Customer-Facing Infrastructure
- saas-service-portal provides **essential customer operational capabilities**
- **High priority** for migration due to customer dependencies
- Ensure **minimal customer impact** during migration

### 2. Coordinate with Customer Success Team
- **Include customer success team** in migration planning
- **Schedule migration windows** during low-customer-usage periods
- **Prepare rollback procedures** for customer portal access

### 3. Enhanced Testing and Validation
- **Comprehensive portal testing** post-migration
- **Integration validation** for all 15+ third-party services
- **Customer experience testing** for portal functionality

### 4. Long-term Operational Considerations
- **Document portal dependencies** on this infrastructure
- **Establish monitoring** for portal operations and customer access
- **Create procedures** for coordinated portal infrastructure changes

## Conclusion

The saas-service-portal repository represents **critical customer-facing infrastructure** that provides operational dashboards, monitoring, and management capabilities for platform customers. This repository requires **direct Pulumi migration** and significantly impacts **customer experience** by providing:

- **Customer-facing service portal** with operational dashboards
- **Multi-service integrations** for comprehensive platform management
- **DocumentDB database** for portal data and configurations
- **Extensive third-party integrations** for operational insights

This discovery reinforces that the Pulumi migration affects not just internal infrastructure but also **customer-facing services** that are essential for customer success. The migration strategy must account for both the technical infrastructure migration and the **customer experience continuity** requirements of the service portal.

The saas-service-portal repository is the **customer interface layer** that enables customers to manage and monitor their platform usage, making it a critical component in the migration project that directly impacts customer satisfaction and platform adoption.

