# Migration Analysis: central-ops-ossec 

## Executive Summary

**DISCOVERY**: The central-ops-ossec repository is a **Pulumi infrastructure project** that deploys Wazuh (formerly OSSEC) security monitoring platform to the central operations Kubernetes cluster. This repository contains TypeScript infrastructure code for Wazuh deployment, CloudWatch integration, and IAM role management.

**Migration Impact**: **DIRECT PULUMI MIGRATION REQUIRED** - This is a Pulumi project that uses Pulumi Cloud backend and requires state migration.

**Relationship to Pulumi Migration**: This repository represents **security monitoring infrastructure** that provides intrusion detection, log analysis, and security monitoring capabilities for the platform.

## Repository Overview

**Repository**: central-ops-ossec  
**Purpose**: Wazuh security monitoring platform deployment and CloudWatch integration  
**Technology**: Pulumi + TypeScript + Kubernetes + Wazuh + CloudWatch + IAM  
**Components**: Wazuh Stack + CloudWatch Integration + IAM Roles + Kustomize Deployments  
**Architecture**: Centralized security monitoring with multi-environment support

### Repository Structure
```
central-ops-ossec/
├── src/
│   ├── index.ts              # Main Pulumi program entry point
│   ├── config.ts             # Configuration and environment setup
│   ├── ossec.ts              # Wazuh deployment and IAM role management
│   └── wazuh-kubernetes/     # Kustomize-based Wazuh deployment manifests
│       └── eks/
│           ├── base/         # Base Wazuh configuration
│           ├── base-cq/      # Cequence-specific customizations
│           └── overlays/     # Environment-specific overlays (dev/prod)
├── Pulumi.yaml               # Pulumi project configuration
├── Pulumi.dev.yaml           # Development environment configuration
├── Pulumi.prod.yaml          # Production environment configuration
├── package.json              # Node.js dependencies
└── tsconfig.json             # TypeScript configuration
```

## Technical Architecture Analysis

### Pulumi Infrastructure Pattern
This repository implements a **centralized security monitoring pattern**:

1. **Wazuh Deployment**: Security monitoring platform using Kustomize
2. **CloudWatch Integration**: Log and metrics forwarding to AWS CloudWatch
3. **IAM Role Management**: Service account roles for CloudWatch access
4. **Environment Separation**: Different configurations for dev/prod environments

### Key Components

#### 1. **Wazuh Security Platform**
- **Purpose**: Intrusion detection, log analysis, and security monitoring
- **Deployment**: Kustomize-based deployment with environment-specific overlays
- **Components**: Wazuh Manager, Workers, Elasticsearch, Kibana
- **Features**: Real-time threat detection, log analysis, compliance monitoring
- **Storage**: Persistent volumes for Wazuh data and Elasticsearch indices

#### 2. **CloudWatch Integration**
- **Purpose**: Forward security logs and metrics to AWS CloudWatch
- **Agent**: CloudWatch agent deployed in Wazuh namespace
- **Logs**: Security events, alerts, and audit logs
- **Metrics**: Performance and security metrics
- **IAM Roles**: Dedicated roles for CloudWatch access

#### 3. **IAM Role Management**
- **Wazuh CloudWatch Role**: For Wazuh manager worker service account
- **Prometheus CloudWatch Role**: For CloudWatch agent service account
- **Permissions**: CloudWatch logs and metrics publishing
- **OIDC Integration**: Uses EKS OIDC provider for role assumption

#### 4. **Kustomize Deployment Structure**
- **Base Configuration**: Common Wazuh deployment manifests
- **Cequence Customizations**: Company-specific configurations and patches
- **Environment Overlays**: Dev/prod specific configurations
- **Ingress Patches**: Environment-specific domain configurations

### Environment Configuration Analysis

#### Development Environment (dev)
- **Domain**: wids-dev.central-ops-dev.int.cequence.ai
- **Namespace**: wazuh
- **CloudWatch Integration**: Enabled with dev-specific configuration
- **Resource Allocation**: Standard resource allocation for development

#### Production Environment (prod)
- **Domain**: wids.central-ops.int.cequence.ai
- **Namespace**: wazuh
- **CloudWatch Integration**: Enabled with production configuration
- **Resource Allocation**: Production-optimized resource allocation

### Dependencies on Other Pulumi Infrastructure
This repository has **critical dependencies** on the central-ops cluster:

1. **Central Operations Cluster**: Referenced via `cequence/central-ops/${env}` StackReference
2. **OIDC Provider**: Uses EKS OIDC provider for IAM role assumption
3. **Kubernetes Configuration**: Uses cluster kubeconfig from central-ops
4. **VPC Infrastructure**: Deploys within central-ops cluster network

## Migration Impact Assessment

### Direct Migration Impact: HIGH
- **Pulumi Cloud dependency**: Repository uses Pulumi Cloud backend for state management
- **State migration required**: All infrastructure state must be migrated to new backend
- **Stack references**: Dependencies on central-ops cluster must be updated
- **IAM role migration**: IAM roles and policies must be migrated

### Infrastructure Components Requiring Migration
1. **IAM Roles**: CloudWatch integration roles and policies
2. **Kubernetes Resources**: Wazuh deployment via Kustomize
3. **CloudWatch Configuration**: Log groups and metric streams
4. **Service Accounts**: Annotated service accounts for IAM role assumption
5. **Ingress Resources**: Environment-specific domain configurations

### Indirect Migration Impact: MEDIUM
- **Central Operations Dependency**: Requires central-ops cluster to be migrated first
- **Security Monitoring Risk**: Security monitoring capabilities may be affected
- **Configuration Updates Required**: Stack references and OIDC provider references must be updated
- **Timing Coordination**: Must be migrated after central-ops cluster migration

## Migration Strategy and Recommendations

### Pre-Migration Preparation
1. **State Backup**: Export current Pulumi state for rollback capability
2. **Configuration Audit**: Document all IAM roles and CloudWatch configurations
3. **Dependency Validation**: Ensure central-ops cluster migration is complete
4. **Security Data Backup**: Backup Wazuh data and configurations
5. **Integration Testing**: Verify CloudWatch integration is working

### Migration Execution Steps
1. **Update Stack References**: Modify central-ops cluster references to new backend
2. **State Migration**: Migrate Pulumi state to new backend
3. **Configuration Migration**: Transfer IAM role configurations
4. **Infrastructure Validation**: Verify all components are operational
5. **Security Monitoring Testing**: Test Wazuh functionality and CloudWatch integration

### Post-Migration Validation
1. **Wazuh Health**: Verify Wazuh manager and workers are healthy
2. **CloudWatch Integration**: Test log and metric forwarding to CloudWatch
3. **Security Monitoring**: Verify threat detection and log analysis functionality
4. **Ingress Access**: Test web interface access via ingress
5. **IAM Role Functionality**: Verify service account role assumption

## Risk Assessment and Mitigation

### High-Risk Areas
1. **Security Data Loss**: Wazuh data and security event history
   - **Mitigation**: Backup Wazuh data before migration
   - **Rollback**: Restore from backup if migration fails

2. **IAM Role Dependencies**: Complex IAM role configurations
   - **Mitigation**: Document all IAM roles and policies before migration
   - **Rollback**: Restore original IAM role configurations

3. **CloudWatch Integration**: Log and metric forwarding
   - **Mitigation**: Test CloudWatch integration before migration
   - **Rollback**: Restore original CloudWatch configuration

### Medium-Risk Areas
1. **Kustomize Deployment**: Complex Kustomize-based deployment
   - **Mitigation**: Test Kustomize deployment before migration
   - **Rollback**: Restore original Kustomize configuration

2. **OIDC Provider Dependencies**: EKS OIDC provider integration
   - **Mitigation**: Ensure OIDC provider is available after migration
   - **Rollback**: Restore original OIDC provider references

3. **Ingress Configuration**: Environment-specific domain configurations
   - **Mitigation**: Document current ingress configuration
   - **Rollback**: Restore original ingress settings

## Timeline Integration with Overall Migration

### Phase Alignment
- **Pre-Migration (Week 1)**: Central-ops cluster migration completion
- **Migration Execution (Week 2)**: central-ops-ossec state and configuration migration
- **Post-Migration (Week 3)**: Security monitoring validation and testing
- **Ongoing**: Monitor security monitoring operations and CloudWatch integration

### Dependencies
- **Prerequisite**: central-ops cluster migration must be completed first
- **Coordination**: Align with security team for monitoring service windows
- **Validation**: Include security team in post-migration testing

## Key Insights for Overall Migration Strategy

### Security Monitoring Platform Complexity
The central-ops-ossec repository reveals **critical security monitoring** requirements:
- **Wazuh Platform**: Comprehensive security monitoring and intrusion detection
- **CloudWatch Integration**: Centralized logging and metrics collection
- **IAM Role Management**: Complex service account role configurations
- **Multi-Environment Support**: Different configurations for dev/prod

### Security Impact Considerations
- **Threat Detection Dependencies**: Security monitoring capabilities must remain available
- **Log Analysis Continuity**: Security event analysis must continue uninterrupted
- **Compliance Requirements**: Security monitoring for compliance and auditing
- **Incident Response**: Security data for incident investigation and response

### Resource Requirements
- **Storage Resources**: Persistent volumes for Wazuh data and Elasticsearch
- **Compute Resources**: Wazuh manager and worker nodes
- **Network Resources**: Ingress rules and SSL certificates
- **CloudWatch Resources**: Log groups and metric streams

## Strategic Recommendations

### 1. Treat as Critical Security Infrastructure
- central-ops-ossec provides **essential security monitoring capabilities**
- **High priority** for migration due to security dependencies
- Ensure **minimal security monitoring disruption** during migration

### 2. Coordinate with Security Team
- **Include security team** in migration planning
- **Schedule migration windows** during low-security-risk periods
- **Prepare rollback procedures** for security monitoring operations

### 3. Enhanced Testing and Validation
- **Comprehensive security monitoring testing** post-migration
- **CloudWatch integration validation** for log and metric forwarding
- **Threat detection testing** for security monitoring functionality

### 4. Long-term Operational Considerations
- **Document security monitoring dependencies** on this infrastructure
- **Establish monitoring** for security monitoring operations
- **Create procedures** for coordinated security infrastructure changes

## Conclusion

The central-ops-ossec repository represents **critical security infrastructure** that provides intrusion detection, log analysis, and security monitoring capabilities for the platform. This repository requires **direct Pulumi migration** and significantly impacts **platform security** by providing:

- **Wazuh security platform** for comprehensive threat detection
- **CloudWatch integration** for centralized security logging
- **IAM role management** for secure CloudWatch access
- **Multi-environment support** with dev/prod configurations

This discovery reinforces that the Pulumi migration affects not just application infrastructure but also **security monitoring capabilities** that are essential for platform security. The migration strategy must account for both the technical infrastructure migration and the **security continuity requirements** of monitoring systems.

The central-ops-ossec repository is the **security monitoring layer** that enables threat detection and security analysis for the platform, making it a critical component in the migration project that directly impacts platform security posture.

