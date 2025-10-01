# Migration Analysis: saas-dex

## Executive Summary

**DISCOVERY**: The dex-master repository is a **Pulumi infrastructure project** that deploys a dedicated EKS cluster for OIDC authentication services using Dex. This repository contains TypeScript infrastructure code for a complete authentication platform with Azure AD integration, tenant management, and multi-tenant OAuth2/OIDC capabilities.

**Migration Impact**: **DIRECT PULUMI MIGRATION REQUIRED** - This is a Pulumi project that uses Pulumi Cloud backend and requires state migration.

**Relationship to Pulumi Migration**: This repository represents **critical authentication infrastructure** that provides OIDC/OAuth2 services for all other platform components and customer tenants.

## Repository Overview

**Repository**: dex-master  
**Purpose**: Centralized OIDC authentication platform with Azure AD integration  
**Technology**: Pulumi + TypeScript + EKS + Dex + Azure AD + Cert-Manager + Ingress  
**Components**: EKS Cluster + Dex + Azure AD SSO + Cert-Manager + Ingress + Multi-tenant OAuth2  
**Architecture**: Dedicated authentication cluster with multi-tenant support

### Repository Structure
```
dex-master/
├── src/
│   ├── index.ts              # Main Pulumi program entry point
│   └── cluster.ts            # EKS cluster infrastructure
├── Pulumi.yaml               # Pulumi project configuration
├── Pulumi.dev.yaml           # Development environment configuration
├── Pulumi.prod.yaml          # Production environment configuration
├── package.json              # Node.js dependencies
└── tsconfig.json             # TypeScript configuration
```

## Technical Architecture Analysis

### Pulumi Infrastructure Pattern
This repository implements a **dedicated authentication cluster pattern**:

1. **EKS Cluster**: Dedicated Kubernetes cluster for authentication services
2. **Dex OIDC Provider**: OpenID Connect identity provider
3. **Azure AD Integration**: Enterprise SSO with Azure Active Directory
4. **Multi-tenant OAuth2**: Support for multiple customer tenants and applications
5. **Certificate Management**: Automated SSL/TLS certificate provisioning

### Key Components

#### 1. **EKS Authentication Cluster**
- **Purpose**: Dedicated Kubernetes cluster for authentication services
- **Version**: Kubernetes 1.31 (configurable)
- **Addons**: EBS CSI Driver, CoreDNS, Kube-Proxy, VPC CNI
- **Capacity**: 2 nodes (configurable)
- **Networking**: VPC with public/private subnets, OIDC provider enabled
- **Security**: Private nodes, OIDC-based authentication

#### 2. **Dex OIDC Provider**
- **Purpose**: OpenID Connect identity provider for platform authentication
- **Features**: Multi-tenant OAuth2/OIDC, static clients, dynamic client management
- **Integration**: Azure AD as identity source
- **Clients**: Central Ops, SaaS Tenants, Engineering, JupyterHub, Airflow
- **Domains**: auth.int.cequence.ai (prod), auth-dev.int.cequence.ai (dev)

#### 3. **Azure AD SSO Integration**
- **Purpose**: Enterprise single sign-on with Azure Active Directory
- **Configuration**: Multi-tenant Azure AD application
- **Features**: SAML2 tokens, group membership claims, OAuth2 permissions
- **Tenant Support**: Dynamic tenant discovery and configuration
- **Security**: Role-based access control, admin consent flows

#### 4. **Multi-tenant OAuth2 Platform**
- **Tenant Discovery**: Dynamic discovery of customer tenants
- **Client Management**: Static and dynamic OAuth2 client registration
- **Redirect URIs**: Tenant-specific callback URLs
- **JupyterHub Integration**: OAuth2 integration for data science platforms
- **Airflow Integration**: OAuth2 integration for workflow orchestration

#### 5. **Certificate and Ingress Management**
- **Cert-Manager**: Automated SSL/TLS certificate provisioning
- **Ingress Controller**: NGINX ingress for external access
- **Load Balancer**: AWS Application Load Balancer
- **SSL/TLS**: Let's Encrypt certificates for all domains
- **Security**: Force SSL redirect, proper TLS configuration

### Environment Configuration Analysis

#### Development Environment (dev)
- **Domain**: auth-dev.int.cequence.ai
- **Issuer**: https://issuer.auth-dev.int.cequence.ai
- **Cluster Version**: 1.31
- **Azure AD**: Development tenant configuration
- **Static Clients**: 2 clients (central-ops, saas-airflow)
- **Tenant Support**: Limited tenant exclusions

#### Production Environment (prod)
- **Domain**: auth.int.cequence.ai
- **Issuer**: https://issuer.auth.int.cequence.ai
- **Cluster Version**: 1.31
- **Azure AD**: Production tenant configuration
- **Static Clients**: 4 clients (engineering-dev, saas-tenants-prod, saas-tenants-dev, central-ops)
- **Tenant Support**: Extensive tenant exclusions (22+ tenants)

### Integration Dependencies
This repository has **extensive dependencies** on external services:

1. **Azure Active Directory**: Enterprise identity provider
2. **Customer Tenants**: 100+ customer tenant configurations
3. **Platform Services**: Central Ops, SaaS Tenants, Engineering teams
4. **Data Platforms**: JupyterHub, Airflow integrations
5. **Certificate Authority**: Let's Encrypt for SSL certificates

## Migration Impact Assessment

### Direct Migration Impact: CRITICAL
- **Pulumi Cloud dependency**: Repository uses Pulumi Cloud backend for state management
- **State migration required**: All infrastructure state must be migrated to new backend
- **Azure AD integration**: Complex Azure AD application configuration must be preserved
- **Multi-tenant configuration**: 100+ customer tenant configurations must be maintained

### Infrastructure Components Requiring Migration
1. **EKS Cluster**: Complete Kubernetes cluster with all addons
2. **Dex Configuration**: OIDC provider configuration and client management
3. **Azure AD Application**: Enterprise SSO application configuration
4. **Certificate Management**: SSL/TLS certificate provisioning
5. **Ingress Configuration**: Load balancer and routing configuration
6. **Multi-tenant Setup**: Customer tenant discovery and configuration

### Indirect Migration Impact: CRITICAL
- **Authentication Platform**: All platform authentication depends on this
- **Customer Access**: Customer tenant authentication may be affected
- **Service Dependencies**: All other services depend on this for authentication
- **Configuration Updates Required**: All dependent services must be updated

## Migration Strategy and Recommendations

### Pre-Migration Preparation
1. **State Backup**: Export current Pulumi state for rollback capability
2. **Azure AD Audit**: Document all Azure AD application configurations
3. **Tenant Configuration Backup**: Export all customer tenant configurations
4. **Certificate Backup**: Backup all SSL certificates and configurations
5. **Dependency Mapping**: Map all services that depend on this authentication platform

### Migration Execution Steps
1. **Azure AD Preparation**: Ensure Azure AD application configurations are documented
2. **State Migration**: Migrate Pulumi state to new backend
3. **Configuration Migration**: Transfer all encrypted configuration values
4. **Infrastructure Validation**: Verify EKS cluster and Dex are operational
5. **Authentication Testing**: Test OIDC/OAuth2 flows for all clients
6. **Tenant Validation**: Verify all customer tenant configurations

### Post-Migration Validation
1. **Cluster Health**: Verify EKS cluster and all addons are operational
2. **Dex Functionality**: Test OIDC provider and client management
3. **Azure AD Integration**: Test enterprise SSO functionality
4. **Certificate Management**: Verify SSL certificate provisioning
5. **Multi-tenant Access**: Test customer tenant authentication
6. **Service Integration**: Test authentication for all dependent services

## Risk Assessment and Mitigation

### High-Risk Areas
1. **Authentication Platform**: Core authentication services for entire platform
   - **Mitigation**: Maintain parallel authentication during migration
   - **Rollback**: Prepare rapid restoration of authentication services

2. **Azure AD Integration**: Complex enterprise SSO configuration
   - **Mitigation**: Document all Azure AD configurations before migration
   - **Rollback**: Restore Azure AD application configurations

3. **Customer Tenant Access**: 100+ customer tenant configurations
   - **Mitigation**: Backup all tenant configurations before migration
   - **Rollback**: Restore customer tenant access configurations

4. **Certificate Management**: SSL/TLS certificate provisioning
   - **Mitigation**: Backup all certificate configurations
   - **Rollback**: Restore certificate management and SSL access

### Medium-Risk Areas
1. **EKS Cluster Migration**: Complete Kubernetes cluster infrastructure
   - **Mitigation**: Test cluster migration in development environment
   - **Rollback**: Restore EKS cluster from backup

2. **Multi-tenant Configuration**: Complex tenant discovery and management
   - **Mitigation**: Validate tenant configurations before migration
   - **Rollback**: Restore tenant discovery and configuration

3. **Service Dependencies**: All platform services depend on authentication
   - **Mitigation**: Coordinate with all dependent service teams
   - **Rollback**: Restore authentication for all dependent services

## Timeline Integration with Overall Migration

### Phase Alignment
- **Pre-Migration (Week 1)**: Azure AD and tenant configuration documentation
- **Migration Execution (Week 2)**: Dex cluster state and configuration migration
- **Post-Migration (Week 3)**: Authentication validation and service integration
- **Ongoing**: Monitor authentication platform and customer access

### Dependencies
- **Prerequisites**: Must be migrated before all other services
- **Coordination**: Align with all platform service teams
- **Validation**: Include customer success team in authentication testing

## Key Insights for Overall Migration Strategy

### Authentication Platform Complexity
The dex-master repository reveals **comprehensive authentication platform** requirements:
- **Enterprise SSO**: Azure AD integration for enterprise customers
- **Multi-tenant OAuth2**: Support for 100+ customer tenants
- **Platform Integration**: Authentication for all platform services
- **Data Platform Support**: JupyterHub and Airflow integrations

### Customer Impact Considerations
- **Customer Authentication**: All customer access depends on this platform
- **Tenant Management**: Complex multi-tenant configuration management
- **Enterprise SSO**: Azure AD integration for enterprise customers
- **Service Dependencies**: All platform services require authentication

### Resource Requirements
- **Dedicated Cluster**: EKS cluster dedicated to authentication services
- **Azure AD Resources**: Enterprise Azure AD application and permissions
- **Certificate Resources**: SSL/TLS certificate management for all domains
- **Network Resources**: Load balancer, ingress, and DNS configuration

## Strategic Recommendations

### 1. Treat as Critical Foundation Infrastructure
- dex-master provides **essential authentication services** for entire platform
- **Highest priority** for migration due to service dependencies
- Ensure **zero authentication downtime** during migration

### 2. Coordinate with All Platform Teams
- **Include all service teams** in migration planning
- **Schedule migration windows** during low-usage periods
- **Prepare rollback procedures** for authentication services

### 3. Enhanced Testing and Validation
- **Comprehensive authentication testing** post-migration
- **Multi-tenant validation** for all customer tenants
- **Enterprise SSO testing** for Azure AD integration

### 4. Long-term Operational Considerations
- **Document authentication dependencies** across platform
- **Establish monitoring** for authentication platform health
- **Create procedures** for coordinated authentication changes

## Conclusion

The dex-master repository represents **critical authentication infrastructure** that provides OIDC/OAuth2 services, Azure AD integration, and multi-tenant authentication for the entire platform. This repository requires **direct Pulumi migration** and significantly impacts **all platform services** by providing:

- **Centralized authentication platform** with enterprise SSO capabilities
- **Multi-tenant OAuth2/OIDC** support for 100+ customer tenants
- **Azure AD integration** for enterprise customer authentication
- **Platform service authentication** for all dependent services

This discovery reinforces that the Pulumi migration affects not just individual services but also **foundational authentication infrastructure** that is essential for all platform operations. The migration strategy must account for both the technical infrastructure migration and the **authentication service continuity** requirements of the entire platform.

The dex-master repository is the **authentication foundation** that enables secure access to all platform services and customer tenant resources, making it the most critical component in the migration project that directly impacts platform security and customer access.
