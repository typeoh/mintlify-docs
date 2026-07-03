# Source: https://rasa.com/docs/reference/changelogs/studio-version-migration-guide

On this page

This page contains information about changes between major versions and how you can migrate from one version to another.

## Rasa Studio v1.13.x → v1.14.x[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#rasa-studio-v113x--v114x 'Direct link to Rasa Studio v1.13.x → v1.14.x')

### What's New[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#whats-new 'Direct link to What's New')

Studio version 1.14 introduces **AWS IAM Database Authentication** for RDS connections, significantly enhancing your deployment's security posture.

### Before You Upgrade[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#before-you-upgrade 'Direct link to Before You Upgrade')

Review the following guide to enable IAM authentication in your environment. The guide covers both database IAM authentication (new in v1.14) and S3 storage setup (for those migrating from persistent volumes).

### Overview[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#overview 'Direct link to Overview')

This version allows you to replace static database passwords with secure, temporary IAM tokens for enhanced security.

**Note**: S3 storage with IAM has always been supported in Studio. This guide focuses on the new IAM database authentication feature. S3 setup steps are applicable only if you're moving to S3 storage or setting it up for the first time.

This guide explains what you need to do to enable IAM database authentication in your environment.

---

### ⚠️ Important: Model Retraining Required (Only When Moving to S3)[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#️-important-model-retraining-required-only-when-moving-to-s3 'Direct link to ⚠️ Important: Model Retraining Required (Only When Moving to S3)')

**If you are moving from persistent volumes to S3 for model storage, you will need to either copy your existing models to S3 or retrain all your assistants.**

#### When This Applies[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#when-this-applies 'Direct link to When This Applies')

- ✅ **Applies if**: You currently use persistent volumes and are moving to S3 storage
- ❌ **Does NOT apply if**: You already use S3 storage (with or without IAM)
- ❌ **Does NOT apply if**: You are only enabling IAM database authentication (not changing storage)

#### Why Retraining is Needed (When Moving to S3)[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#why-retraining-is-needed-when-moving-to-s3 'Direct link to Why Retraining is Needed (When Moving to S3)')

- Current models stored in persistent volumes will not be accessible after moving to S3
- This is a one-time requirement when you first move to S3 storage
- Your training data and assistant configurations are not affected and remain available
- **Alternative**: You can copy models from persistent volumes to S3 instead of retraining

#### What You Need to Do (If Moving to S3)[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#what-you-need-to-do-if-moving-to-s3 'Direct link to What You Need to Do (If Moving to S3)')

1. **Choose Your Approach**:
 - **Option A**: Copy existing models from persistent volumes to S3 (recommended to avoid retraining)
 - **Option B**: Retrain all active assistants (if you prefer to regenerate models)
2. **Plan Ahead**: Schedule time for either copying models or retraining all active assistants
3. **Test Environment**: Consider testing in a staging environment first

#### If You Already Use S3[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#if-you-already-use-s3 'Direct link to If You Already Use S3')

- **No Retraining Required**: Your existing S3-stored models will continue to work
- **No Impact**: Enabling IAM database authentication does not affect your models

---

### 🚀 How to Enable IAM Authentication[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#-how-to-enable-iam-authentication 'Direct link to 🚀 How to Enable IAM Authentication')

#### Step 1: Set Up AWS Infrastructure[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#step-1-set-up-aws-infrastructure 'Direct link to Step 1: Set Up AWS Infrastructure')

##### Create IAM Roles[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#create-iam-roles 'Direct link to Create IAM Roles')

You need to create IAM roles that your application can use:

1. **Create RDS IAM Role**:
 
 - Name: `your-app-rds-role`
 - Trust policy: Allow EKS service accounts to assume this role (see example below)
 - Attach policy for RDS database connection
2. **Create S3 IAM Role**:
 
 - Name: `your-app-s3-role`
 - Trust policy: Allow EKS service accounts to assume this role (see example below)
 - Attach policy for S3 bucket access

**Example Trust Policy** (for both roles, replace YOUR-OIDC-PROVIDER-ARN and YOUR-OIDC-PROVIDER-URL):

```
{  "Version": "2012-10-17",  "Statement": [    {      "Effect": "Allow",      "Principal": {        "Federated": "YOUR-OIDC-PROVIDER-ARN"      },      "Action": "sts:AssumeRoleWithWebIdentity",      "Condition": {        "StringEquals": {          "YOUR-OIDC-PROVIDER-URL:aud": "sts.amazonaws.com"        }      }    }  ]}
```

##### Required IAM Policies[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#required-iam-policies 'Direct link to Required IAM Policies')

**RDS Policy** (attach to your RDS role):

```
{  "Version": "2012-10-17",  "Statement": [    {      "Effect": "Allow",      "Action": ["rds-db:connect"],      "Resource": [        "arn:aws:rds-db:YOUR-REGION:YOUR-ACCOUNT-ID:dbuser:YOUR-DB-INSTANCE-ID/*"      ]    }  ]}
```

**To find your DB instance ID:**

- Go to AWS RDS console → Your database instance
- The instance ID is shown in the instance details (e.g., `mydb-instance-1`)
- Or use AWS CLI: `aws rds describe-db-instances --query 'DBInstances[*].DBInstanceIdentifier'`

**S3 Policy** (attach to your S3 role):

```
{  "Version": "2012-10-17",  "Statement": [    {      "Effect": "Allow",      "Action": [        "s3:GetObject",        "s3:PutObject",        "s3:DeleteObject",        "s3:ListBucket",        "s3:GetBucketLocation",        "s3:ListBucketMultipartUploads",        "s3:AbortMultipartUpload",        "s3:ListMultipartUploadParts"      ],      "Resource": [        "arn:aws:s3:::YOUR-BUCKET-NAME",        "arn:aws:s3:::YOUR-BUCKET-NAME/*"      ]    }  ]}
```

##### Configure EKS Service Accounts[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#configure-eks-service-accounts 'Direct link to Configure EKS Service Accounts')

**Service accounts are automatically created when you configure Helm chart annotations.**

1. **Create or Verify OIDC Provider**: Ensure your EKS cluster has an OIDC provider configured
 
 **Check if OIDC provider exists:**
 
 - Go to AWS IAM console → Identity providers
 - Look for your EKS cluster's OIDC provider URL
 
 **If it doesn't exist, create it:**
 
 **Option 1: Using AWS CLI**
 
 ```
    # Get your EKS cluster's OIDC issuer URLaws eks describe-cluster --name YOUR-CLUSTER-NAME --query "cluster.identity.oidc.issuer" --output text# Create the OIDC provider (replace with your actual issuer URL)aws iam create-open-id-connect-provider \  --url https://oidc.eks.YOUR-REGION.amazonaws.com/id/YOUR-OIDC-ID \  --client-id-list sts.amazonaws.com \  --thumbprint-list YOUR-THUMBPRINT
    ```
 
 **Option 2: Using AWS Console**
 
 - Go to IAM → Identity providers → Add provider
 - Select "OpenID Connect"
 - Provider URL: `https://oidc.eks.YOUR-REGION.amazonaws.com/id/YOUR-OIDC-ID`
 - Audience: `sts.amazonaws.com`
 - Click "Add provider"
2. **Configure Helm Chart Annotations**:
 
 - Add IAM role annotations to your Helm chart values for the following services
 - Service accounts will be automatically created with the IAM role ARNs
 
 **For Database (RDS IAM) access**, add annotations to:
 
 - Backend service
 - Event ingestion service
 - Database migration job
 
 **For S3 access**, add annotations to:
 
 - Rasa Model Service
 
 Example annotation format for database services:
 
 ```
    serviceAccount:  annotations:    eks.amazonaws.com/role-arn: arn:aws:iam::YOUR-ACCOUNT-ID:role/your-app-rds-role
    ```
 
 Example annotation format for S3 services:
 
 ```
    serviceAccount:  annotations:    eks.amazonaws.com/role-arn: arn:aws:iam::YOUR-ACCOUNT-ID:role/your-app-s3-role
    ```
 

#### Step 2: Set Up RDS for IAM Authentication[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#step-2-set-up-rds-for-iam-authentication 'Direct link to Step 2: Set Up RDS for IAM Authentication')

1. **Enable IAM Database Authentication** on your RDS instance:
 
 - Go to AWS RDS console
 - Select your RDS instance
 - Click "Modify"
 - Enable "IAM database authentication"
 - Apply changes (may require restart)
 
 For more information, see the [AWS documentation for modifying DB instances](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html) to enable IAM database authentication
 
2. **Create IAM Database User** (connect to your RDS instance as admin):
 
 ```
    -- Create IAM database userCREATE USER "studio_iam_user" WITH LOGIN;-- Grant rds_iam role to IAM user (required for IAM auth)-- Note: rds_iam is a built-in AWS RDS role available after enabling IAM authGRANT rds_iam TO "studio_iam_user";
    ```
 
3. **Grant Database Permissions**:
 
 ```
    -- Grant ALL privileges on database to IAM userGRANT ALL PRIVILEGES ON DATABASE "your_database_name" TO "studio_iam_user";-- Grant schema privilegesGRANT ALL ON SCHEMA "public" TO "studio_iam_user";-- Grant privileges on all existing tables and sequencesGRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA "public" TO "studio_iam_user";GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA "public" TO "studio_iam_user";-- Transfer ownership of all objects to IAM user (optional, if you want to change ownership)REASSIGN OWNED BY "your_original_db_user" TO "studio_iam_user";
    ```
 

**Note**: Replace `your_original_db_user` with your current database user and `your_database_name` with your actual database name.

#### Step 3: Set Up S3 Bucket[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#step-3-set-up-s3-bucket 'Direct link to Step 3: Set Up S3 Bucket')

1. **Create S3 Bucket**:
 
 - Go to AWS S3 console
 - Click "Create bucket"
 - **Bucket name**: `your-app-shared-storage-ENVIRONMENT` (must be globally unique)
 - **Region**: Choose your preferred region
 - **Versioning**: Enable versioning
 - **Default encryption**: Enable → Choose "AES-256"
 - **Block public access**: Enable all 4 options:
 - ✅ Block all public access
 - ✅ Block public access to buckets and objects granted through new public bucket or access point policies
 - ✅ Block public access to buckets and objects granted through any public bucket or access point policies
 - ✅ Block public access to buckets and objects granted through new ACLs and uploading objects with public ACLs
 - Click "Create bucket"
 
 For more information, see the [AWS documentation for creating S3 buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html).
 
2. **Configure Bucket Policy**:
 
 - In "Permissions" tab, find "Bucket policy"
 - Click "Edit" and add this policy (replace YOUR-BUCKET-NAME and YOUR-ACCOUNT-ID):
 
 ```
    {  "Version": "2012-10-17",  "Statement": [    {      "Sid": "AllowSharedIAMRoleAccess",      "Effect": "Allow",      "Principal": {        "AWS": "arn:aws:iam::YOUR-ACCOUNT-ID:role/your-app-s3-role"      },      "Action": [        "s3:GetObject",        "s3:PutObject",        "s3:DeleteObject",        "s3:ListBucket",        "s3:GetBucketLocation",        "s3:ListBucketMultipartUploads",        "s3:AbortMultipartUpload",        "s3:ListMultipartUploadParts"      ],      "Resource": [        "arn:aws:s3:::YOUR-BUCKET-NAME",        "arn:aws:s3:::YOUR-BUCKET-NAME/*"      ]    }  ]}
    ```
 

#### Step 4: Update Your Application Configuration[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#step-4-update-your-application-configuration 'Direct link to Step 4: Update Your Application Configuration')

Add these configuration values to your Helm chart:

##### Database IAM Configuration[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#database-iam-configuration 'Direct link to Database IAM Configuration')

Add to the `config.database` section of your Helm chart values:

```
config:  database:    useAwsIamAuth: "true"    awsRegion: "your-aws-region"    iamDbUsername: "studio_iam_user"
```

**These variables apply to:**

- Backend service
- Event ingestion service
- Database migration job

##### S3 Configuration (for Rasa Pro)[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#s3-configuration-for-rasa-pro 'Direct link to S3 Configuration (for Rasa Pro)')

Add to the `rasa.rasa.overrideEnv` section of your Helm chart values:

```
rasa:  rasa:    overrideEnv:      - name: "BUCKET_NAME"        value: "your-app-shared-storage-ENVIRONMENT"      - name: "AWS_DEFAULT_REGION"        value: "your-aws-region"      - name: "RASA_REMOTE_STORAGE"        value: "aws"
```

**These variables apply to:**

- Rasa Model Service

#### Step 5: Deploy and Test[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#step-5-deploy-and-test 'Direct link to Step 5: Deploy and Test')

1. **Deploy** your updated application
2. **Verify** database connections are working
3. **Retrain** your assistants (only if you're moving from persistent volumes to S3)

## Rasa Studio v1.12.x → v1.13.x[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#rasa-studio-v112x--v113x 'Direct link to Rasa Studio v1.12.x → v1.13.x')

### What's New[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#whats-new-1 'Direct link to What's New')

We've made important improvements to Rasa Studio's database migrations:

- No more `superuser` required: In earlier versions, certain database migrations required a user with superuser privileges. This is no longer necessary. All migrations can now be completed using a standard database user.

### Before You Upgrade[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#before-you-upgrade-1 'Direct link to Before You Upgrade')

If you're upgrading from a version before `v1.13.x`, please follow the below steps.

#### Step-by-Step Upgrade Instructions[​](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#step-by-step-upgrade-instructions 'Direct link to Step-by-Step Upgrade Instructions')

1. **Upgrade to `v1.12.7` First**
 
 This ensures that all necessary database migrations are applied before moving to the `1.13.x` version
 
2. **Mark Migrations as Complete**
 
 After upgrading to `v1.12.7`, run the following SQL command on your Studio database:
 
 ```
    insert into public._prisma_migrations (id,checksum,finished_at,migration_name,started_at,applied_steps_count)values ('08eb97ec-85fa-4578-921e-091d50c4a816','c0993f05c8c4021b096d2d8c78d7f3977e81388ae36e860387eddb2c3553a65b',now(),'000000000000_squashed_migrations',now(),1);
    ```
 
 This tells the system the earlier migrations are already applied, so they won't run again.
 
3. **Upgrade to `v1.13.x` or Later**
 
 After completing the steps above, you're ready to upgrade to the latest version of Rasa Studio.
 

- [Rasa Studio v1.13.x → v1.14.x](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#rasa-studio-v113x--v114x)
 - [What's New](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#whats-new)
 - [Before You Upgrade](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#before-you-upgrade)
 - [Overview](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#overview)
 - [⚠️ Important: Model Retraining Required (Only When Moving to S3)](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#️-important-model-retraining-required-only-when-moving-to-s3)
 - [🚀 How to Enable IAM Authentication](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#-how-to-enable-iam-authentication)
- [Rasa Studio v1.12.x → v1.13.x](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#rasa-studio-v112x--v113x)
 - [What's New](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#whats-new-1)
 - [Before You Upgrade](https://rasa.com/docs/reference/changelogs/studio-version-migration-guide/#before-you-upgrade-1)