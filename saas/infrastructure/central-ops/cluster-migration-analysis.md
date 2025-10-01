# Migration Analysis: cluster

## Executive Summary

**DISCOVERY**: The central-ops repository is a **Pulumi infrastructure project** that deploys the central operations Kubernetes cluster on AWS EKS. This repository contains TypeScript infrastructure code for EKS cluster, VPC, networking, monitoring, and platform services.

**Migration Impact**: **DIRECT PULUMI MIGRATION REQUIRED** - This is a Pulumi project that uses Pulumi Cloud backend and requires state migration.

**Relationship to Pulumi Migration**: This repository represents **core infrastructure** that hosts all central operations services and is the foundation for all other platform components.

## Repository Overview

**Repository**: central-ops  
**Purpose**: Central operations Kubernetes cluster infrastructure on AWS EKS  
**Technology**: Pulumi + TypeScript + AWS EKS + Karpenter + Monitoring  
**Components**: EKS Cluster + VPC + Karpenter + Ingress + Cert Manager + Monitoring  
**Architecture**: Centralized operations platform with auto-scaling and monitoring

### Repository Structure
```
central-ops/
├── src/
│   ├── index.ts              # Main Pulumi program entry point
│   └── config.ts             # Configuration and environment setup
├── scripts/
│   ├── delete-node-group.sh  # Node group management scripts
│   ├── drain-node-group.sh   # Node draining utilities
│   └── get-recommended-ami.sh # AMI selection helper
├── Pulumi.yaml               # Pulumi project configuration
├── Pulumi.dev.yaml           # Development environment configuration
├── Pulumi.prod.yaml          # Production environment configuration
├── package.json              # Node.js dependencies
└── tsconfig.json             # TypeScript configuration
```

## Technical Architecture Analysis

### Pulumi Infrastructure Pattern
This repository implements a **centralized operations cluster pattern**:

1. **EKS Cluster**: Core Kubernetes cluster with managed node groups
2. **Karpenter Integration**: Advanced auto-scaling and node provisioning
3. **Platform Services**: Ingress, cert-manager, metrics-server, monitoring
4. **Environment Separation**: Separate dev/prod clusters with different configurations

### Key Components

#### 1. **EKS Cluster Infrastructure**
- **Purpose**: Central Kubernetes cluster for all operations services
- **Version**: Kubernetes 1.31
- **Node Groups**: Managed node groups with ARM64 and x86_64 support
- **Addons**: EBS CSI driver, CoreDNS, kube-proxy, VPC CNI
- **Logging**: VPC flow logs and cluster logging enabled

#### 2. **Karpenter Auto-Scaling**
- **Purpose**: Advanced node provisioning and auto-scaling
- **Version**: v0.27.0 with drift detection enabled
- **Provisioners**: Multiple provisioners for different workload types
- **Resource Limits**: CPU and memory limits per provisioner
- **Consolidation**: Spot instance consolidation for cost optimization

#### 3. **Platform Services**
- **Ingress Nginx**: Load balancer and ingress controller
- **Cert Manager**: SSL certificate management with Let's Encrypt
- **Metrics Server**: Kubernetes metrics collection
- **Monitoring**: Datadog and Grafana Cloud integration

#### 4. **VPC and Networking**
- **VPC**: Custom VPC with public and private subnets
- **NAT Gateways**: High availability NAT gateways
- **Security Groups**: Cluster and node security groups
- **Flow Logs**: VPC flow logging for network monitoring

### Environment Configuration Analysis

#### Development Environment (dev)
- **Cluster Name**: central-ops-cluster-eksCluster-18ce112
- **Karpenter Replicas**: 1 replica
- **Datadog**: Disabled
- **Grafana Cloud**: Enabled with pod logs collection
- **Log Collection**: service-portal, documentdb-proxy, grafana-k8s-monitoring

#### Production Environment (prod)
- **Cluster Name**: central-ops-cluster-eksCluster-1af4671
- **Karpenter Replicas**: 1 replica
- **Datadog**: Disabled
- **Grafana Cloud**: Enabled with pod logs collection
- **Log Collection**: service-portal, documentdb-proxy, grafana-k8s-monitoring

### Karpenter Provisioner Configuration
The cluster uses **advanced Karpenter provisioners** for different workload types:

1. **Spot ARM64**: Cost-optimized ARM64 instances with consolidation
2. **Spot Primary AZ ARM64**: Primary AZ ARM64 spot instances
3. **On-Demand Primary AZ ARM64**: Primary AZ ARM64 on-demand instances
4. **Spot Primary AZ**: Primary AZ x86_64 spot instances
5. **Spot**: General x86_64 spot instances with consolidation

### Dependencies on External Services
This repository has **critical dependencies** on external services:

1. **Grafana Cloud**: Prometheus, Loki, and Tempo for monitoring
2. **Datadog**: Optional monitoring and observability
3. **AWS Services**: EKS, VPC, IAM, EC2, and other AWS services
4. **Container Registry**: GitLab container registry for images

## Migration Impact Assessment

### Direct Migration Impact: CRITICAL
- **Pulumi Cloud dependency**: Repository uses Pulumi Cloud backend for state management
- **State migration required**: All infrastructure state must be migrated to new backend
- **Cluster recreation risk**: EKS cluster may need to be recreated during migration
- **Configuration migration**: All encrypted configuration values must be migrated

### Infrastructure Components Requiring Migration
1. **EKS Cluster**: Complete Kubernetes cluster infrastructure
2. **VPC and Networking**: VPC, subnets, security groups, NAT gateways
3. **Karpenter Configuration**: Auto-scaling and node provisioning setup
4. **Platform Services**: Ingress, cert-manager, metrics-server
5. **Monitoring Integration**: Grafana Cloud and Datadog configurations
6. **IAM Roles and Policies**: Service roles and permissions

### Indirect Migration Impact: CRITICAL
- **Platform Foundation**: All other services depend on this cluster
- **Service Disruption Risk**: All platform services may be affected
- **Configuration Updates Required**: All dependent services must be updated
- **Timing Coordination**: Must be migrated before all dependent services

## Migration Strategy and Recommendations

### Pre-Migration Preparation
1. **State Backup**: Export current Pulumi state for rollback capability
2. **Configuration Audit**: Document all encrypted configuration values
3. **Cluster Documentation**: Document current cluster state and configurations
4. **Dependency Mapping**: Map all services that depend on this cluster
5. **Data Backup**: Backup any persistent data in the cluster

### Migration Execution Steps
1. **State Migration**: Migrate Pulumi state to new backend
2. **Configuration Migration**: Transfer all encrypted configuration values
3. **Infrastructure Validation**: Verify all AWS resources are properly configured
4. **Cluster Health Check**: Ensure EKS cluster is healthy and operational
5. **Service Validation**: Verify all platform services are working

### Post-Migration Validation
1. **Cluster Health**: Verify EKS cluster and all nodes are healthy
2. **Platform Services**: Test ingress, cert-manager, and metrics-server
3. **Karpenter Functionality**: Test auto-scaling and node provisioning
4. **Monitoring Integration**: Verify Grafana Cloud and Datadog connectivity
5. **Network Connectivity**: Test VPC and networking configurations

## Risk Assessment and Mitigation

### Critical-Risk Areas
1. **EKS Cluster Recreation**: Risk of cluster recreation during migration
   - **Mitigation**: Plan for cluster recreation with data migration
   - **Rollback**: Restore from backup or recreate cluster

2. **Platform Service Disruption**: All services depend on this cluster
   - **Mitigation**: Coordinate migration with all dependent services
   - **Rollback**: Restore cluster and re-deploy all services

3. **Data Loss Risk**: Persistent data in the cluster
   - **Mitigation**: Backup all persistent volumes and data
   - **Rollback**: Restore data from backups

### High-Risk Areas
1. **Karpenter Configuration**: Complex auto-scaling setup
   - **Mitigation**: Document current Karpenter configuration
   - **Rollback**: Restore original Karpenter configuration

2. **Monitoring Integration**: Grafana Cloud and Datadog connectivity
   - **Mitigation**: Test monitoring connectivity before migration
   - **Rollback**: Restore original monitoring configuration

3. **Network Configuration**: VPC and security group setup
   - **Mitigation**: Document current network configuration
   - **Rollback**: Restore original network configuration

### Medium-Risk Areas
1. **SSL Certificate Management**: Cert-manager configuration
   - **Mitigation**: Verify certificate management before migration
   - **Rollback**: Restore original cert-manager configuration

2. **Load Balancer Configuration**: Ingress and load balancer setup
   - **Mitigation**: Document current load balancer configuration
   - **Rollback**: Restore original load balancer configuration

## Timeline Integration with Overall Migration

### Phase Alignment
- **Pre-Migration (Week 1)**: Cluster documentation and backup
- **Migration Execution (Week 2)**: central-ops cluster migration
- **Post-Migration (Week 3)**: Platform service validation and testing
- **Ongoing**: Monitor cluster health and platform services

### Dependencies
- **Prerequisites**: No dependencies - this is the foundation cluster
- **Coordination**: All other services depend on this cluster
- **Validation**: Include platform operations team in post-migration testing

## Key Insights for Overall Migration Strategy

### Central Operations Cluster Complexity
The central-ops repository reveals **critical infrastructure complexity**:
- **EKS Cluster Management**: Complex Kubernetes cluster with advanced features
- **Karpenter Integration**: Sophisticated auto-scaling and node provisioning
- **Platform Services**: Multiple integrated services for operations
- **Monitoring Integration**: Comprehensive observability setup

### Platform-Wide Impact Considerations
- **Foundation Infrastructure**: All platform services depend on this cluster
- **Service Orchestration**: Central cluster for all operations services
- **Resource Management**: Karpenter manages all compute resources
- **Monitoring Hub**: Central point for all platform monitoring

### Resource Requirements
- **High-Performance Requirements**: Production cluster with significant resources
- **Auto-Scaling Configuration**: Karpenter with multiple provisioners
- **Network Resources**: VPC, subnets, NAT gateways, security groups
- **Storage Resources**: EBS CSI driver and persistent volumes

## Strategic Recommendations

### 1. Treat as Critical Foundation Infrastructure
- central-ops is **essential for all platform operations**
- **Highest priority** for migration due to platform-wide dependencies
- Ensure **minimal service disruption** during migration

### 2. Coordinate with All Platform Services
- **Include all platform teams** in migration planning
- **Schedule migration windows** during low-activity periods
- **Prepare rollback procedures** for all dependent services

### 3. Enhanced Testing and Validation
- **Comprehensive cluster testing** post-migration
- **Platform service validation** for all dependent services
- **Karpenter testing** for auto-scaling functionality

### 4. Long-term Operational Considerations
- **Document cluster dependencies** for all platform services
- **Establish monitoring** for cluster health and platform services
- **Create procedures** for coordinated infrastructure changes

## Conclusion

The central-ops repository represents **critical foundation infrastructure** that hosts all central operations services and provides the foundation for the entire platform. This repository requires **direct Pulumi migration** and has **platform-wide impact** by providing:

- **Central Kubernetes cluster** for all operations services
- **Advanced auto-scaling** with Karpenter for resource management
- **Platform services** including ingress, cert-manager, and monitoring
- **Monitoring integration** with Grafana Cloud and Datadog

This discovery reinforces that the Pulumi migration affects not just individual services but the **entire platform's foundation**. The migration strategy must account for both the technical infrastructure migration and the **platform-wide operational continuity** that depends on this central cluster.

The central-ops repository is the **foundation layer** that makes all other platform services possible, making it the most critical component in the migration project that affects every aspect of platform operations.

