# SaaS Tenant Cluster - Migration Analysis & Documentation

## **Repository Information**

### **GitLab Repository**
- **Project Path**: `saas/infrastructure/tenants/cluster`
- **Repository Name**: `cluster-master`
- **GitLab URL**: `https://gitlab.com/cequence/saas/infrastructure/tenants/cluster`

### **Pulumi Project Details**
- **Project Name**: `saas-tenant-cluster`
- **Stack Naming Pattern**: `cequence/{tenant-name}-{env}` (e.g., `cequence/esteelauder-china-prod`)
- **Workload Account**: `saas-tenants` (inferred from template usage)

---

## **Service Overview**

### **Purpose**
The `saas-tenant-cluster` is a **MASSIVE multi-tenant Kubernetes cluster provisioning service** that creates dedicated EKS clusters for SaaS customers. This is the **largest and most complex service** in the infrastructure portfolio.

### **Business Function**
- **Customer Cluster Provisioning**: Dedicated EKS clusters per customer/tenant
- **Multi-Tenant Platform**: Kubernetes-based SaaS platform foundation
- **ArgoCD Integration**: GitOps deployment platform for each tenant
- **Network Infrastructure**: VPC, subnets, routing per customer
- **Ingress Management**: Load balancer and domain routing per tenant
- **Security & Access Control**: RBAC, IAM integration, cluster access management

---

## **Scale & Complexity Analysis**

### **MASSIVE SCALE: 80+ Customer Environments**

#### **Environment Count**: **~80 active customer environments**
Based on Pulumi stack files, active customers include:
- **Enterprise Customers**: American Express, Samsung, T-Mobile, Navy Federal, Garmin
- **Financial Services**: Banco Inter, BNP, Itau, CitiBank, First Tech Fed  
- **Retail/E-commerce**: Ulta, Five Below, Dominos, Poshmark, VSCO
- **Technology**: SAP, Synopsis, Enphase, Houzz, Ping Identity
- **Global Customers**: EsteeLauder China (separate AWS partition)
- **Internal/Dev**: api-security-dev, cequence-internal-prod, various dev environments

#### **Geographic Distribution**
- **Primary Regions**: us-west-2 (majority), us-east-1
- **China Region**: cn-northwest-1 (EsteeLauder China - separate AWS partition)
- **Multi-region Support**: Some customers have multiple regions/stages

#### **Environment Patterns**
- **Production Focus**: Most environments are `{customer}-prod`
- **Staging Environments**: Some customers have `{customer}-stage-prod`
- **Development**: Limited dev environments (api-security-dev, pmdemo1-dev)
- **Special Cases**: amex (multiple regions), garmin (stage + prod), etc.

---

## **Current Architecture**

### **Technology Stack**
- **Runtime**: Node.js/TypeScript
- **Infrastructure as Code**: Pulumi with custom component library
- **Cloud Provider**: AWS EKS (+ China partition)
- **Kubernetes**: EKS with Karpenter auto-scaling
- **CI/CD Integration**: Custom SaaS-Tenant GitLab template
- **Custom Components**: `@cequence/pulumi-k8s` proprietary library

### **Infrastructure Components**

#### **1. EKS Cluster (`Cluster`)**
- **Kubernetes Version**: Configurable per customer (1.23-1.32)
- **Karpenter Auto-scaling**: Spot/on-demand instances, ARM64 support
- **Managed Node Groups**: Customer-specific instance types
- **Cluster Add-ons**: EBS CSI, CoreDNS, VPC CNI, Kube-proxy
- **Network Policies**: Multi-tenant security (when enabled)

#### **2. Network Infrastructure**
- **VPC**: Dedicated VPC per customer
- **Subnets**: Public/private subnet architecture
- **NAT Gateways**: Outbound internet connectivity
- **VPC Flow Logs**: Optional network monitoring

#### **3. Ingress & DNS (`IngressNginx`)**
- **Load Balancer**: Application Load Balancer per tenant
- **Domain Routing**: `*.{tenant}.{zone}` DNS records
- **TLS Termination**: Certificate management
- **Traffic Routing**: Customer-specific ingress patterns

#### **4. ArgoCD Integration**
- **Self-Registration**: Clusters auto-register with ArgoCD
- **Service Accounts**: Dedicated ArgoCD manager accounts
- **Project Isolation**: ArgoCD projects per tenant
- **GitOps Platform**: Complete CI/CD integration

#### **5. Access Control (`ClusterAccess`)**
- **Azure AD Integration**: SSO with customer success/dev groups
- **RBAC**: Cluster admin vs read-only permissions
- **Cross-Account**: Different roles for prod vs dev

---

## **CI/CD Integration**

### **Custom GitLab CI Template**
```yaml
include:
  - project: 'cequence/saas/infrastructure/tenants/cicd-templates'
    ref: master
    file: 'SaaS-Tenant.gitlab-ci.yml'  # CUSTOM template (not standard Pulumi)
```

### **Tenant-Specific Operations**
- **Cluster Provisioning**: Create full EKS environment
- **Cluster Destruction**: Complete teardown with safeguards  
- **Kubeconfig Generation**: Access credentials for operations
- **Matrix Deployment**: Variable `$TENANT` for customer selection

---

## **Dependencies & Integration**

### **Cross-Stack Dependencies**
```typescript
// Heavy dependency on account-services
const accountServicesStack = new pulumi.StackReference(
  `cequence/saas-tenant-account-services/${getAccountServicesStackName()}`
)

// DNS and routing from account-services
export const zoneId = accountServicesStack.requireOutput('zoneId')
export const zoneName = accountServicesStack.requireOutput('zoneName')

// ArgoCD integration
const centralOpsStack = new pulumi.StackReference(`cequence/central-ops-argocd/${env}`)
```

### **Dependency Chain**
1. **Upstream Dependencies** (must be migrated first):
   - `saas-tenant-account-services/*` (DNS zones, routing, secrets)
   - `central-ops-argocd/*` (ArgoCD platform, kubeconfig access)

2. **Downstream Dependencies** (depend on this service):
   - **ALL CUSTOMER APPLICATIONS** - Every SaaS customer depends on their cluster
   - ArgoCD applications deployed to these clusters
   - Customer-specific services and workloads

---

## **Migration Complexity Assessment**

### **Service Migration (Extreme Complexity Category)**

**Effort**: ~20–25 days (EXTREME complexity - largest service in portfolio)

#### **Deliverables**
- State export/import for **~80 customer environments**
- Customer coordination for **ALL active SaaS customers**
- Kubernetes cluster continuity validation
- ArgoCD platform integration verification
- DNS routing validation for all customer domains
- Cross-stack dependency resolution (account-services, central-ops)
- Customer-specific configuration migration
- China partition special handling (separate AWS regions)
- Load balancer and ingress functionality validation
- Access control and RBAC verification

#### **Acceptance Criteria**
- `pulumi preview` shows zero drift across all 80 environments
- All customer Kubernetes clusters remain operational
- ArgoCD self-registration functions correctly for all tenants
- DNS resolution works for all customer domains (`*.{tenant}.{zone}`)
- Load balancers and ingress controllers operational
- Customer applications continue deploying via ArgoCD
- Access control (Azure AD SSO) functions correctly
- China partition environments fully isolated and functional
- All Karpenter auto-scaling policies active
- Zero customer downtime during migration

#### **Migration Risks (EXTREME)**
1. **Customer Impact**: 80+ active SaaS customers affected simultaneously
2. **Business Continuity**: Customer workloads could be disrupted
3. **Platform Dependency**: Core platform service - everything depends on this
4. **Scale Complexity**: Largest repository by environment count
5. **China Partition**: Separate AWS partition with different IAM/networking
6. **ArgoCD Integration**: Complex GitOps platform integration
7. **DNS Routing**: Customer-facing domain routing dependencies
8. **State Volume**: Massive state files with complex resource graphs

#### **Nice to Have**
- Customer-specific migration windows coordinated with CSMs
- Blue/green cluster provisioning for zero-downtime migration
- Automated rollback triggers for cluster health failures
- Real-time customer impact monitoring during migration
- Staged migration by customer criticality (enterprise last)

### **Complexity Factors (10/10 - MAXIMUM)**
1. **Extreme Scale**: 80+ environments vs typical 2-4 environments
2. **Customer Criticality**: Direct customer-facing infrastructure
3. **Platform Foundation**: Everything depends on Kubernetes clusters
4. **Cross-Stack Integration**: Deep dependencies on account-services/central-ops
5. **Multi-Partition**: China AWS partition with different networking/IAM
6. **Custom Components**: Proprietary `@cequence/pulumi-k8s` library
7. **ArgoCD Complexity**: GitOps platform self-registration logic
8. **Customer Coordination**: Requires coordination with ALL customers

---

## **Migration Strategy & Execution Plan**

### **Migration Sequencing: FINAL PHASE (Phase 4)**

**Effort**: ~20–25 days with dedicated team

#### **Deliverables**
- **Phase 1-3 Prerequisites**: All dependencies migrated (account-services, central-ops)
- **Customer Communication Plan**: Coordination with Customer Success team
- **Migration Batching**: Customers grouped by criticality and coordination needs
- **State Migration Strategy**: Bulk export/import with validation at scale
- **Downtime Windows**: Customer-specific maintenance windows
- **China Partition Coordination**: Separate migration approach for different AWS partition
- **ArgoCD Re-registration**: Validation of GitOps platform integration
- **DNS Propagation**: Verification of customer domain routing

#### **Acceptance Criteria**
- All upstream dependencies (account-services, central-ops) successfully migrated
- Customer Success team briefed and coordination plan approved
- Migration batching strategy validated with enterprise customer SLAs
- State export completes for all 80 environments without corruption
- All customer clusters remain operational throughout migration
- ArgoCD platform integration verified for all tenants
- DNS routing validated for all customer domains
- China partition environments maintain isolation and functionality
- Zero impact to customer-facing applications and services
- All customer-specific configurations preserved and functional

#### **Rollback Plan**
- **Customer SLA Adherence**: Rollback within customer maintenance windows
- **Cluster Backup**: Full EKS cluster snapshots before migration
- **DNS Failover**: Emergency DNS routing for critical customers
- **Staged Rollback**: Per-customer rollback capability
- **China Partition**: Independent rollback strategy for different AWS regions
- **Customer Communication**: Real-time status updates during rollback

#### **Nice to Have**
- **Zero-Downtime Strategy**: Blue/green cluster migration where possible
- **Customer Dashboard**: Real-time migration status per customer
- **Automated Health Checks**: Continuous cluster and application monitoring
- **Priority-Based Migration**: Enterprise customers migrated during optimal windows

---

## **Customer Impact Assessment**

### **Enterprise Customer Coordination Required**
Based on stack names, key customers requiring special coordination:
- **Financial Services**: American Express, Navy Federal, Banco Inter, BNP
- **Large Enterprise**: Samsung, T-Mobile, SAP, Garmin, Ulta
- **International**: EsteeLauder China (separate AWS partition)
- **High-Volume**: Dominos, Poshmark, VSCO (consumer-facing applications)

### **Customer Success Integration**
- **CSM Coordination**: Customer Success Managers for each enterprise account
- **SLA Requirements**: Customer-specific uptime and maintenance windows
- **Communication Plan**: Pre-migration, during migration, post-migration updates
- **Escalation Process**: Direct customer escalation paths for issues

---

## **Special Considerations**

### **China Partition (EsteeLauder)**
- **Separate AWS Partition**: `aws-cn` vs `aws` - different networking/IAM
- **Dedicated Migration**: Separate migration plan and execution
- **Compliance Requirements**: China-specific data residency and compliance
- **Coordination Complexity**: Different time zones and stakeholder management

### **Custom Component Library**
- **`@cequence/pulumi-k8s`**: Proprietary Kubernetes component library
- **Version Dependencies**: Library version compatibility across environments
- **Migration Risk**: Custom components may have backend-specific behavior

### **ArgoCD Self-Registration**
- **Complex Integration**: Clusters self-register with ArgoCD using secrets
- **Token Management**: Service account tokens for ArgoCD access
- **Cross-Cluster**: Integration with central-ops ArgoCD deployment

---

## **Execution Checklist & Validation**

### **Pre-Migration Validation (Critical)**
- [ ] **Dependency Verification**: account-services and central-ops-argocd migrated successfully
- [ ] **Customer Success Alignment**: All CSMs briefed and migration windows approved
- [ ] **China Partition Planning**: Separate migration strategy for EsteeLauder China
- [ ] **Enterprise Customer SLAs**: Maintenance windows negotiated and approved
- [ ] **State Export Testing**: Validate export process on non-critical environments
- [ ] **ArgoCD Integration**: Verify central-ops ArgoCD platform operational
- [ ] **DNS Resolution Baseline**: Document current customer domain routing
- [ ] **Rollback Procedures**: Tested and validated on development environments

### **Migration Execution (Per Customer Batch)**
- [ ] **Customer Notification**: 72-hour advance notice sent
- [ ] **State Export**: Complete backup with integrity verification
- [ ] **Backend Configuration**: S3/DynamoDB backend configured for customer
- [ ] **State Import**: Import with comprehensive validation
- [ ] **Cluster Health Check**: EKS cluster operational status verified
- [ ] **ArgoCD Re-registration**: Cluster self-registration with ArgoCD validated
- [ ] **DNS Verification**: Customer domain routing tested and confirmed
- [ ] **Application Validation**: Customer applications deployed successfully
- [ ] **Customer Sign-off**: Customer Success confirmation of operational status

### **Post-Migration Validation (Per Customer)**
- [ ] **Full Platform Testing**: End-to-end customer workflow validation
- [ ] **Performance Baseline**: Cluster performance metrics compared to pre-migration
- [ ] **ArgoCD Applications**: All customer applications deploying successfully
- [ ] **Monitoring Integration**: Cluster monitoring and alerting operational
- [ ] **Access Control**: Azure AD SSO and RBAC functioning correctly
- [ ] **Customer Feedback**: Direct customer confirmation of system functionality
- [ ] **Documentation Update**: Customer-specific runbooks and procedures updated

---

## **Key Insights for Overall Migration**

### **Scale Revelation**
- **Largest Service**: 80+ environments vs typical 2-4 environments per service
- **Customer-Facing**: Direct business impact to ALL SaaS customers
- **Migration Bottleneck**: This service will determine overall migration timeline
- **Resource Requirements**: Dedicated team and extended timeline required

### **Business Impact**
- **Revenue Risk**: Customer churn risk if migration fails
- **SLA Compliance**: Customer contractual obligations
- **Competitive Risk**: Service disruption affects competitive position
- **Stakeholder Coordination**: Extensive internal and external coordination required

### **Technical Complexity**
- **Custom Components**: Proprietary Kubernetes library adds complexity
- **Multi-Partition**: China AWS partition requires separate approach
- **Platform Integration**: Deep ArgoCD and DNS integration dependencies
- **State Volume**: Massive state files with complex resource relationships

---

## **Updated Migration Effort Estimates**

### **Revised Overall Project Scope**
- **Previous Estimate**: ~90 services
- **Cluster Reality**: 80+ environments in ONE repository
- **Total Scope**: Cluster service = equivalent of 80% of entire migration effort

### **Resource Allocation Recommendation**
- **Dedicated Cluster Team**: 3-4 senior engineers exclusively for cluster migration
- **Extended Timeline**: 6-8 weeks for cluster service alone
- **Customer Success Integration**: Dedicated CSM coordination resource
- **China Partition Specialist**: Engineer with China AWS partition experience

---

**Service Classification**: **MEGA-SCALE PLATFORM FOUNDATION - EXTREME COMPLEXITY (10/10)**  
**Migration Priority**: **FINAL PHASE - AFTER ALL DEPENDENCIES**  
**Business Impact**: **CRITICAL - ALL CUSTOMER INFRASTRUCTURE - REVENUE RISK**