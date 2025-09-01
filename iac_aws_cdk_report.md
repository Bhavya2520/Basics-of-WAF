# Infrastructure as Code (IaC) using AWS CDK Stack



---

## Executive Summary

This report provides a comprehensive analysis of Infrastructure as Code (IaC) implementation using AWS Cloud Development Kit (CDK) Stack. The document covers fundamental concepts, practical implementation strategies, code examples, and best practices for deploying and managing cloud infrastructure programmatically using AWS CDK.

AWS CDK enables developers to define cloud infrastructure using familiar programming languages, providing type safety, IDE support, and powerful abstraction capabilities that traditional template-based approaches lack.

---

## 1. Introduction to Infrastructure as Code (IaC)

### 1.1 Definition and Core Concepts

Infrastructure as Code (IaC) is the practice of managing and provisioning computing infrastructure through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools. IaC treats infrastructure the same way developers treat application code.

### 1.2 Benefits of IaC

#### 1.2.1 Consistency and Reproducibility
- Eliminates configuration drift between environments
- Ensures identical infrastructure across development, staging, and production
- Reduces human error in manual deployments
- Enables reliable disaster recovery procedures

#### 1.2.2 Version Control and Collaboration
- Infrastructure definitions stored in version control systems
- Track changes and maintain audit trails
- Enable collaborative infrastructure development
- Facilitate code reviews for infrastructure changes

#### 1.2.3 Automation and Efficiency
- Automated deployment and scaling processes
- Faster provisioning compared to manual processes
- Integration with CI/CD pipelines
- Reduced operational overhead

#### 1.2.4 Cost Management and Optimization
- Predictable resource provisioning
- Automated resource cleanup and optimization
- Environment-specific scaling configurations
- Better cost tracking and allocation

### 1.3 IaC Approaches

#### 1.3.1 Declarative vs Imperative
- **Declarative:** Define desired end state (What you want)
- **Imperative:** Define specific steps to achieve state (How to get there)
- AWS CDK supports both approaches through high-level constructs

#### 1.3.2 Mutable vs Immutable Infrastructure
- **Mutable:** Update existing resources in place
- **Immutable:** Replace entire resources with new versions
- CDK supports both patterns based on resource types

---

## 2. AWS Cloud Development Kit (CDK) Overview

### 2.1 Introduction to AWS CDK

AWS CDK is an open-source software development framework for defining cloud infrastructure in code and provisioning it through AWS CloudFormation. CDK allows developers to harness the expressive power of programming languages to define reusable cloud components.

### 2.2 CDK Architecture Components

#### 2.2.1 Core Components
- **App:** Root of CDK application tree
- **Stack:** Unit of deployment in CDK
- **Construct:** Basic building block of CDK applications
- **Resource:** AWS resources like EC2 instances, S3 buckets

#### 2.2.2 Construct Levels
- **Level 1 (L1) Constructs:** Direct CloudFormation resource mappings
- **Level 2 (L2) Constructs:** Higher-level abstractions with sensible defaults
- **Level 3 (L3) Constructs:** Patterns that combine multiple resources

### 2.3 Supported Programming Languages

#### 2.3.1 Primary Languages
- **TypeScript:** Native CDK language with full feature support
- **Python:** Popular choice for data science and automation teams
- **Java:** Enterprise-friendly with strong typing
- **C#/.NET:** Microsoft ecosystem integration
- **Go:** High-performance applications (experimental)

#### 2.3.2 Language Considerations
- TypeScript offers the most comprehensive documentation
- Python provides excellent readability and rapid development
- Java ensures enterprise-grade type safety
- All languages compile to CloudFormation templates

---

## 3. CDK Installation and Environment Setup

### 3.1 Prerequisites

#### 3.1.1 Required Tools
- Node.js (version 14.x or later)
- AWS CLI configured with appropriate permissions
- Git for version control
- IDE with language-specific extensions

#### 3.1.2 AWS Account Configuration
```bash
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS credentials
aws configure
```

### 3.2 CDK Installation

#### 3.2.1 Global CDK Installation
```bash
# Install CDK CLI globally
npm install -g aws-cdk

# Verify installation
cdk --version

# Bootstrap CDK environment
cdk bootstrap aws://ACCOUNT-NUMBER/REGION
```

#### 3.2.2 Project Initialization
```bash
# Create new CDK project
mkdir my-cdk-project
cd my-cdk-project

# Initialize CDK app (TypeScript)
cdk init app --language typescript

# Initialize CDK app (Python)
cdk init app --language python
```

### 3.3 Project Structure

#### 3.3.1 TypeScript Project Structure
```
my-cdk-project/
├── bin/
│   └── my-cdk-project.ts        # App entry point
├── lib/
│   └── my-cdk-project-stack.ts  # Stack definition
├── test/
│   └── my-cdk-project.test.ts   # Unit tests
├── cdk.json                     # CDK configuration
├── package.json                 # Node.js dependencies
└── tsconfig.json               # TypeScript configuration
```

#### 3.3.2 Python Project Structure
```
my-cdk-project/
├── app.py                      # App entry point
├── my_cdk_project/
│   ├── __init__.py
│   └── my_cdk_project_stack.py # Stack definition
├── tests/
├── requirements.txt            # Python dependencies
└── cdk.json                   # CDK configuration
```

---

## 4. CDK Stack Development

### 4.1 Basic Stack Creation

#### 4.1.1 TypeScript Stack Example
```typescript
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as s3 from 'aws-cdk-lib/aws-s3';
import { Construct } from 'constructs';

export class MyCdkProjectStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create VPC
    const vpc = new ec2.Vpc(this, 'MyVpc', {
      maxAzs: 2,
      cidr: '10.0.0.0/16',
      subnetConfiguration: [
        {
          cidrMask: 24,
          name: 'PublicSubnet',
          subnetType: ec2.SubnetType.PUBLIC,
        },
        {
          cidrMask: 24,
          name: 'PrivateSubnet',
          subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS,
        }
      ]
    });

    // Create S3 bucket
    const bucket = new s3.Bucket(this, 'MyBucket', {
      versioned: true,
      encryption: s3.BucketEncryption.S3_MANAGED,
      lifecycleRules: [{
        id: 'DeleteOldVersions',
        noncurrentVersionExpiration: cdk.Duration.days(90),
      }]
    });

    // Output bucket name
    new cdk.CfnOutput(this, 'BucketName', {
      value: bucket.bucketName,
      description: 'Name of the S3 bucket'
    });
  }
}
```

#### 4.1.2 Python Stack Example
```python
from aws_cdk import (
    Stack,
    aws_ec2 as ec2,
    aws_s3 as s3,
    Duration,
    CfnOutput
)
from constructs import Construct

class MyCdkProjectStack(Stack):
    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # Create VPC
        vpc = ec2.Vpc(self, "MyVpc",
            max_azs=2,
            cidr="10.0.0.0/16",
            subnet_configuration=[
                ec2.SubnetConfiguration(
                    cidr_mask=24,
                    name="PublicSubnet",
                    subnet_type=ec2.SubnetType.PUBLIC
                ),
                ec2.SubnetConfiguration(
                    cidr_mask=24,
                    name="PrivateSubnet",
                    subnet_type=ec2.SubnetType.PRIVATE_WITH_EGRESS
                )
            ]
        )

        # Create S3 bucket
        bucket = s3.Bucket(self, "MyBucket",
            versioned=True,
            encryption=s3.BucketEncryption.S3_MANAGED,
            lifecycle_rules=[s3.LifecycleRule(
                id="DeleteOldVersions",
                noncurrent_version_expiration=Duration.days(90)
            )]
        )

        # Output bucket name
        CfnOutput(self, "BucketName",
            value=bucket.bucket_name,
            description="Name of the S3 bucket"
        )
```

### 4.2 Advanced Stack Patterns

#### 4.2.1 Multi-Stack Architecture
```typescript
// networking-stack.ts
export class NetworkingStack extends cdk.Stack {
  public readonly vpc: ec2.Vpc;

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    this.vpc = new ec2.Vpc(this, 'Vpc', {
      maxAzs: 3,
      cidr: '10.0.0.0/16'
    });
  }
}

// application-stack.ts
export class ApplicationStack extends cdk.Stack {
  constructor(scope: Construct, id: string, vpc: ec2.Vpc, props?: cdk.StackProps) {
    super(scope, id, props);

    // Use VPC from networking stack
    const securityGroup = new ec2.SecurityGroup(this, 'AppSecurityGroup', {
      vpc: vpc,
      description: 'Security group for application'
    });
  }
}
```

#### 4.2.2 Environment-Specific Configuration
```typescript
interface EnvironmentConfig {
  instanceType: ec2.InstanceType;
  minCapacity: number;
  maxCapacity: number;
  enableLogging: boolean;
}

const environments: Record<string, EnvironmentConfig> = {
  dev: {
    instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
    minCapacity: 1,
    maxCapacity: 2,
    enableLogging: false
  },
  prod: {
    instanceType: ec2.InstanceType.of(ec2.InstanceClass.M5, ec2.InstanceSize.LARGE),
    minCapacity: 2,
    maxCapacity: 10,
    enableLogging: true
  }
};
```

---

## 5. CDK Constructs and Patterns

### 5.1 Common AWS Service Constructs

#### 5.1.1 Compute Services
```typescript
// EC2 Instance
const instance = new ec2.Instance(this, 'WebServer', {
  instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
  machineImage: ec2.MachineImage.latestAmazonLinux(),
  vpc: vpc,
  securityGroup: securityGroup
});

// Lambda Function
const lambdaFunction = new lambda.Function(this, 'MyFunction', {
  runtime: lambda.Runtime.PYTHON_3_9,
  handler: 'index.handler',
  code: lambda.Code.fromAsset('lambda'),
  environment: {
    BUCKET_NAME: bucket.bucketName
  }
});

// ECS Fargate Service
const cluster = new ecs.Cluster(this, 'Cluster', { vpc });
const taskDefinition = new ecs.FargateTaskDefinition(this, 'TaskDef');
const container = taskDefinition.addContainer('WebContainer', {
  image: ecs.ContainerImage.fromRegistry('nginx'),
  memoryLimitMiB: 256
});
```

#### 5.1.2 Database Services
```typescript
// RDS Database
const database = new rds.DatabaseInstance(this, 'Database', {
  engine: rds.DatabaseInstanceEngine.postgres({
    version: rds.PostgresEngineVersion.VER_13_7
  }),
  instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
  credentials: rds.Credentials.fromGeneratedSecret('postgres'),
  vpc: vpc,
  multiAz: false,
  allocatedStorage: 100,
  storageEncrypted: true,
  deletionProtection: false,
  backupRetention: cdk.Duration.days(7)
});

// DynamoDB Table
const table = new dynamodb.Table(this, 'Table', {
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  partitionKey: { name: 'id', type: dynamodb.AttributeType.STRING },
  encryption: dynamodb.TableEncryption.AWS_MANAGED,
  pointInTimeRecovery: true
});
```

#### 5.1.3 Storage and Content Delivery
```typescript
// S3 Bucket with CloudFront
const originBucket = new s3.Bucket(this, 'OriginBucket', {
  encryption: s3.BucketEncryption.S3_MANAGED,
  blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL
});

const distribution = new cloudfront.CloudFrontWebDistribution(this, 'Distribution', {
  originConfigs: [{
    s3OriginSource: {
      s3BucketSource: originBucket
    },
    behaviors: [{ isDefaultBehavior: true }]
  }]
});

// EFS File System
const fileSystem = new efs.FileSystem(this, 'FileSystem', {
  vpc: vpc,
  encrypted: true,
  lifecyclePolicy: efs.LifecyclePolicy.AFTER_30_DAYS,
  performanceMode: efs.PerformanceMode.GENERAL_PURPOSE
});
```

### 5.2 Custom Constructs

#### 5.2.1 Reusable Construct Pattern
```typescript
export interface WebApplicationProps {
  vpc: ec2.Vpc;
  domainName: string;
  certificateArn: string;
}

export class WebApplication extends Construct {
  public readonly loadBalancer: elbv2.ApplicationLoadBalancer;
  public readonly service: ecs.FargateService;

  constructor(scope: Construct, id: string, props: WebApplicationProps) {
    super(scope, id);

    // Application Load Balancer
    this.loadBalancer = new elbv2.ApplicationLoadBalancer(this, 'ALB', {
      vpc: props.vpc,
      internetFacing: true
    });

    // ECS Cluster and Service
    const cluster = new ecs.Cluster(this, 'Cluster', {
      vpc: props.vpc
    });

    const taskDefinition = new ecs.FargateTaskDefinition(this, 'TaskDef', {
      memoryLimitMiB: 512,
      cpu: 256
    });

    taskDefinition.addContainer('WebContainer', {
      image: ecs.ContainerImage.fromRegistry('nginx:latest'),
      portMappings: [{ containerPort: 80 }]
    });

    this.service = new ecs.FargateService(this, 'Service', {
      cluster,
      taskDefinition,
      desiredCount: 2
    });

    // Target Group and Listener
    const targetGroup = new elbv2.ApplicationTargetGroup(this, 'TargetGroup', {
      vpc: props.vpc,
      protocol: elbv2.ApplicationProtocol.HTTP,
      targets: [this.service]
    });

    this.loadBalancer.addListener('Listener', {
      port: 443,
      certificates: [elbv2.ListenerCertificate.fromArn(props.certificateArn)],
      defaultTargetGroups: [targetGroup]
    });
  }
}
```

---

## 6. CDK Deployment and Management

### 6.1 CDK CLI Commands

#### 6.1.1 Basic Commands
```bash
# Synthesize CloudFormation template
cdk synth

# Compare deployed stack with current state
cdk diff

# Deploy stack
cdk deploy

# Deploy specific stack
cdk deploy MyStackName

# Deploy all stacks
cdk deploy --all

# Destroy stack
cdk destroy

# List all stacks
cdk list
```

#### 6.1.2 Advanced Commands
```bash
# Deploy with approval for security changes
cdk deploy --require-approval never

# Deploy with parameters
cdk deploy --parameters MyStack:parameterName=value

# Deploy with context values
cdk deploy -c key=value

# Watch mode for development
cdk deploy --hotswap

# Generate CloudFormation template
cdk synth > template.yaml
```

### 6.2 Environment Management

#### 6.2.1 Context Configuration
```json
// cdk.json
{
  "app": "npx ts-node --prefer-ts-exts bin/my-app.ts",
  "context": {
    "@aws-cdk/core:enableStackNameDuplicates": true,
    "aws-cdk:enableDiffNoFail": true,
    "@aws-cdk/core:stackRelativeExports": true
  }
}
```

#### 6.2.2 Environment-Specific Deployment
```typescript
const app = new cdk.App();

// Development environment
new MyStack(app, 'MyStack-Dev', {
  env: {
    account: '123456789012',
    region: 'us-east-1'
  },
  stage: 'dev'
});

// Production environment
new MyStack(app, 'MyStack-Prod', {
  env: {
    account: '123456789012',
    region: 'us-west-2'
  },
  stage: 'prod'
});
```

---

## 7. Testing and Validation

### 7.1 Unit Testing

#### 7.1.1 CDK Testing Framework
```typescript
import { Template } from 'aws-cdk-lib/assertions';
import * as cdk from 'aws-cdk-lib';
import { MyStack } from '../lib/my-stack';

test('S3 Bucket Created', () => {
  const app = new cdk.App();
  const stack = new MyStack(app, 'MyTestStack');
  const template = Template.fromStack(stack);

  template.hasResourceProperties('AWS::S3::Bucket', {
    VersioningConfiguration: {
      Status: 'Enabled'
    }
  });
});

test('Bucket Count', () => {
  const app = new cdk.App();
  const stack = new MyStack(app, 'MyTestStack');
  const template = Template.fromStack(stack);

  template.resourceCountIs('AWS::S3::Bucket', 1);
});
```

#### 7.1.2 Snapshot Testing
```typescript
test('Stack matches snapshot', () => {
  const app = new cdk.App();
  const stack = new MyStack(app, 'MyTestStack');
  const template = Template.fromStack(stack);
  
  expect(template.toJSON()).toMatchSnapshot();
});
```

### 7.2 Integration Testing

#### 7.2.1 CDK Integration Tests
```typescript
import { IntegTest } from '@aws-cdk/integ-tests';
import { App, Stack } from 'aws-cdk-lib';

const app = new App();
const stack = new Stack(app, 'IntegrationTestStack');

// Add constructs to test
// ...

// Create integration test
new IntegTest(app, 'MyIntegTest', {
  testCases: [stack]
});
```

### 7.3 Security and Compliance Validation

#### 7.3.1 CDK Security Checks
```bash
# Install cdk-nag for security checks
npm install cdk-nag

# Run security checks
cdk synth --plugin cdk-nag
```

#### 7.3.2 Custom Security Rules
```typescript
import { AwsSolutionsChecks } from 'cdk-nag';
import { NagSuppressions } from 'cdk-nag';

const app = new cdk.App();

// Add security checks
cdk.Aspects.of(app).add(new AwsSolutionsChecks({ verbose: true }));

// Suppress specific rules
NagSuppressions.addStackSuppressions(stack, [
  { id: 'AwsSolutions-S1', reason: 'Bucket logging not required for this use case' }
]);
```

---

## 8. CI/CD Integration

### 8.1 GitHub Actions Integration

#### 8.1.1 Basic CI/CD Pipeline
```yaml
name: CDK Deploy

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run tests
      run: npm test
      
    - name: Run CDK diff
      run: |
        npm run build
        cdk diff
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Deploy to AWS
      run: |
        npm run build
        cdk deploy --require-approval never
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### 8.2 AWS CodePipeline Integration

#### 8.2.1 CDK Pipeline Construct
```typescript
import * as codepipeline from 'aws-cdk-lib/aws-codepipeline';
import * as codepipeline_actions from 'aws-cdk-lib/aws-codepipeline-actions';
import { CodePipeline, CodePipelineSource, ShellStep } from 'aws-cdk-lib/pipelines';

export class CdkPipelineStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const pipeline = new CodePipeline(this, 'Pipeline', {
      pipelineName: 'MyCdkPipeline',
      synth: new ShellStep('Synth', {
        input: CodePipelineSource.gitHub('USER/REPO', 'main'),
        commands: [
          'npm ci',
          'npm run build',
          'npx cdk synth'
        ]
      })
    });

    // Add application stages
    pipeline.addStage(new ApplicationStage(this, 'Dev', {
      env: { account: '111111111111', region: 'us-east-1' }
    }));

    pipeline.addStage(new ApplicationStage(this, 'Prod', {
      env: { account: '111111111111', region: 'us-west-2' }
    }));
  }
}
```

---

## 9. Best Practices and Guidelines

### 9.1 Code Organization

#### 9.1.1 Project Structure Best Practices
- Separate stacks by environment and purpose
- Use consistent naming conventions
- Implement proper error handling
- Document complex constructs and patterns

#### 9.1.2 Construct Design Patterns
```typescript
// Good: Focused, reusable construct
export class DatabaseConstruct extends Construct {
  public readonly database: rds.DatabaseInstance;
  public readonly secret: secretsmanager.Secret;

  constructor(scope: Construct, id: string, props: DatabaseProps) {
    super(scope, id);
    
    // Implementation focused on database functionality
  }
}

// Good: Composition over inheritance
export class WebApplicationStack extends Stack {
  constructor(scope: Construct, id: string, props: WebApplicationProps) {
    super(scope, id, props);

    const network = new NetworkConstruct(this, 'Network', {...});
    const database = new DatabaseConstruct(this, 'Database', {...});
    const application = new ApplicationConstruct(this, 'Application', {
      vpc: network.vpc,
      database: database.database
    });
  }
}
```

### 9.2 Security Best Practices

#### 9.2.1 IAM and Permissions
```typescript
// Principle of least privilege
const role = new iam.Role(this, 'LambdaRole', {
  assumedBy: new iam.ServicePrincipal('lambda.amazonaws.com'),
  managedPolicies: [
    iam.ManagedPolicy.fromAwsManagedPolicyName('service-role/AWSLambdaBasicExecutionRole')
  ]
});

// Grant specific permissions only
bucket.grantRead(role);
table.grantReadData(role);
```

#### 9.2.2 Resource Security Configuration
```typescript
// Secure S3 bucket configuration
const secureBucket = new s3.Bucket(this, 'SecureBucket', {
  encryption: s3.BucketEncryption.S3_MANAGED,
  blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
  versioned: true,
  lifecycleRules: [{
    id: 'DeleteIncompleteMultipartUploads',
    abortIncompleteMultipartUploadAfter: cdk.Duration.days(7)
  }]
});

// RDS security configuration
const secureDatabase = new rds.DatabaseInstance(this, 'SecureDB', {
  engine: rds.DatabaseInstanceEngine.postgres({
    version: rds.PostgresEngineVersion.VER_13_7
  }),
  storageEncrypted: true,
  backupRetention: cdk.Duration.days(30),
  deletionProtection: true,
  multiAz: true
});
```

### 9.3 Performance and Cost Optimization

#### 9.3.1 Resource Optimization
```typescript
// Use appropriate instance sizes
const optimizedInstance = new ec2.Instance(this, 'OptimizedInstance', {
  instanceType: ec2.InstanceType.of(
    ec2.InstanceClass.T3A, // AMD instances for cost savings
    ec2.InstanceSize.MEDIUM
  ),
  machineImage: ec2.MachineImage.latestAmazonLinux2(),
  userData: ec2.UserData.forLinux()
});

// Implement auto-scaling
const autoScalingGroup = new autoscaling.AutoScalingGroup(this, 'ASG', {
  vpc: vpc,
  instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
  machineImage: ec2.MachineImage.latestAmazonLinux2(),
  minCapacity: 1,
  maxCapacity: 5,
  desiredCapacity: 2
});
```

#### 9.3.2 Monitoring and Alerting
```typescript
// CloudWatch alarms
const cpuAlarm = new cloudwatch.Alarm(this, 'HighCPU', {
  metric: autoScalingGroup.metricCPUUtilization(),
  threshold: 80,
  evaluationPeriods: 2,
  treatMissingData: cloudwatch.TreatMissingData.NOT_BREACHING
});

// SNS notification
const topic = new sns.Topic(this, 'AlertTopic');
cpuAlarm.addAlarmAction(new sns_actions.SnsAction(topic));
```

---

## 10. Troubleshooting and Debugging

### 10.1 Common Issues and Solutions

#### 10.1.1 Bootstrap Issues
```bash
# Re-bootstrap with newer version
cdk bootstrap --toolkit-stack-name CDKToolkit-v2

# Bootstrap with custom settings
cdk bootstrap --cloudformation-execution-policies arn:aws:iam::aws:policy/PowerUserAccess
```

#### 10.1.2 Dependency Resolution
```bash
# Clear CDK cache
rm -rf cdk.out/

# Reinstall dependencies
rm -rf node_modules/ package-lock.json
npm install

# Update CDK to latest version
npm update aws-cdk-lib
```

### 10.2 Debugging Techniques

#### 10.2.1 CloudFormation Template Analysis
```bash
# Generate and examine CloudFormation template
cdk synth > template.json

# Validate template
aws cloudformation validate-template --template-body file://template.json
```

#### 10.2.2 Stack Drift Detection
```bash
# Detect configuration drift
aws cloudformation detect-stack-drift --stack-name MyStack

# View drift results
aws cloudformation describe-stack-drift-detection-status --stack-drift-detection-id <id>
```

---

## 11. Advanced CDK Topics

### 11.1 Custom Resources

#### 11.1.1 Lambda-backed Custom Resources
```typescript
import * as cr from 'aws-cdk-lib/custom-resources';

const customResource = new cr.AwsCustomResource(this, 'MyCustomResource', {
  onCreate: {
    service: 'S3',
    action: 'putBucketNotification',
    parameters: {
      Bucket: bucket.bucketName,
      NotificationConfiguration: {
        CloudWatchConfigurations: [{
          Event: 's3:ObjectCreated:*',
          CloudWatchConfiguration: {
            LogGroupName: logGroup.logGroupName
          }
        }]
      }
    },
    physicalResourceId: cr.PhysicalResourceId.of('MyCustomResource')
  },
  policy: cr.AwsCustomResourcePolicy.fromSdkCalls({
    resources: cr.AwsCustomResourcePolicy.ANY_RESOURCE
  })
});
```

### 11.2 CDK Aspects

#### 11.2.1 Cross-Cutting Concerns
```typescript
import { IAspect, IConstruct } from 'constructs';

class TaggingAspect implements IAspect {
  constructor(private readonly tags: Record<string, string>) {}

  visit(node: IConstruct): void {
    if (node instanceof cdk.Cfn