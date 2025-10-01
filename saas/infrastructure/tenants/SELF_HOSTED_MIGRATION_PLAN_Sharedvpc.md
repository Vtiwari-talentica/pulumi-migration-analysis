# Self-Hosted Pulumi Migration Plan: shared-vpc (GCP Foundation Network)

## Executive Summary

This document outlines the detailed migration plan for the **shared-vpc repository** (2 GCP foundation environments) from Pulumi Cloud to a self-hosted Pulumi backend using **GCS + Cloud Storage locking + Vault** architecture, optimized for GCP infrastructure.

**Migration Scope**: 2 shared VPC environments (dev/prod) serving as network foundation  
**Target Architecture**: Self-hosted with GCS state storage, Cloud Storage locking, Vault secrets, GitLab CI/CD  
**Timeline**: 4-5 weeks total migration  
**Priority**: **CRITICAL - Must migrate first** (foundation dependency for all GCP infrastructure)

## Current State Analysis

### Existing Pulumi Cloud Setup
```yaml
# Current backend configuration
Organization: cequence
Project: saas-shared-vpc
Stacks:
  - cequence/saas-shared-vpc/dev
  - cequence/saas-shared-vpc/prod

# Current GCP Projects
Host Projects:
  - saas-vpc-host-dev-7ce69e6
  - saas-vpc-host-prod-e7644fd
Service Projects:
  - saas-tenants-svc-dev-508dc2b
  - saas-tenants-svc-prod-72a2d72
```

### Current Configuration Analysis
- **No explicit secrets**: This repository manages network infrastructure, minimal secret requirements
- **Cross-project IAM**: Complex service account permissions across host/service projects
- **Regional complexity**: 40+ GCP regions with subnet allocations
- **Critical dependency**: Foundation for all GCP GKE clusters

## Self-Hosted Architecture Design

### 1. State Storage Backend (GCS + Cloud Storage Locking)

#### GCS Bucket Configuration
```yaml
# GCS bucket structure optimized for GCP infrastructure
gs://pulumi-state-saas-shared-vpc/
├── dev/
│   ├── .pulumi/
│   │   ├── stacks/
│   │   └── meta.yaml
│   └── history/
│       ├── dev.checkpoint.1
│       ├── dev.checkpoint.2
│       └── ...
└── prod/
    ├── .pulumi/
    │   ├── stacks/
    │   └── meta.yaml
    └── history/
        ├── prod.checkpoint.1
        ├── prod.checkpoint.2
        └── ...
```

#### Cloud Storage Locking Strategy
```yaml
# Custom locking mechanism using GCS
Locking Bucket: gs://pulumi-locks-saas-shared-vpc/
Lock Structure:
  - locks/dev.lock
  - locks/prod.lock
  
Lock Format:
  timestamp: 2025-10-01T10:30:00Z
  owner: gitlab-ci-job-12345
  user: devops@cequence.ai
  ttl: 3600  # 1 hour safety TTL
```

### 2. Secret Management (Vault Integration)

#### Vault Structure for Shared VPC
```yaml
# Vault KV v2 structure - minimal secrets for network infrastructure
secret/saas-shared-vpc/
├── global/
│   └── gcp-credentials/
│       ├── dev-host-project-sa-key
│       └── prod-host-project-sa-key
└── monitoring/
    └── network-monitoring-token
```

### 3. GitLab CI/CD Pipeline Architecture

#### Pipeline Structure for Foundation Infrastructure
```yaml
# .gitlab-ci.yml for self-hosted Pulumi (foundation network)
stages:
  - validate
  - preview
  - deploy
  - verify

variables:
  PULUMI_BACKEND_URL: "gs://pulumi-state-saas-shared-vpc"
  PULUMI_CONFIG_PASSPHRASE: ""  # Using Vault for encryption
  GOOGLE_CLOUD_PROJECT: "${GCP_PROJECT_ID}"
  VAULT_ADDR: "${VAULT_URL}"

before_script:
  - export PULUMI_ACCESS_TOKEN=""  # Disabled for self-hosted
  - vault auth -method=jwt role=pulumi-network-ci
  - export GOOGLE_CREDENTIALS=$(vault kv get -field=key secret/saas-shared-vpc/global/gcp-credentials/${CI_ENVIRONMENT_NAME}-host-project-sa-key)
```

## Migration Implementation Plan

### Phase 1: Infrastructure Setup (Week 1)

#### 1.1 GCP Infrastructure for State Management
```bash
# Create GCS bucket for state storage
gsutil mb -p saas-infrastructure-mgmt gs://pulumi-state-saas-shared-vpc
gsutil versioning set on gs://pulumi-state-saas-shared-vpc
gsutil iam ch serviceAccount:pulumi-ci@saas-infrastructure-mgmt.iam.gserviceaccount.com:objectAdmin gs://pulumi-state-saas-shared-vpc

# Create separate bucket for locking mechanism
gsutil mb -p saas-infrastructure-mgmt gs://pulumi-locks-saas-shared-vpc
gsutil lifecycle set lifecycle-config.json gs://pulumi-locks-saas-shared-vpc

# lifecycle-config.json for automatic lock cleanup
cat > lifecycle-config.json <<EOF
{
  "rule": [
    {
      "action": {"type": "Delete"},
      "condition": {"age": 1}
    }
  ]
}
EOF
```

#### 1.2 Vault Setup for Network Infrastructure
```bash
# Enable KV v2 secrets engine for network secrets
vault secrets enable -path=secret kv-v2

# Create JWT auth method for GitLab CI
vault auth enable jwt
vault write auth/jwt/config \
  jwks_url="https://gitlab.com/-/jwks" \
  bound_issuer="gitlab.com"

# Create policy for network infrastructure CI access
vault policy write pulumi-network-policy - <<EOF
path "secret/data/saas-shared-vpc/*" {
  capabilities = ["read"]
}
path "secret/metadata/saas-shared-vpc/*" {
  capabilities = ["read"]
}
EOF

# Create role for network infrastructure CI
vault write auth/jwt/role/pulumi-network-ci \
  bound_audiences="https://gitlab.com" \
  bound_claims='{"project_id":"NETWORK_PROJECT_ID","ref_type":"branch","ref":"main"}' \
  user_claim="user_email" \
  role_type="jwt" \
  policies="pulumi-network-policy" \
  ttl=2h
```

#### 1.3 Custom Locking Mechanism
```bash
#!/bin/bash
# scripts/gcs-lock.sh - Custom locking for GCS backend

STACK_NAME=$1
ACTION=$2  # acquire or release
LOCK_BUCKET="gs://pulumi-locks-saas-shared-vpc"
LOCK_FILE="locks/${STACK_NAME}.lock"
TTL=3600  # 1 hour

acquire_lock() {
    local lock_content=$(cat <<EOF
{
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "owner": "${CI_JOB_ID:-local}",
  "user": "${CI_COMMIT_AUTHOR:-$(whoami)}",
  "ttl": $TTL,
  "stack": "$STACK_NAME"
}
EOF
)
    
    # Check if lock exists and is not expired
    if gsutil stat ${LOCK_BUCKET}/${LOCK_FILE} 2>/dev/null; then
        echo "Lock exists, checking expiration..."
        local lock_time=$(gsutil cat ${LOCK_BUCKET}/${LOCK_FILE} | jq -r '.timestamp')
        local current_time=$(date -u +%Y-%m-%dT%H:%M:%SZ)
        
        # If lock is older than TTL, consider it expired
        if [[ $(date -d "$current_time" +%s) -gt $(($(date -d "$lock_time" +%s) + TTL)) ]]; then
            echo "Lock expired, acquiring..."
        else
            echo "❌ Stack $STACK_NAME is locked by another process"
            exit 1
        fi
    fi
    
    # Acquire lock
    echo "$lock_content" | gsutil cp - ${LOCK_BUCKET}/${LOCK_FILE}
    echo "✅ Lock acquired for stack $STACK_NAME"
}

release_lock() {
    gsutil rm ${LOCK_BUCKET}/${LOCK_FILE} 2>/dev/null || true
    echo "🔓 Lock released for stack $STACK_NAME"
}

case $ACTION in
    acquire) acquire_lock ;;
    release) release_lock ;;
    *) echo "Usage: $0 <stack_name> <acquire|release>" ;;
esac
```

### Phase 2: Secret Migration (Week 1-2)

#### 2.1 Extract and Migrate Service Account Keys
```bash
# Extract GCP service account keys for host projects
for env in dev prod; do
  PROJECT_ID=$(gcloud config get-value project --configuration=$env-host)
  
  # Create dedicated service account for Pulumi operations
  gcloud iam service-accounts create pulumi-network-sa \
    --display-name="Pulumi Network Management" \
    --project=$PROJECT_ID
  
  # Grant necessary permissions
  gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:pulumi-network-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
    --role="roles/compute.networkAdmin"
  
  gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:pulumi-network-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
    --role="roles/compute.securityAdmin"
  
  # Create and store key in Vault
  gcloud iam service-accounts keys create key.json \
    --iam-account=pulumi-network-sa@${PROJECT_ID}.iam.gserviceaccount.com \
    --project=$PROJECT_ID
  
  vault kv put secret/saas-shared-vpc/global/gcp-credentials/${env}-host-project-sa-key \
    key="$(cat key.json | base64 -w 0)"
  
  rm key.json
done
```

### Phase 3: GitLab CI/CD Pipeline Development (Week 2-3)

#### 3.1 Self-Hosted Pipeline Configuration
```yaml
# .gitlab-ci.yml for shared-vpc
include:
  - project: 'cequence/saas/infrastructure/tenants/cicd-templates'  
    ref: master
    file: '.self-hosted-pulumi-network.yml'

variables:
  PULUMI_BACKEND_URL: "gs://pulumi-state-saas-shared-vpc"
  PULUMI_CONFIG_PASSPHRASE: ""
  VAULT_ADDR: "${VAULT_URL}"
  LOCK_TIMEOUT: "3600"  # 1 hour

stages:
  - validate
  - preview
  - deploy
  - verify

.vault-auth: &vault-auth
  - apk add --no-cache vault jq
  - export VAULT_TOKEN=$(vault write -field=token auth/jwt/login role=pulumi-network-ci jwt=$CI_JOB_JWT)

.pulumi-setup: &pulumi-setup
  - curl -fsSL https://get.pulumi.com | sh
  - export PATH=$PATH:$HOME/.pulumi/bin
  - export GOOGLE_CREDENTIALS=$(vault kv get -field=key secret/saas-shared-vpc/global/gcp-credentials/${CI_ENVIRONMENT_NAME}-host-project-sa-key | base64 -d)

.lock-management: &lock-management
  - chmod +x scripts/gcs-lock.sh

validate-network:
  stage: validate
  image: google/cloud-sdk:alpine
  before_script:
    - *vault-auth
    - *pulumi-setup
    - *lock-management
  script:
    - ./scripts/gcs-lock.sh ${CI_ENVIRONMENT_NAME} acquire
    - pulumi stack select --create ${CI_ENVIRONMENT_NAME}
    - pulumi preview --diff | tee validate-${CI_ENVIRONMENT_NAME}.log
    - ./scripts/gcs-lock.sh ${CI_ENVIRONMENT_NAME} release
  after_script:
    - ./scripts/gcs-lock.sh ${CI_ENVIRONMENT_NAME} release || true
  artifacts:
    paths:
      - validate-*.log
    expire_in: 7 days
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  parallel:
    matrix:
      - CI_ENVIRONMENT_NAME: [dev, prod]

preview-network:
  stage: preview
  image: google/cloud-sdk:alpine
  before_script:
    - *vault-auth
    - *pulumi-setup
    - *lock-management
  script:
    - ./scripts/gcs-lock.sh ${CI_ENVIRONMENT_NAME} acquire
    - pulumi stack select --create ${CI_ENVIRONMENT_NAME}
    - pulumi preview --diff --detailed-diff | tee preview-${CI_ENVIRONMENT_NAME}.log
    - ./scripts/network-analysis.sh ${CI_ENVIRONMENT_NAME} preview-${CI_ENVIRONMENT_NAME}.log
    - ./scripts/gcs-lock.sh ${CI_ENVIRONMENT_NAME} release
  after_script:
    - ./scripts/gcs-lock.sh ${CI_ENVIRONMENT_NAME} release || true
  artifacts:
    reports:
      junit: network-analysis-*.xml
    paths:
      - preview-*.log
      - network-analysis-*.json
    expire_in: 14 days
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  parallel:
    matrix:
      - CI_ENVIRONMENT_NAME: [dev, prod]

deploy-network-dev:
  stage: deploy
  image: google/cloud-sdk:alpine
  before_script:
    - *vault-auth
    - *pulumi-setup
    - *lock-management
  script:
    - ./scripts/gcs-lock.sh dev acquire
    - pulumi stack select --create dev
    - pulumi up --yes | tee deploy-dev.log
    - ./scripts/network-verification.sh dev
    - ./scripts/gcs-lock.sh dev release
  after_script:
    - ./scripts/gcs-lock.sh dev release || true
  artifacts:
    paths:
      - deploy-*.log
      - network-verification-*.json
    expire_in: 30 days
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  environment:
    name: dev
    deployment_tier: development

deploy-network-prod:
  stage: deploy
  image: google/cloud-sdk:alpine
  before_script:
    - *vault-auth
    - *pulumi-setup
    - *lock-management
  script:
    - ./scripts/gcs-lock.sh prod acquire
    - pulumi stack select --create prod
    - pulumi up --yes | tee deploy-prod.log
    - ./scripts/network-verification.sh prod
    - ./scripts/gcs-lock.sh prod release
  after_script:
    - ./scripts/gcs-lock.sh prod release || true
  artifacts:
    paths:
      - deploy-*.log
      - network-verification-*.json
    expire_in: 30 days
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual  # Production requires manual approval
  environment:
    name: prod
    deployment_tier: production
  needs: ["deploy-network-dev"]

verify-network-connectivity:
  stage: verify
  image: google/cloud-sdk:alpine
  before_script:
    - *vault-auth
    - *pulumi-setup
  script:
    - ./scripts/cross-project-connectivity-test.sh ${CI_ENVIRONMENT_NAME}
    - ./scripts/regional-subnet-validation.sh ${CI_ENVIRONMENT_NAME}
  artifacts:
    paths:
      - connectivity-test-*.log
      - subnet-validation-*.json
    expire_in: 7 days
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  parallel:
    matrix:
      - CI_ENVIRONMENT_NAME: [dev, prod]
  needs: ["deploy-network-dev", "deploy-network-prod"]
```

#### 3.2 Custom Network Monitoring and Validation
```bash
#!/bin/bash
# scripts/network-verification.sh

STACK_NAME=$1
LOG_FILE="deploy-${STACK_NAME}.log"

echo "🔍 Starting network verification for $STACK_NAME"

# Get stack outputs
VPC_NAME=$(pulumi stack output networkName --stack $STACK_NAME)
SUBNETS=$(pulumi stack output subnetworkRangesByRegion --stack $STACK_NAME --json)

# Verify VPC exists
if gcloud compute networks describe $VPC_NAME --project=$(pulumi stack output project --stack $STACK_NAME) >/dev/null 2>&1; then
    echo "✅ VPC $VPC_NAME exists and is accessible"
else
    echo "❌ VPC $VPC_NAME not found or inaccessible"
    exit 1
fi

# Verify regional subnets
REGION_COUNT=0
echo "$SUBNETS" | jq -r 'keys[]' | while read region; do
    SUBNET_CIDR=$(echo "$SUBNETS" | jq -r ".[\"$region\"]")
    
    if gcloud compute networks subnets describe $region --region=$region --project=$(pulumi stack output project --stack $STACK_NAME) >/dev/null 2>&1; then
        echo "✅ Subnet in region $region exists with CIDR $SUBNET_CIDR"
        REGION_COUNT=$((REGION_COUNT + 1))
    else
        echo "❌ Subnet in region $region not found"
        exit 1
    fi
done

# Verify cross-project IAM permissions
SERVICE_PROJECTS=$(pulumi config get serviceProjects --stack $STACK_NAME)
echo "$SERVICE_PROJECTS" | jq -r '.[]' | while read service_project; do
    if gcloud compute shared-vpc get-host-project $service_project >/dev/null 2>&1; then
        echo "✅ Service project $service_project is attached to shared VPC"
    else
        echo "❌ Service project $service_project not properly attached"
        exit 1
    fi
done

# Generate network verification report
cat > network-verification-${STACK_NAME}.json <<EOF
{
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "stack": "$STACK_NAME",
  "vpc_name": "$VPC_NAME",
  "regions_validated": $REGION_COUNT,
  "service_projects_attached": $(echo "$SERVICE_PROJECTS" | jq length),
  "status": "success"
}
EOF

echo "🎉 Network verification completed successfully for $STACK_NAME"
```

### Phase 4: State Migration (Week 3-4)

#### 4.1 Foundation-First Migration Strategy
```bash
#!/bin/bash
# migrate-shared-vpc.sh - Critical foundation migration

echo "🚨 CRITICAL: Migrating foundation network infrastructure"
echo "⚠️  All dependent GCP infrastructure will be affected"

STACKS=("dev" "prod")

for stack in "${STACKS[@]}"; do
  echo "🔄 Starting migration for shared-vpc $stack"
  
  # Pre-migration network state backup
  pulumi stack select cequence/saas-shared-vpc/$stack
  pulumi stack export --file ${stack}-vpc-state-backup.json
  
  # Document current network configuration
  gcloud compute networks list --project=$(pulumi config get gcp:project) > ${stack}-networks-pre-migration.txt
  gcloud compute networks subnets list --project=$(pulumi config get gcp:project) > ${stack}-subnets-pre-migration.txt
  
  # Export network topology for validation
  ./scripts/export-network-topology.sh $stack > ${stack}-topology-backup.json
  
  # Configure new backend
  pulumi logout
  export PULUMI_BACKEND_URL="gs://pulumi-state-saas-shared-vpc"
  
  # Acquire migration lock
  ./scripts/gcs-lock.sh $stack acquire
  
  # Create new stack and import state
  pulumi stack init $stack
  pulumi stack import --file ${stack}-vpc-state-backup.json
  
  # Critical validation - network integrity check
  echo "🔍 Performing critical network integrity validation..."
  pulumi preview --diff > ${stack}-post-migration-diff.log
  
  if [ $? -eq 0 ] && [ ! -s ${stack}-post-migration-diff.log ]; then
    echo "✅ Migration successful for shared-vpc $stack - no resource drift detected"
    
    # Additional network connectivity validation
    ./scripts/network-verification.sh $stack
    
    if [ $? -eq 0 ]; then
      echo "✅ Network connectivity validation passed for $stack"
    else
      echo "❌ Network connectivity validation failed for $stack"
      ./scripts/rollback-shared-vpc.sh $stack
      exit 1
    fi
  else
    echo "❌ Migration validation failed for shared-vpc $stack"
    cat ${stack}-post-migration-diff.log
    ./scripts/rollback-shared-vpc.sh $stack
    exit 1
  fi
  
  # Release lock
  ./scripts/gcs-lock.sh $stack release
  
  echo "✅ Shared VPC $stack migration completed successfully"
  
  # Wait between environments for safety
  if [ "$stack" = "dev" ]; then
    echo "⏳ Waiting 30 minutes before production migration..."
    sleep 1800
  fi
done

echo "🎉 All shared VPC environments migrated successfully"
echo "📋 Next step: Coordinate with clusters repository team for dependent infrastructure migration"
```

#### 4.2 Network Topology Export and Validation
```bash
#!/bin/bash
# scripts/export-network-topology.sh

STACK_NAME=$1
PROJECT_ID=$(pulumi config get gcp:project --stack $STACK_NAME)

# Export complete network topology
cat > ${STACK_NAME}-topology-backup.json <<EOF
{
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "stack": "$STACK_NAME",
  "project": "$PROJECT_ID",
  "networks": $(gcloud compute networks list --project=$PROJECT_ID --format=json),
  "subnets": $(gcloud compute networks subnets list --project=$PROJECT_ID --format=json),
  "firewall_rules": $(gcloud compute firewall-rules list --project=$PROJECT_ID --format=json),
  "routes": $(gcloud compute routes list --project=$PROJECT_ID --format=json),
  "nat_gateways": $(gcloud compute routers list --project=$PROJECT_ID --format=json)
}
EOF

echo "Network topology exported for $STACK_NAME"
```

### Phase 5: RBAC and Security Implementation (Week 4-5)

#### 5.1 GitLab-Based RBAC for Network Infrastructure
```yaml
# Enhanced RBAC for critical network infrastructure
.deploy-network-production:
  rules:
    - if: $CI_COMMIT_BRANCH == "main" 
      && $CI_COMMIT_AUTHOR =~ /@cequence\.ai$/ 
      && $CI_COMMIT_AUTHOR =~ /(network-admin|infrastructure-lead)@/
      when: manual
      allow_failure: false

.emergency-network-access:
  rules:
    - if: $CI_COMMIT_MESSAGE =~ /\[EMERGENCY\]/
      && $CI_COMMIT_AUTHOR =~ /(cto|head-of-infrastructure)@cequence\.ai$/
      when: manual

# Network-specific validation gates
validate-network-changes:
  script:
    - ./scripts/validate-cidr-changes.sh
    - ./scripts/validate-cross-project-impact.sh
    - ./scripts/validate-regional-impact.sh
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      changes:
        - "src/**/*"
        - "Pulumi.*.yaml"
```

#### 5.2 Network Change Audit and Compliance
```bash
#!/bin/bash
# scripts/network-audit-logger.sh

OPERATION=$1
STACK=$2
USER=$3
TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%SZ)

# Enhanced audit logging for network changes
AUDIT_ENTRY=$(cat <<EOF
{
  "timestamp": "$TIMESTAMP",
  "operation": "$OPERATION",
  "stack": "$STACK",
  "user": "$USER",
  "infrastructure_type": "network_foundation",
  "criticality": "high",
  "affected_dependencies": ["clusters", "gke-services"],
  "compliance": {
    "data_residency_impact": true,
    "security_boundary_changes": true
  },
  "metadata": {
    "job_id": "$CI_JOB_ID",
    "pipeline_id": "$CI_PIPELINE_ID",
    "commit_sha": "$CI_COMMIT_SHA",
    "vpc_name": "$(pulumi stack output networkName --stack $STACK)",
    "regions_affected": "$(pulumi stack output subnetworkRangesByRegion --stack $STACK --json | jq -r 'keys | length')"
  }
}
EOF
)

# Log to multiple audit systems
echo "$AUDIT_ENTRY" >> /var/log/pulumi-network-audit.log

# Send to centralized audit system
curl -X POST "${NETWORK_AUDIT_WEBHOOK}" \
  -H "Content-Type: application/json" \
  -H "X-Audit-Type: network-infrastructure" \
  -d "$AUDIT_ENTRY"

# Send to security monitoring
curl -X POST "${SECURITY_AUDIT_WEBHOOK}" \
  -H "Content-Type: application/json" \
  -H "X-Security-Event: infrastructure-change" \
  -d "$AUDIT_ENTRY"
```

### Phase 6: Testing and Validation (Week 5)

#### 6.1 Comprehensive Network Validation
```bash
#!/bin/bash
# scripts/comprehensive-network-test.sh

STACK_NAME=$1
echo "🧪 Starting comprehensive network testing for $STACK_NAME"

# Test 1: VPC and subnet connectivity
./scripts/test-vpc-connectivity.sh $STACK_NAME

# Test 2: Cross-project IAM permissions
./scripts/test-cross-project-iam.sh $STACK_NAME

# Test 3: Regional subnet accessibility
./scripts/test-regional-subnets.sh $STACK_NAME

# Test 4: NAT gateway functionality
./scripts/test-nat-gateways.sh $STACK_NAME

# Test 5: Load balancer subnet assignments
./scripts/test-lb-subnets.sh $STACK_NAME

# Test 6: Service project attachment validation
./scripts/test-service-project-attachment.sh $STACK_NAME

# Generate comprehensive test report
cat > network-test-report-${STACK_NAME}.json <<EOF
{
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "stack": "$STACK_NAME",
  "test_results": {
    "vpc_connectivity": "$(cat vpc-test-${STACK_NAME}.status)",
    "cross_project_iam": "$(cat iam-test-${STACK_NAME}.status)",
    "regional_subnets": "$(cat subnet-test-${STACK_NAME}.status)",
    "nat_gateways": "$(cat nat-test-${STACK_NAME}.status)",
    "load_balancer_subnets": "$(cat lb-test-${STACK_NAME}.status)",
    "service_projects": "$(cat service-project-test-${STACK_NAME}.status)"
  },
  "overall_status": "success"
}
EOF

echo "✅ Comprehensive network testing completed for $STACK_NAME"
```

#### 6.2 Rollback Procedures for Network Infrastructure
```bash
#!/bin/bash
# scripts/rollback-shared-vpc.sh - EMERGENCY ROLLBACK

STACK_NAME=$1
BACKUP_FILE="${STACK_NAME}-vpc-state-backup.json"

echo "🚨 EMERGENCY ROLLBACK: shared-vpc $STACK_NAME"
echo "⚠️  This will affect ALL dependent GCP infrastructure"

# Immediate lock acquisition
./scripts/gcs-lock.sh $STACK_NAME acquire

# Stop any running operations
pulumi cancel --stack $STACK_NAME || true

# Restore from Pulumi Cloud backup
pulumi logout
pulumi login
pulumi stack select cequence/saas-shared-vpc/$STACK_NAME
pulumi stack import --file $BACKUP_FILE

# Verify rollback
pulumi preview --diff --stack $STACK_NAME

# Network connectivity validation
./scripts/network-verification.sh $STACK_NAME

if [ $? -eq 0 ]; then
  echo "✅ Rollback successful for shared-vpc $STACK_NAME"
else
  echo "❌ Rollback validation failed - manual intervention required"
  exit 1
fi

# Release lock
./scripts/gcs-lock.sh $STACK_NAME release

echo "🔄 Rollback completed for shared-vpc $STACK_NAME"
echo "📞 Notify dependent teams: clusters repository team must be informed"
```

## Implementation Checklist

### Pre-Migration Critical Setup
- [ ] GCS buckets created with proper versioning and IAM
- [ ] Custom locking mechanism implemented and tested
- [ ] Vault configured with network-specific JWT auth
- [ ] Service account keys created for host projects
- [ ] Network topology backup scripts tested

### Migration Execution (FOUNDATION FIRST)
- [ ] **CRITICAL**: Coordinate with all dependent teams before migration
- [ ] Complete network state backup for both environments
- [ ] Migrate dev environment with full validation
- [ ] 30-minute monitoring period before prod migration
- [ ] Migrate prod environment with enhanced monitoring
- [ ] **CRITICAL**: Validate all dependent clusters can still deploy

### Post-Migration Validation
- [ ] All 40+ regional subnets operational with correct CIDR blocks
- [ ] Cross-project IAM permissions functional
- [ ] Service project attachments validated
- [ ] NAT gateways operational in all regions
- [ ] Load balancer subnets accessible
- [ ] **CRITICAL**: Clusters repository team confirms network connectivity

## Risk Mitigation

### Critical Network Risks
1. **CIDR Block Changes**: Automated validation prevents subnet CIDR modifications
2. **Cross-Project Permissions**: Pre/post migration IAM binding verification
3. **Regional Connectivity**: Comprehensive regional subnet testing
4. **Dependent Service Impact**: Coordination with clusters repository team
5. **Lock Contention**: Custom locking with TTL and emergency override

### Emergency Procedures
1. **Immediate Rollback**: Automated rollback to Pulumi Cloud state
2. **Network Isolation**: Emergency procedures to isolate affected regions
3. **Cross-Team Communication**: Automated notifications to dependent teams
4. **Escalation Path**: Direct escalation to network architecture team

## Success Metrics

### Technical Success Metrics
- CIDR block preservation: 100% (all 40+ regions)
- Cross-project IAM functionality: 100%
- Regional connectivity: 100% (all enabled regions)
- Service project attachment: 100%
- Migration time per environment: <2 hours

### Operational Success Metrics
- Zero network-related incidents during migration
- All dependent clusters maintain connectivity
- No customer-impacting network disruptions
- Successful coordination with dependent teams
- Emergency rollback procedures validated

This comprehensive migration plan ensures the critical shared VPC foundation is migrated safely while maintaining connectivity for all dependent GCP infrastructure.