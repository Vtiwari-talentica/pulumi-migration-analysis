# Migration Analysis: central-ops-cluster-access 

## Executive Summary

**DISCOVERY**: The central-ops-cluster-access repository is a **Pulumi infrastructure project** that provides cluster access management for the central operations Kubernetes cluster. This repository contains TypeScript infrastructure code for OIDC authentication, RBAC, and cluster access configuration.

**Migration Impact**: **DIRECT PULUMI MIGRATION REQUIRED** - This is a Pulumi project that uses Pulumi Cloud backend and requires state migration.

**Relationship to Pulumi Migration**: This repository represents **cluster access infrastructure** that provides authentication and authorization for users accessing the central operations cluster.

## Repository Overview

**Repository**: central-ops-cluster-access  
**Purpose**: Cluster access management with OIDC authentication and RBAC  
**Technology**: Pulumi + TypeScript + Kubernetes + AWS EKS Identity Provider  
**Components**: EKS Identity Provider + Group Cluster Role Binding + OIDC Configuration  
**Architecture**: Centralized cluster access management with multi-environment support

### Repository Structure
```
central-ops-cluster-access/
├── src/
│   ├── index.ts              # Main Pulumi program entry point
│   └── config.ts             # Configuration and environment setup
├── Pulumi.yaml               # Pulumi project configuration
├── Pulumi.dev.yaml           # Development environment configuration
├── Pulumi.prod.yaml          # Production environment configuration
├── package.json              # Node.js dependencies
└── tsconfig.json             # TypeScript configuration
```

## Technical Architecture Analysis

### Pulumi Infrastructure Pattern
This repository implements a **centralized cluster access management pattern**:

1. **EKS Identity Provider**: Configures OIDC authentication for the cluster
2. **RBAC Management**: Maps user groups to cluster roles
3. **Environment Separation**: Different configurations for dev/prod environments
4. **Dependency Management**: Integrates with central-ops and saas-dex stacks

### Key Components

#### 1. **EKS Identity Provider Configuration**
- **Purpose**: Enables OIDC authentication for the Kubernetes cluster
- **Integration**: Uses Dex (saas-dex stack) as the identity provider
- **Configuration**: Maps OIDC issuer URL and client ID to EKS cluster
- **Security**: Provides secure authentication without storing credentials

#### 2. **Group Cluster Role Binding**
- **Purpose**: Maps user groups to Kubernetes cluster roles
- **Admin Access**: Maps cluster admin group to cluster-admin role
- **Environment Separation**: Different group IDs for dev/prod environments
- **Security**: Ensures proper authorization for cluster access

#### 3. **OIDC Integration**
- **Identity Provider**: Integration with Dex (saas-dex stack)
- **Client Configuration**: Uses central-ops static client from Dex
- **Issuer URL**: Dynamic issuer host from Dex stack reference
- **Authentication Flow**: Enables seamless user authentication

### Environment Configuration Analysis

#### Development Environment (dev)
- **Group ID**: d6dbcb8c-dece-4463-a784-6e6cd976deac (SaaS Developer group)
- **Cluster Issuer**: Production issuer (letsencrypt-prod)
- **Domain**: central-ops-dev.int.cequence.ai
- **Features**: Development access with production-level SSL

#### Production Environment (prod)
- **Group ID**: 853d76e0-df8d-4505-9f3a-bb2ae81ec7a5 (SaaS Operations group)
- **Cluster Issuer**: Production issuer (letsencrypt-prod)
- **Domain**: central-ops.int.cequence.ai
- **Features**: Production access with restricted admin group

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
- **Configuration migration**: Environment-specific configurations must be migrated

### Infrastructure Components Requiring Migration
1. **EKS Identity Provider**: OIDC authentication configuration
2. **Group Cluster Role Binding**: RBAC mapping configuration
3. **Kubernetes Resources**: Cluster role bindings and access controls
4. **OIDC Configuration**: Authentication and authorization setup

### Indirect Migration Impact: HIGH
- **Cluster Access Dependency**: All cluster access depends on this configuration
- **User Authentication Risk**: OIDC authentication may be affected during migration
- **Configuration Updates Required**: Stack references and OIDC configurations must be updated
- **Timing Coordination**: Must be migrated after central-ops and saas-dex migrations

## Migration Strategy and Recommendations

### Pre-Migration Preparation
1. **State Backup**: Export current Pulumi state for rollback capability
2. **Configuration Audit**: Document all OIDC and RBAC configurations
3. **Dependency Validation**: Ensure central-ops and saas-dex cluster migrations are complete
4. **User Access Documentation**: Document current user group mappings
5. **Authentication Testing**: Verify OIDC authentication flows

### Migration Execution Steps
1. **Update Stack References**: Modify central-ops and saas-dex cluster references to new backend
2. **State Migration**: Migrate Pulumi state to new backend
3. **Configuration Migration**: Transfer OIDC and RBAC configurations
4. **Infrastructure Validation**: Verify EKS Identity Provider configuration
5. **Authentication Testing**: Test OIDC login and user group mappings

### Post-Migration Validation
1. **OIDC Authentication**: Verify OIDC authentication flows work correctly
2. **RBAC Functionality**: Test user group to role mappings
3. **Cluster Access**: Validate cluster access for different user groups
4. **User Experience**: Test end-to-end authentication and authorization
5. **Security Validation**: Verify access controls are properly enforced

## Risk Assessment and Mitigation

### High-Risk Areas
1. **OIDC Authentication**: EKS Identity Provider configuration
   - **Mitigation**: Test authentication flows before migration
   - **Rollback**: Restore original OIDC configuration

2. **User Access Disruption**: RBAC configuration changes
   - **Mitigation**: Document current user group mappings
   - **Rollback**: Restore original RBAC configurations

3. **Stack Dependencies**: Dependencies on central-ops and saas-dex
   - **Mitigation**: Ensure prerequisite migrations are complete and stable
   - **Rollback**: Revert to original stack references

### Medium-Risk Areas
1. **Group ID Mappings**: User group to role mappings
   - **Mitigation**: Document current group configurations
   - **Rollback**: Restore original group mappings

2. **Cluster Issuer Configuration**: SSL certificate management
   - **Mitigation**: Verify cluster issuer configuration
   - **Rollback**: Restore original issuer configuration

## Timeline Integration with Overall Migration

### Phase Alignment
- **Pre-Migration (Week 1)**: Central-ops and saas-dex cluster migrations completion
- **Migration Execution (Week 2)**: central-ops-cluster-access state and configuration migration
- **Post-Migration (Week 3)**: Authentication and access validation
- **Ongoing**: Monitor cluster access and user authentication

### Dependencies
- **Prerequisites**: central-ops and saas-dex cluster migrations must be completed first
- **Coordination**: Align with platform operations team for access management
- **Validation**: Include platform operations team in post-migration testing

## Key Insights for Overall Migration Strategy

### Cluster Access Management Complexity
The central-ops-cluster-access repository reveals **critical access management** requirements:
- **OIDC Integration**: Complex authentication setup with Dex identity provider
- **Multi-Environment Support**: Different user groups for dev/prod environments
- **RBAC Management**: User group to Kubernetes role mappings
- **Security Integration**: EKS Identity Provider for secure cluster access

### User Access Impact Considerations
- **Authentication Dependencies**: All cluster access depends on OIDC configuration
- **User Group Management**: Different access levels for different user groups
- **Security Requirements**: Proper authorization and access control enforcement
- **User Experience**: Seamless authentication and authorization flows

### Resource Requirements
- **EKS Integration**: EKS Identity Provider configuration
- **OIDC Configuration**: Authentication and authorization setup
- **RBAC Resources**: Cluster role bindings and access controls
- **Network Resources**: OIDC issuer URL and client configuration

## Strategic Recommendations

### 1. Treat as Critical Access Infrastructure
- central-ops-cluster-access is **essential for cluster access**
- **High priority** for migration due to user access dependencies
- Ensure **minimal access disruption** during migration

### 2. Coordinate with Platform Operations Team
- **Include platform operations team** in migration planning
- **Schedule migration windows** during low-access periods
- **Prepare rollback procedures** for access management

### 3. Enhanced Testing and Validation
- **Comprehensive authentication testing** post-migration
- **RBAC validation** for all user groups and access levels
- **User experience testing** for authentication flows

### 4. Long-term Operational Considerations
- **Document access dependencies** on this infrastructure
- **Establish monitoring** for cluster access and authentication
- **Create procedures** for coordinated access management changes

## Conclusion

The central-ops-cluster-access repository represents **critical access infrastructure** that provides authentication and authorization for the central operations cluster. This repository requires **direct Pulumi migration** and has **high user impact** by providing:

- **OIDC authentication** for secure cluster access
- **RBAC management** for user group to role mappings
- **Multi-environment support** with different access levels
- **Security integration** with EKS Identity Provider

This discovery reinforces that the Pulumi migration affects not just infrastructure but also **user access and authentication**. The migration strategy must account for both the technical infrastructure migration and the **user access continuity** requirements that are essential for platform operations.

The central-ops-cluster-access repository is the **access control layer** that enables secure cluster access for all platform users, making it a critical component in the migration project that directly impacts user experience and security.

