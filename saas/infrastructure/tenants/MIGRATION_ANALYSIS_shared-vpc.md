# Pulumi Migration Analysis: shared-vpc (GCP Foundation Network)

## Executive Summary

**DISCOVERY**: The shared-vpc repository manages **2 GCP shared VPC environments** (dev/prod) that serve as the **foundational network infrastructure** for all GCP-based SaaS services. This is a **critical dependency** for the clusters repository and represents foundational infrastructure that must be migrated first.

**Migration Complexity: HIGH (Level 4)**
- 2 shared VPC environments (dev/prod) serving as network foundation
- Global multi-region subnet allocation (40+ GCP regions)
- Cross-project IAM permissions and service project attachments
- Critical dependency for all GCP GKE clusters and services

## Repository Overview

**Repository**: shared-vpc  
**Purpose**: Provision and manage shared VPC networks for multi-tenant GCP infrastructure  
**Stack Count**: 2 environments (dev, prod)  
**Primary Language**: TypeScript  
**Pulumi Project**: saas-shared-vpc  
**Cloud Platform**: Google Cloud Platform (GCP)

### Current Pulumi Cloud Configuration
- **Organization**: `cequence`  
- **Project**: `saas-shared-vpc`  
- **Stack Pattern**: `cequence/saas-shared-vpc/{env}`
- **Backend URL**: Pulumi Cloud (app.pulumi.com)

### Stack Environments
1. **dev** - Development shared VPC (host project: saas-vpc-host-dev-7ce69e6)
2. **prod** - Production shared VPC (host project: saas-vpc-host-prod-e7644fd)

## Technical Architecture Analysis

### Shared VPC Infrastructure
1. **Network Foundation**:
   - **Global VPC Networks**: Single VPC per environment spanning all regions
   - **Regional Subnets**: Automatically provisioned subnets in 40+ GCP regions
   - **NAT Gateways**: 2 NAT IPs per regional subnet for outbound connectivity
   - **Load Balancer Subnets**: Dedicated subnets for GCP load balancers

2. **Multi-Region Coverage**:
   ```yaml
   Regions: [
     africa-south1, asia-east1, asia-east2, asia-northeast1,
     asia-northeast2, asia-northeast3, asia-south1, asia-south2,
     asia-southeast1, asia-southeast2, australia-southeast1,
     australia-southeast2, europe-central2, europe-north1,
     europe-southwest1, europe-west1, europe-west10, europe-west12,
     europe-west2, europe-west3, europe-west4, europe-west6,
     europe-west8, europe-west9, me-central1, me-west1,
     northamerica-northeast1, northamerica-northeast2,
     southamerica-east1, southamerica-west1, us-central1,
     us-east1, us-east4, us-east5, us-south1, us-west1,
     us-west2, us-west3, us-west4, northamerica-south1, me-central2
   ]
   ```

### Service Project Integration
1. **Host-Service Model**:
   - **VPC Host Projects**: saas-vpc-host-{env}-{id}
   - **Service Projects**: saas-tenants-svc-{env}-{id}
   - **Cross-Project Permissions**: IAM bindings for GKE clusters

2. **IAM Permission Structure**:
   - **Google API Service Account**: Cross-project compute access
   - **Kubernetes Service Account**: GKE cluster management permissions
   - **Subnet-Level Permissions**: Regional subnet access for GKE nodes

### Critical Dependencies
This repository serves as the **network foundation** for:
- **clusters repository**: All 5 GKE clusters depend on these shared VPCs
- **Future GCP services**: Any additional GCP-based infrastructure
- **Cross-project connectivity**: Service-to-service communication

## Migration Risk Assessment

### Risk Level: HIGH (4/5)

**Critical Risk Factors**:
1. **Foundation Infrastructure**: Core dependency for all GCP services
2. **Multi-Region Complexity**: 40+ regional subnets across global regions
3. **Cross-Project Dependencies**: Complex IAM permissions across multiple GCP projects
4. **Service Disruption Impact**: VPC migration could affect all dependent GKE clusters
5. **Network State Sensitivity**: IP address allocation and routing table integrity

**Specific Technical Risks**:
- **Subnet CIDR Blocks**: Regional IP address range allocation must be preserved
- **NAT Gateway IPs**: Outbound connectivity disruption for dependent services
- **IAM Bindings**: Cross-project permissions could break during migration
- **Route Tables**: Network routing configuration integrity
- **Load Balancer Dependencies**: GCP load balancer subnet assignments

### Dependency Impact Analysis
**Downstream Dependencies**:
- **clusters repository**: 5 GKE clusters rely on shared VPC subnets
- **Future services**: Any new GCP infrastructure will depend on this foundation
- **Cross-cloud connectivity**: Potential VPN/interconnect dependencies

**Migration Order Requirement**: 
This repository **MUST be migrated first** before any dependent GCP infrastructure.

## Migration Strategy Recommendations

### Phase 1: Pre-Migration Foundation (Week 1)
1. **Dependency Mapping**:
   - Document all service projects using shared VPCs
   - Map subnet allocations to specific GKE clusters
   - Inventory all cross-project IAM bindings
   - Identify any VPN/interconnect connections

2. **State Documentation**:
   - Export complete network topology
   - Document CIDR block allocations for all regions
   - Backup all IAM policy configurations
   - Map NAT gateway IP assignments

3. **Risk Assessment**:
   - Identify critical business hours for each region
   - Plan minimal-impact migration windows
   - Prepare rollback procedures for network changes

### Phase 2: Development Environment Migration (Week 2)
1. **Dev Environment First**:
   - Migrate **dev** stack first (lower risk)
   - Validate all subnets preserve CIDR blocks
   - Test cross-project IAM permissions
   - Verify NAT gateway functionality

2. **Validation Procedures**:
   - Test GKE cluster connectivity from dependent clusters
   - Validate cross-region network routing
   - Confirm load balancer subnet assignments
   - Test outbound NAT connectivity

### Phase 3: Production Environment Migration (Week 3)
1. **Production Migration**:
   - Schedule during global low-traffic window
   - Migrate **prod** stack with enhanced monitoring
   - Validate all 40+ regional subnets operational
   - Test all service project connectivity

2. **Post-Migration Validation**:
   - Full network connectivity testing
   - Cross-project IAM validation
   - Performance baseline comparison
   - 48-hour monitoring period

### Phase 4: Integration Testing (Week 4)
1. **Dependent Service Testing**:
   - Coordinate with clusters repository team
   - Test all GKE cluster network functionality
   - Validate any cross-cloud connectivity
   - Confirm monitoring and logging systems

## Timeline Estimates

### Migration Timeline
- **Foundation Phase**: 1 week (dependency mapping, state documentation)
- **Development Migration**: 1 week (dev environment with validation)
- **Production Migration**: 1 week (prod environment with monitoring)
- **Integration Testing**: 1 week (dependent service validation)

**Total Timeline**: 4 weeks (1 month)

### Critical Path
1. **Week 1**: Complete state documentation and dependency analysis
2. **Week 2**: Migrate dev, validate network functionality
3. **Week 3**: Migrate prod during planned maintenance window
4. **Week 4**: Full integration testing with dependent clusters

## Resource Requirements

### Specialized Team Requirements
- **GCP Network Engineers**: 2-3 engineers with VPC and IAM expertise
- **Infrastructure Architects**: 1-2 engineers for dependency coordination
- **Security Engineers**: 1-2 specialists for IAM permission validation
- **Site Reliability Engineers**: 2-3 engineers for monitoring and validation

### Infrastructure Requirements
- **Self-Hosted Backend**: Global deployment for multi-region performance
- **Network Monitoring**: Enhanced observability for VPC and subnet health
- **Migration Tooling**: GCP-specific state validation and rollback tools
- **Testing Infrastructure**: Isolated validation environment

## Success Criteria

### Technical Success Metrics
- ✅ All regional subnets preserve CIDR block allocations
- ✅ Cross-project IAM permissions functional across all service projects
- ✅ NAT gateway connectivity maintained for all regions
- ✅ Load balancer subnets operational in all regions
- ✅ Network performance baselines maintained or improved

### Integration Success Metrics
- ✅ All dependent GKE clusters maintain connectivity
- ✅ Cross-region network routing operational
- ✅ Service project permissions validated
- ✅ No network-related incidents during migration
- ✅ Self-hosted backend accessible from all regions

## Strategic Recommendations

### Migration Sequencing
1. **Foundation First**: Migrate shared-vpc before any dependent infrastructure
2. **Dev-Prod Sequence**: Always migrate development environment first
3. **Regional Validation**: Test connectivity in all active regions
4. **Dependency Coordination**: Coordinate closely with clusters repository team

### Risk Mitigation Priorities
1. **Network State Preservation**: Ensure CIDR blocks and routing unchanged
2. **IAM Permission Continuity**: Validate cross-project access throughout migration
3. **Multi-Region Testing**: Test functionality in all geographic regions
4. **Rollback Readiness**: Prepare rapid network restoration procedures

### Long-term Considerations
1. **Regional Expansion**: Plan for easy addition of new GCP regions
2. **Cross-Cloud Integration**: Consider future AWS-GCP connectivity needs
3. **Network Security**: Implement enhanced network security policies
4. **Cost Optimization**: Review regional subnet utilization post-migration

## Project Scope Impact

The discovery of the shared-vpc repository adds **2 foundational GCP environments** that are **critical dependencies** for the clusters repository:

**Updated GCP Infrastructure Scope**:
- **Foundation Layer**: 2 shared VPC environments (this repository)
- **Cluster Layer**: 5 GKE cluster environments (clusters repository)
- **Total GCP Environments**: 7 requiring coordinated migration

**Migration Dependencies**: 
shared-vpc → clusters → (any other GCP services)

This confirms the **multi-layer dependency structure** in the GCP infrastructure and emphasizes the need for **sequential migration planning** with shared-vpc as the foundation layer.