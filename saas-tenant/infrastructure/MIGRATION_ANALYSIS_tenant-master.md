# Pulumi Migration Analysis: tenant-master

## Executive Summary

**MASSIVE SCALE DISCOVERY**: The tenant-master repository contains **99 customer tenant environments**, representing the core SaaS application deployment for each customer. This repository creates the complete Kubernetes-based multi-tenant application stack including identity management, ingress, CDN, monitoring, and customer-specific configurations.

**Migration Complexity: EXTREME (Level 5)**
- 99 customer tenant environments requiring individual migration
- Core customer-facing SaaS application stacks with zero-downtime requirements
- Complex multi-component architecture (Keycloak, Airflow, CDN, ingress)
- International compliance requirements (China AWS partition customers)
- Deep integration dependencies with cluster infrastructure and account services

## Repository Overview

**Repository**: tenant-master  
**Purpose**: Deploy complete SaaS tenant application stacks to customer Kubernetes clusters  
**Stack Count**: 99 customer tenant environments  
**Primary Language**: TypeScript  
**Pulumi Project**: saas-tenant  
**Dependencies**: cluster-master (EKS clusters), account-services (shared services), cluster-services (platform operators)

### Current Pulumi Cloud Configuration
- **Organization**: `cequence`  
- **Project**: `saas-tenant`  
- **Stack Pattern**: `cequence/saas-tenant/{customer-name}-{env}`
- **Backend URL**: Pulumi Cloud (app.pulumi.com)

## Technical Architecture Analysis

### Core Application Components
1. **Identity & Authentication**:
   - Keycloak identity provider (customer-specific configuration)
   - Custom Keycloak themes and branding
   - PostgreSQL backend for identity data
   - mTLS certificate authentication (optional)

2. **Application Ingress & Networking**:
   - NGINX ingress controllers with customer-specific rules
   - SSL/TLS certificate management
   - Multi-domain routing (api, ui, auth, policy-engine)
   - Customer VPN allowlists and IP restrictions
   - Defender CIDR configuration for API security

3. **Content Delivery & CDN**:
   - AWS CloudFront distributions per customer
   - Custom CloudFront functions (TrueClient header handling)
   - WAF integration for application security
   - ACM certificate management for CDN endpoints

4. **Data & Analytics**:
   - Apache Airflow for customer-specific data workflows
   - GitLab DAG projects for workflow management
   - ElastiCache Redis clusters for caching
   - Customer-specific data export IAM roles

5. **Monitoring & Observability**:
   - Spyder integration for tenant monitoring
   - Customer-specific alerting and notifications
   - Health check endpoints and dashboards

### Customer Segmentation Patterns
- **Standard Customers**: Full application stack with all services enabled
- **China Customers**: AWS China partition, restricted services, compliance controls
- **Enterprise Customers**: Enhanced security, dedicated resources, custom configurations
- **Trial Customers**: Limited-time deployments with automatic expiration
- **Air-gapped Customers**: Private ECR registries, restricted external connectivity

### Key Dependencies
- **Cluster Infrastructure**: Requires EKS clusters from cluster-master
- **Platform Services**: Depends on operators from cluster-services-master
- **Shared Services**: DNS, routing, secrets from account-services
- **Custom Components**: Extensive use of `@cequence/pulumi-k8s` library
- **External Integrations**: Workato, GitLab, customer identity providers

## Migration Risk Assessment

### Risk Level: EXTREME (5/5)

**Critical Risk Factors**:
1. **Customer-Facing Application**: Direct impact on customer production workloads
2. **Identity Provider**: Keycloak disruption affects customer authentication/authorization
3. **Zero Downtime Requirement**: SaaS customers cannot tolerate service interruptions
4. **Complex State Management**: Persistent identity data, user sessions, application state
5. **International Compliance**: China partition customers have strict regulatory requirements
6. **Multi-Component Dependencies**: Failure of any component affects entire customer experience

**Specific Technical Risks**:
- **Keycloak Identity Data**: User accounts, roles, permissions, active sessions
- **PostgreSQL Databases**: Customer identity and configuration data
- **CloudFront Distributions**: DNS propagation delays, certificate renewals
- **Ingress Controllers**: Load balancer state, SSL certificates, routing rules
- **Redis Clusters**: Session data, cache invalidation, application state
- **Airflow Workflows**: Customer data pipelines, scheduled jobs, workflow state

### Customer Impact Assessment
- **Immediate Impact**: Complete service outage for 99 customer environments
- **Business Impact**: Direct revenue impact, customer churn risk, SLA breaches
- **Reputation Impact**: Customer trust, market reputation, competitive disadvantage
- **Compliance Impact**: Regulatory violations in restricted regions
- **Support Impact**: Massive support ticket volume, customer escalations

## Migration Strategy Recommendations

### Phase 1: Pre-Migration Foundation (6-8 weeks)
1. **Customer Communication Strategy**:
   - Executive-level customer notifications (3-4 weeks advance)
   - Detailed technical briefings for customer DevOps teams  
   - Maintenance window coordination across global timezones
   - Rollback communication procedures

2. **State Preservation Strategy**:
   - Complete Keycloak identity data backup procedures
   - PostgreSQL database dump and restore testing
   - CloudFront distribution state documentation
   - Redis cluster state snapshot procedures
   - Airflow workflow and DAG state preservation

3. **Testing & Validation Infrastructure**:
   - Isolated testing environment replicating full production stack
   - Customer-specific test scenarios and validation procedures
   - Identity provider migration testing with non-production data
   - CDN cutover testing with DNS management

### Phase 2: Pilot Migration Program (4-6 weeks)
1. **Pilot Customer Selection**:
   - Choose development/staging environments first
   - Select customers with flexible maintenance agreements
   - Avoid high-volume or mission-critical customers initially
   - Exclude China partition customers from initial pilots

2. **Migration Procedure Development**:
   - Keycloak identity data export/import procedures
   - PostgreSQL database migration with zero downtime
   - CloudFront distribution recreation with DNS management
   - Ingress controller state transfer and validation
   - End-to-end application functionality testing

### Phase 3: Production Migration Execution (12-16 weeks)
1. **Batch Processing Strategy**:
   - 3-5 customers per maintenance window (risk mitigation)
   - Regional grouping to minimize timezone impact
   - 72-hour monitoring period post-migration per batch
   - Dedicated customer success management during migration

2. **Special Customer Handling**:
   - China partition customers: Separate team, compliance approval
   - Enterprise customers: Dedicated migration windows, on-site support
   - High-volume customers: Extended testing, staged rollout
   - Trial customers: Expedited migration or graceful termination

3. **Critical Path Management**:
   - Identity provider migration with session preservation
   - DNS cutover coordination with minimal TTL impact
   - Certificate renewal and validation procedures
   - Application health monitoring and alerting

### Phase 4: Post-Migration Optimization (4-6 weeks)
1. **Performance Validation**:
   - Application response time benchmarking
   - Identity provider performance testing
   - CDN cache hit ratio optimization
   - Database query performance analysis

2. **Customer Success Validation**:
   - Customer satisfaction surveys
   - Support ticket trend analysis
   - Application usage pattern validation
   - Performance metric comparison (pre/post migration)

## Timeline Estimates

### Detailed Migration Timeline
- **Foundation Phase**: 6-8 weeks (customer communication, testing infrastructure)
- **Pilot Migration**: 4-6 weeks (low-risk customer validation)
- **Production Migration**: 12-16 weeks (99 customer tenant environments)
- **Optimization Phase**: 4-6 weeks (performance tuning, documentation)

**Total Timeline**: 26-36 weeks (6-9 months)

### Critical Dependencies
1. Customer agreement to maintenance windows and migration timeline
2. Self-hosted backend deployment and hardening
3. Identity provider migration tooling development and testing
4. China partition compliance approval and specialized team
5. Customer success team scaling and training

## Resource Requirements

### Migration Team Structure
- **Technical Lead**: Senior architect with multi-tenant SaaS experience
- **Identity Management Specialists**: 2-3 engineers with Keycloak expertise
- **Infrastructure Engineers**: 3-4 engineers with Pulumi/Kubernetes experience
- **Customer Success Managers**: 3-4 dedicated CSMs for customer communication
- **Site Reliability Engineers**: 3-4 SREs for monitoring and incident response
- **DevOps Engineers**: 2-3 engineers for CI/CD pipeline management
- **QA Engineers**: 2-3 engineers for testing and validation procedures

### Infrastructure Requirements
- **Self-Hosted Backend**: Enterprise-grade with high availability and backup
- **Migration Tooling**: Custom automation for identity and state migration
- **Testing Environments**: Full production replicas for validation
- **Monitoring Stack**: Enhanced observability during migration windows
- **Communication Tools**: Customer notification and status page systems

## Success Criteria

### Technical Success Metrics
- ✅ Zero customer authentication outages during migration
- ✅ All customer identity data preserved with 100% integrity
- ✅ Application performance maintained or improved post-migration
- ✅ All CDN distributions functional with no DNS resolution issues
- ✅ Complete Airflow workflow continuity for all customers
- ✅ Redis cluster state preserved with no cache invalidation issues

### Business Success Metrics
- ✅ Customer satisfaction maintained above 95% during migration period
- ✅ No customer churn directly attributable to migration activities
- ✅ All 99 customer environments successfully migrated within timeline
- ✅ Zero SLA breaches related to migration activities
- ✅ Self-hosted backend achieving 99.95% uptime post-migration
- ✅ Customer support ticket volume returns to baseline within 30 days

### Operational Success Metrics
- ✅ Migration team knowledge transfer to operations team
- ✅ Enhanced monitoring and alerting systems operational
- ✅ Customer communication and incident response procedures validated
- ✅ Disaster recovery procedures tested and documented

## Strategic Recommendations

### Immediate Executive Actions (Next 2 weeks)
1. **Board/Executive Approval**: Present revised scope and investment requirements
2. **Customer Advisory**: Begin executive-level customer communications
3. **Team Assembly**: Recruit specialized migration team with SaaS experience
4. **Risk Insurance**: Evaluate cyber/business interruption insurance coverage

### Strategic Alternatives to Consider
1. **Managed Service Migration**: Evaluate AWS/GCP managed identity providers
2. **Hybrid Approach**: Keep critical services on Pulumi Cloud during transition
3. **Customer-Specific Migration**: Allow enterprise customers to opt for managed migration
4. **Service Redesign**: Consider microservices decomposition to reduce migration complexity

### Critical Success Factors
1. **Customer Buy-in**: Unanimous customer agreement to migration timeline
2. **Executive Sponsorship**: C-level commitment to resource allocation and timeline
3. **Technical Excellence**: Zero-defect migration procedures and tooling
4. **Communication Excellence**: Proactive, transparent customer communication
5. **Risk Management**: Comprehensive rollback and emergency response procedures

**STRATEGIC RECOMMENDATION**: This repository represents the highest business risk in the entire migration due to direct customer revenue impact. Consider this the "core business critical" migration requiring executive-level oversight and maximum resource allocation.

## Project Scope Revision

With the discovery of tenant-master containing 99 customer environments, the **total known customer environments across three repositories is now 279+**:
- cluster-master: 80+ customer environments (EKS clusters)  
- cluster-services-master: 100 customer environments (Kubernetes services)
- tenant-master: 99 customer environments (SaaS applications)

This represents a **3x increase** from the original estimate of ~90 services, fundamentally changing the project scope, timeline, and resource requirements.