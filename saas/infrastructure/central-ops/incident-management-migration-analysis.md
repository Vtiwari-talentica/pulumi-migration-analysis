# Migration Analysis: central-ops-incident-management 

## Executive Summary

**DISCOVERY**: The central-ops-incident-management repository is a **Pulumi infrastructure project** that deploys an incident management platform to the central operations Kubernetes cluster. This repository contains TypeScript infrastructure code for incident bot, UI, PostgreSQL database, and various integrations.

**Migration Impact**: **DIRECT PULUMI MIGRATION REQUIRED** - This is a Pulumi project that uses Pulumi Cloud backend and requires state migration.

**Relationship to Pulumi Migration**: This repository represents **incident management infrastructure** that provides incident response and management capabilities for the platform operations team.

## Repository Overview

**Repository**: central-ops-incident-management  
**Purpose**: Incident management platform with bot, UI, and database  
**Technology**: Pulumi + TypeScript + Kubernetes + PostgreSQL + Helm Charts  
**Components**: Incident Bot + UI + PostgreSQL + Ingress + RBAC + Integrations  
**Architecture**: Centralized incident management with multi-environment support

### Repository Structure
```
central-ops-incident-management/
├── src/
│   ├── index.ts              # Main Pulumi program entry point
│   ├── config.ts             # Configuration and environment setup
│   ├── im.ts                 # Incident management bot and UI deployment
│   ├── postgres.ts           # PostgreSQL database infrastructure
│   ├── ingress.ts            # Ingress and networking configuration
│   └── namespaceAccess.ts    # Kubernetes RBAC configuration
├── Pulumi.yaml               # Pulumi project configuration
├── Pulumi.dev.yaml           # Development environment configuration
├── Pulumi.prod.yaml          # Production environment configuration
├── package.json              # Node.js dependencies
└── tsconfig.json             # TypeScript configuration
```

## Technical Architecture Analysis

### Pulumi Infrastructure Pattern
This repository implements a **centralized incident management pattern**:

1. **Incident Bot**: Slack-based incident management bot with Helm deployment
2. **Web UI**: Incident management console with separate Helm deployment
3. **PostgreSQL Database**: Dedicated RDS instance for incident data
4. **Integrations**: Multiple third-party service integrations

### Key Components

#### 1. **Incident Management Bot**
- **Purpose**: Slack-based incident management and response bot
- **Deployment**: Helm chart `incident-management/incidentbot` (version 1.0.1)
- **Features**: Channel management, slash commands, incident tracking
- **Integrations**: Slack, PagerDuty, Atlassian, Teams, Zendesk
- **Authentication**: SAML integration with Microsoft Azure AD

#### 2. **Incident Management UI**
- **Purpose**: Web-based incident management console
- **Deployment**: Helm chart `incident-bot-console/incidentbot-console` (version 0.0.1)
- **Features**: Incident dashboard, management interface
- **Access**: IP allowlisted with SSL certificates
- **Authentication**: SAML integration

#### 3. **PostgreSQL Database Infrastructure**
- **Engine**: AWS RDS PostgreSQL 16.4
- **Instance**: db.t4g.large (configurable)
- **Storage**: 20GB GP3 encrypted storage
- **Security**: VPC security groups, encrypted at rest
- **Backup**: 7-day retention, automated backups

#### 4. **Third-Party Integrations**
- **Slack**: Bot tokens, app tokens, user tokens for incident communication
- **PagerDuty**: API integration for incident escalation
- **Atlassian**: Confluence integration for postmortem documentation
- **Microsoft Teams**: Meeting integration for incident calls
- **Zendesk**: Ticket management integration
- **Vitally**: Customer success integration
- **Grafana**: Monitoring and observability integration

### Environment Configuration Analysis

#### Development Environment (dev)
- **Channel Prefix**: tst (test channels)
- **Slash Command**: /im-test
- **Digest Channel**: incident-management-dev-notifications
- **UI Tag**: ops-dev-latest
- **Teams Integration**: Disabled
- **SAML Domain**: bot.im-dev.int.cequence.ai

#### Production Environment (prod)
- **Channel Prefix**: inc (incident channels)
- **Slash Command**: /im
- **Digest Channel**: incident-management-v2-notifications
- **UI Tag**: ops-prod-latest
- **Teams Integration**: Enabled
- **SAML Domain**: bot.im.int.cequence.ai

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
- **Secret management**: Multiple encrypted configuration values must be migrated

### Infrastructure Components Requiring Migration
1. **AWS RDS Instance**: PostgreSQL database with 7-day backup retention
2. **Kubernetes Resources**: Namespaces, secrets, ingress, RBAC
3. **Helm Releases**: Incident bot and UI application deployments
4. **Security Groups**: Database and network security configurations
5. **Container Registry Secrets**: GitLab registry authentication

### Indirect Migration Impact: MEDIUM
- **Central Operations Dependency**: Requires central-ops cluster to be migrated first
- **Service Disruption Risk**: Incident management capabilities may be affected
- **Configuration Updates Required**: Stack references and cluster configurations must be updated
- **Timing Coordination**: Must be migrated after central-ops cluster migration

## Migration Strategy and Recommendations

### Pre-Migration Preparation
1. **State Backup**: Export current Pulumi state for rollback capability
2. **Configuration Audit**: Document all encrypted configuration values and integrations
3. **Dependency Validation**: Ensure central-ops cluster migration is complete
4. **Integration Testing**: Verify all third-party integrations are working
5. **Data Backup**: Backup incident management data and configurations

### Migration Execution Steps
1. **Update Stack References**: Modify central-ops cluster references to new backend
2. **State Migration**: Migrate Pulumi state to new backend
3. **Configuration Migration**: Transfer all encrypted configuration values
4. **Infrastructure Validation**: Verify all components are operational
5. **Integration Testing**: Test all third-party service integrations

### Post-Migration Validation
1. **Database Connectivity**: Verify PostgreSQL connection and data integrity
2. **Bot Functionality**: Test Slack bot commands and incident management
3. **UI Access**: Verify web UI accessibility and functionality
4. **Integration Testing**: Test all third-party service integrations
5. **SAML Authentication**: Verify SAML login and user access

## Risk Assessment and Mitigation

### High-Risk Areas
1. **Database Migration**: RDS instance with 7-day backup retention
   - **Mitigation**: Create final snapshot before migration
   - **Rollback**: Restore from snapshot if migration fails

2. **Integration Dependencies**: Multiple third-party service integrations
   - **Mitigation**: Test all integrations before migration
   - **Rollback**: Restore original integration configurations

3. **SAML Authentication**: Complex SAML configuration with Azure AD
   - **Mitigation**: Document SAML configuration before migration
   - **Rollback**: Restore original SAML settings

### Medium-Risk Areas
1. **Secret Management**: Multiple encrypted configuration values
   - **Mitigation**: Document all secrets before migration
   - **Rollback**: Re-encrypt secrets in original backend

2. **Helm Chart Updates**: Custom incident management charts
   - **Mitigation**: Test chart compatibility before migration
   - **Rollback**: Revert to previous chart versions

3. **Network Configuration**: Ingress and SSL certificate management
   - **Mitigation**: Document current network configuration
   - **Rollback**: Restore original ingress and SSL settings

## Timeline Integration with Overall Migration

### Phase Alignment
- **Pre-Migration (Week 1)**: Central-ops cluster migration completion
- **Migration Execution (Week 2)**: central-ops-incident-management state and configuration migration
- **Post-Migration (Week 3)**: Incident management validation and integration testing
- **Ongoing**: Monitor incident management operations and integrations

### Dependencies
- **Prerequisite**: central-ops cluster migration must be completed first
- **Coordination**: Align with incident management team for service windows
- **Validation**: Include incident management team in post-migration testing

## Key Insights for Overall Migration Strategy

### Incident Management Platform Complexity
The central-ops-incident-management repository reveals **operational incident management** requirements:
- **Multi-Integration Platform**: Complex integration with 6+ third-party services
- **SAML Authentication**: Enterprise authentication with Azure AD
- **Multi-Environment Support**: Different configurations for dev/prod
- **Database Dependencies**: Dedicated PostgreSQL instance for incident data

### Operational Impact Considerations
- **Incident Response Dependencies**: Platform incident management capabilities
- **Integration Continuity**: Multiple third-party service integrations must remain functional
- **User Access Management**: SAML authentication affects incident management users
- **Data Integrity**: Incident management data must be preserved

### Resource Requirements
- **Database Resources**: PostgreSQL RDS instance with backup requirements
- **Integration Resources**: API tokens and credentials for multiple services
- **Network Resources**: Ingress rules, SSL certificates, and IP allowlisting
- **Authentication Resources**: SAML configuration and certificate management

## Strategic Recommendations

### 1. Treat as Critical Operations Infrastructure
- central-ops-incident-management provides **essential incident management capabilities**
- **High priority** for migration due to operational dependencies
- Ensure **minimal service disruption** during migration

### 2. Coordinate with Incident Management Team
- **Include incident management team** in migration planning
- **Schedule migration windows** during low-incident periods
- **Prepare rollback procedures** for incident management operations

### 3. Enhanced Testing and Validation
- **Comprehensive integration testing** post-migration
- **Incident management validation** for all bot and UI functionality
- **SAML authentication testing** for user access

### 4. Long-term Operational Considerations
- **Document incident management dependencies** on this infrastructure
- **Establish monitoring** for incident management operations and integrations
- **Create procedures** for coordinated infrastructure and incident management changes

## Conclusion

The central-ops-incident-management repository represents **critical operational infrastructure** that provides incident management and response capabilities for the platform. This repository requires **direct Pulumi migration** and significantly impacts **operational incident management** by providing:

- **Incident management platform** with Slack bot and web UI
- **Multi-service integrations** for comprehensive incident response
- **SAML authentication** for enterprise user access
- **Dedicated database** for incident data management

This discovery reinforces that the Pulumi migration affects not just customer-facing applications but also **internal operational capabilities** that are essential for incident management and response. The migration strategy must account for both the technical infrastructure migration and the **operational continuity requirements** of incident management systems.

The central-ops-incident-management repository is the **incident response layer** that enables effective incident management for the platform, making it a critical component in the migration project that directly impacts operational resilience.

