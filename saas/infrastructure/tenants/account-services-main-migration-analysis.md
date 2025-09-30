# SaaS Account Services - Migration Analysis & Documentation

## **Repository Information**

### **GitLab Repository**
- **Project Path**: `saas/infrastructure/tenants/account-services`
- **Repository Name**: `account-services-main`
- **GitLab URL**: `https://gitlab.com/cequence/saas/infrastructure/tenants/account-services`

### **Pulumi Project Details**
- **Project Name**: `saas-tenant-account-services`
- **Stack Naming Pattern**: `cequence/saas-tenant-account-services/$CI_ENVIRONMENT_NAME`
- **Workload Account**: `saas-tenants`

---

## **Service Overview**

### **Purpose**
The `saas-tenant-account-services` is a **critical platform infrastructure service** that provides foundational multi-tenant SaaS capabilities across AWS regions. This service manages core platform services that other tenant applications depend on.

### **Business Function**
- **Tenant Routing**: DNS and routing infrastructure for multi-tenant access
- **Security Platform**: Regional WAF, CloudFront security functions
- **Secret Management**: Centralized secret distribution across regions
- **GitOps Platform**: ArgoCD setup and configuration
- **Container Infrastructure**: ECR pull-through cache for registry optimization
- **Monitoring Integration**: Datadog, Grafana, and observability setup

---

## **Current Architecture**

### **Technology Stack**
- **Runtime**: Node.js/TypeScript
- **Infrastructure as Code**: Pulumi
- **Cloud Provider**: AWS (multi-region deployment)
- **Deployment**: GitLab CI/CD with cross-account role assumption

### **Multi-Region Deployment**
Deploys across **all AWS regions** with region-specific configurations:
- **WAF-enabled regions**: Most regions except disabled list
- **Tenant routing regions**: Configurable per environment
- **Global services**: CloudFront, Route53 (us-east-1)

---

## **Environment Configuration**

### **Environments (4 total)**

| **Environment** | **Purpose** | **AWS Region** | **Domain** | **Special Config** |
|-----------------|-------------|----------------|------------|-------------------|
| `dev` | Development | us-west-2 | s.cequence.cloud | All regions WAF-disabled except core |
| `prod` | Production | us-west-2 | cequence.cloud | Selective tenant routing regions |
| `esteelauder-china-dev` | Customer Dev | us-west-2 | Custom | China-specific deployment |
| `esteelauder-china-prod` | Customer Prod | us-west-2 | Custom | China-specific deployment |

### **Customer-Specific Pattern**
- **EsteeLauder China**: Separate environments with different configuration
- **Conditional Logic**: `isNotEsteeLauderChina()` controls feature deployment
- **Isolation Strategy**: Complete infrastructure separation for specific customers

---

## **Infrastructure Components**

### **Core Services Deployed**

#### **1. Tenant Routing (`TenantRouting`)**
- **Route53 Hosted Zones**: Primary and Ultra API zones
- **DNS Management**: Tenant-specific domain routing
- **Outputs**: Zone IDs, names, and name servers

#### **2. Security Infrastructure**
- **Regional WAF** (`RegionalWaf`): Per-region web application firewall
- **CloudFront WAF** (`DefaultCdnWebAcl`): Global CDN protection
- **CloudFront Functions** (`TrueClientCqCloudFrontFunction`): Custom edge logic

#### **3. Secret Management (`GlobalSecrets`)**
- **Multi-region secret distribution**
- **Third-party integrations**: Datadog, Grafana, Airflow credentials
- **Container registry credentials**: Docker Hub, GitLab

#### **4. Container Infrastructure**
- **ECR Pull-Through Cache** (`EcrPullThroughCache`): Per-region cache setup
- **Registry Optimization**: Docker Hub and GitLab registry caching

#### **5. GitOps Platform (`ArgocdSetup`)**
- **ArgoCD Configuration**: GitOps deployment platform
- **IAM Roles**: ArgoCD component roles and permissions

#### **6. Systems Management (`AwsSystemsManager`)**
- **Session Manager**: Secure instance access
- **Cross-account integration**: With workload account roles

#### **7. Monitoring & Observability**
- **Defender Configuration** (`DefenderConfiguration`): Security monitoring
- **CDN Prefix Lists** (`CdnPrefixLists`): Network access control

---

## **CI/CD Integration**

### **GitLab CI Configuration**
```yaml
include:
  - project: 'cequence/cicd-templates'
    ref: master
    file: 'Pulumi.gitlab-ci.yml'

variables:
  WORKLOAD_ACCOUNT: saas-tenants
  PULUMI_STACK: cequence/saas-tenant-account-services/$CI_ENVIRONMENT_NAME
```

### **Deployment Pattern**
- **Template**: Uses standard `Pulumi.gitlab-ci.yml` template
- **Cross-account**: Assumes roles in `saas-tenants` workload account
- **Environments**: Deploys to dev/prod based on Git branch/MR patterns

---

## **Dependencies & Integration**

### **Cross-Stack Dependencies**
```typescript
// References other Pulumi stacks
const centralOpsStack = new pulumi.StackReference(`cequence/central-ops/${env}`)
const workloadAccountStack = new pulumi.StackReference(`cequence/workload-account/saas-tenants-${config.getStackEnv()}`)
```

### **Dependency Chain**
1. **Upstream Dependencies** (must be migrated first):
   - `workload-account/saas-tenants-*` (provides deployment roles)
   - `central-ops/*` (provides OIDC provider)

2. **Downstream Dependencies** (depend on this service):
   - Any service using tenant routing
   - Services requiring regional WAF protection
   - Applications using ArgoCD for deployment

---

## **Secret Management Analysis**

### **Current Approach: Pulumi Cloud Secrets**
All secrets are encrypted using Pulumi Cloud's secret provider:

#### **Secret Categories**
1. **Container Registry Credentials**
   - `dockerHubUser` / `dockerHubToken`
   - `gitlabUser` / `gitlabToken`

2. **Third-party Service APIs**
   - `datadog.apiKey` / `datadog.cqPrimeApiKey`
   - `grafana.apiKey`
   - `airflow.username` / `airflow.password`

3. **Platform Configuration**
   - `globalSecrets`: Nested secret structure
   - Region-specific configurations

#### **Migration Impact**
- **High secret volume**: ~12+ secrets per environment
- **Nested structure**: Complex secret hierarchies
- **Multi-environment**: Secrets replicated across all 4 environments

---

## **Migration Complexity Assessment**

### **Service Migration (Per-Service Category)**

**Effort**: ~5–6 days (HIGH complexity platform service)

#### **Deliverables**
- State export from Pulumi Cloud for 4 environments
- Secret migration for ~48 encrypted values (12 per environment)
- Multi-region state consistency validation
- Cross-stack dependency verification
- Customer-specific environment coordination (EsteeLauder China)
- DNS/routing functionality validation
- Platform service integration testing

#### **Acceptance Criteria** 
- `pulumi preview` shows zero drift post-migration across all 4 environments
- All cross-stack references resolve correctly (`central-ops`, `workload-account`)
- DNS routing works for all tenant domains (`cequence.cloud`, `s.cequence.cloud`)
- Regional WAF policies active in all enabled regions
- ArgoCD platform remains operational
- ECR pull-through cache functional across regions
- All secrets decrypt correctly in new backend
- Customer environments (China) fully isolated and functional

#### **Migration Risks (HIGH)**
1. **Service dependencies**: 15+ downstream services may fail if this breaks
2. **Multi-region coordination**: State consistency across ~20 AWS regions
3. **DNS/routing impact**: Could affect all tenant access to platform
4. **Secret migration**: Complex nested secret structures with 48 values
5. **Customer SLA impact**: EsteeLauder China has dedicated environments

#### **Nice to Have**
- Automated drift detection for multi-region resources
- Blue/green deployment capability for DNS failover
- Automated rollback triggers for failed health checks
- Customer-specific monitoring dashboards

### **Complexity Factors (8/10)**
1. **Multi-region deployment**: All AWS regions with region-specific configs
2. **Cross-stack dependencies**: Multiple upstream dependencies (`central-ops`, `workload-account`)
3. **Customer-specific logic**: China-specific deployment patterns requiring coordination
4. **High secret volume**: 48 encrypted secrets across nested structures
5. **Platform criticality**: Foundation service - other services depend on this
6. **Complex infrastructure**: WAF, CDN, DNS, GitOps, monitoring integration

---

## **Migration Strategy & Execution Plan**

### **Migration Sequencing: LATE PHASE (Phase 3)**

**Effort**: ~5–6 days total

#### **Deliverables**
- **Phase 1 Prerequisites**: `workload-account/saas-tenants-*` and `central-ops/*` stacks migrated
- **State Migration**: Export/import for all 4 environments with validation
- **Secret Migration**: Bulk transfer of 48 encrypted values to new backend
- **Environment Sequencing**: dev → esteelauder-china-dev → prod → esteelauder-china-prod
- **Cross-stack Validation**: All StackReference calls function correctly
- **Customer Coordination**: EsteeLauder stakeholder communication and approval
- **Health Validation**: Full platform functionality verification

#### **Acceptance Criteria**
- All upstream dependencies successfully migrated and validated
- State export completes without corruption for all environments
- All 48 secrets successfully decrypt in new backend
- Cross-stack references resolve (central-ops OIDC, workload-account roles)
- DNS resolution works for all domains (cequence.cloud, s.cequence.cloud)
- Regional WAF active in all configured regions
- ArgoCD platform operational post-migration
- Customer environments maintain full isolation
- Zero downtime during migration execution

#### **Rollback Plan** 
- **State Snapshots**: Complete Pulumi Cloud export before migration start
- **DNS Failover**: Temporary Route53 records for emergency rollback
- **Rollback Window**: 30-minute maximum rollback time
- **Customer SLA**: Pre-agreed maintenance window for China environments
- **Validation Gates**: Go/no-go decision points between environments

#### **Nice to Have**
- **Automated Rollback**: Triggered by health check failures
- **Blue/Green DNS**: Zero-downtime DNS switching capability
- **Customer Dashboard**: Real-time migration status for EsteeLauder
- **Parallel Environment Migration**: If dependency validation allows

---

## **Key Insights for Overall Migration**

### **Patterns Discovered**
1. **Customer-specific environments**: Not all services are just dev/prod
2. **Platform service complexity**: Some services are significantly more complex
3. **Deep cross-stack integration**: Services are highly interconnected
4. **Multi-region patterns**: Regional deployment is common
5. **Heavy secret usage**: Pulumi Cloud secret provider heavily utilized

### **Impact on Migration Planning**
- **Timeline**: Platform services require more time
- **Expertise**: High-complexity services need senior engineers
- **Risk management**: Critical services need extensive testing
- **Customer coordination**: Special environments need stakeholder communication

---

## **Monitoring & Validation**

### **Post-Migration Validation**
1. **DNS resolution**: Verify tenant routing works
2. **WAF functionality**: Test security rules
3. **Secret access**: Validate all integrations work
4. **ArgoCD connectivity**: Verify GitOps platform
5. **Cross-stack outputs**: Confirm dependent services can access

### **Health Checks**
- Regional WAF deployment status
- CloudFront function execution
- ECR pull-through cache functionality
- Systems Manager session connectivity

---

## **Documentation Updates Needed**

1. **Runbooks**: Update operational procedures for new backend
2. **Secret management**: Document new secret access patterns
3. **Troubleshooting**: Regional deployment debugging
4. **Customer procedures**: EsteeLauder China environment management

---

## **Execution Checklist & Validation**

### **Pre-Migration Validation**
- [ ] **Dependency Check**: `workload-account/saas-tenants-*` successfully migrated and tested
- [ ] **Dependency Check**: `central-ops/*` stacks migrated with OIDC provider functional
- [ ] **State Export**: Complete backup of all 4 environments to local files
- [ ] **Secret Inventory**: Catalog and validate all 48 encrypted secrets
- [ ] **Customer Communication**: EsteeLauder China stakeholder approval obtained
- [ ] **Rollback Testing**: Validate rollback procedures in non-prod environment
- [ ] **Health Check Baseline**: Document current DNS/WAF/ArgoCD functionality

### **Migration Execution (Per Environment)**
- [ ] **State Export**: `pulumi stack export` with integrity verification
- [ ] **Backend Switch**: Configure S3/DynamoDB backend URL
- [ ] **State Import**: `pulumi stack import` with validation
- [ ] **Secret Migration**: Bulk import with decryption verification
- [ ] **Cross-stack Test**: Validate all StackReference calls
- [ ] **Health Validation**: DNS, WAF, ArgoCD, ECR functionality check
- [ ] **Go/No-Go Gate**: Decision point before next environment

### **Post-Migration Validation**
- [ ] **End-to-End Testing**: Full platform integration test suite
- [ ] **Performance Baseline**: Compare against pre-migration metrics
- [ ] **Downstream Services**: Validate dependent services still function
- [ ] **Customer Acceptance**: EsteeLauder China environment sign-off
- [ ] **Documentation Update**: New backend procedures and runbooks
- [ ] **Team Enablement**: Training session on new backend operations
- [ ] **Pulumi Cloud Cleanup**: Archive and decommission old stacks

---

**Service Classification**: **Platform Foundation Service - HIGH COMPLEXITY**  
**Migration Priority**: **Late Sequence (after dependencies)**  
**Business Impact**: **CRITICAL - Customer-facing platform infrastructure**