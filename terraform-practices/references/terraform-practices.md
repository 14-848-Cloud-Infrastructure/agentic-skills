# Terraform practices reference

## Module design patterns

### Composition pattern
Build infrastructure from small modules that each handle one job.

```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.0.0.0/16"
}

module "database" {
  source    = "./modules/rds"
  vpc_id    = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids
}
```

### Facade pattern
Put a simplified interface in front of a complex multi-module setup.

```hcl
module "application_stack" {
  source       = "./modules/app-stack"
  app_name     = "myapp"
  environment  = "production"
  # Internally creates VPC, ECS, ALB, RDS, etc.
}
```

## File organization

### Standard module layout
```
modules/service-name/
  main.tf          - Primary resources
  variables.tf     - Input variables (all with type + description)
  outputs.tf       - Module outputs (all with description)
  versions.tf      - terraform { required_providers {} }
  locals.tf        - Computed local values
  data.tf          - Data sources (optional)
```

### Environment layout
```
environments/
  production/
    main.tf        - Module calls with prod values
    backend.tf     - Remote state config
    terraform.tfvars
  staging/
    main.tf
    backend.tf
    terraform.tfvars
```

## Security best practices

### S3 bucket security
```hcl
resource "aws_s3_bucket" "data" {
  bucket = "my-secure-bucket"
}

resource "aws_s3_bucket_public_access_block" "data" {
  bucket = aws_s3_bucket.data.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_server_side_encryption_configuration" "data" {
  bucket = aws_s3_bucket.data.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
  }
}
```

### Security group rules
```hcl
# Good: specific CIDR
resource "aws_security_group_rule" "ssh" {
  type        = "ingress"
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["10.0.0.0/8"]  # Internal only
  security_group_id = aws_security_group.main.id
}

# Bad: open to world
resource "aws_security_group_rule" "ssh_bad" {
  cidr_blocks = ["0.0.0.0/0"]  # Never do this for SSH
}
```

### IAM least privilege
```hcl
data "aws_iam_policy_document" "app" {
  statement {
    effect = "Allow"
    actions = [
      "s3:GetObject",
      "s3:PutObject",
    ]
    resources = [
      "${aws_s3_bucket.app.arn}/*",
    ]
  }
}
```

## State management

### Remote state configuration
```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-myorg"
    key            = "production/vpc/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

### State locking
Always lock state with DynamoDB or an equivalent service so that concurrent runs cannot modify it at the same time.

## Naming conventions

| Resource | Convention | Example |
|----------|-----------|---------|
| Resources | snake_case, descriptive | `aws_s3_bucket.app_data` |
| Variables | snake_case, prefixed | `var.vpc_cidr` |
| Outputs | snake_case, descriptive | `output.vpc_id` |
| Modules | kebab-case directories | `modules/app-cluster/` |
| Files | lowercase, descriptive | `main.tf`, `variables.tf` |

## Provider version pinning

```hcl
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

Always pin at least the minor version, so a new provider release cannot bring in breaking changes.
