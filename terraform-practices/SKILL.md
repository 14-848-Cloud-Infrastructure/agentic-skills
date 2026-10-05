---
name: terraform-practices
description: >
  This skill should be used when the user asks to "analyze Terraform modules",
  "scan IaC for security issues", "review Terraform configurations",
  "check infrastructure code for misconfigurations", or "audit cloud resources".
license: MIT + Commons Clause
---
# Terraform Practices

> The category is Engineering, and the domain is Infrastructure as Code.

## Overview

This skill analyzes Terraform configurations for module complexity, security misconfigurations, and infrastructure best practices. It catches open ports, public buckets, missing encryption, and overly permissive IAM policies before they reach production.

## Clarify first

Confirm these inputs before analyzing or scanning. If any of them is unknown or vague, ask the user instead of assuming an answer.

- [ ] **Target path.** The module or environment directory to analyze, passed as `--path`. This is the subject of the run.
- [ ] **Task.** Module complexity and quality analysis uses `tf_module_analyzer`, and a security misconfiguration scan uses `tf_security_scanner`.
- [ ] **Minimum severity or gate.** The severity bar for findings and whether a finding blocks a PR, passed as `--min-severity`. It changes both the report and the CI pass or fail result.

**Stop rule.** Ask only the 2-3 questions that most change the output. If the user says "just draft it," proceed and list your assumptions at the top of the artifact.

## Quick start

```bash
# Analyze Terraform module structure and complexity
python scripts/tf_module_analyzer.py --path ./modules/vpc

# Scan for security misconfigurations
python scripts/tf_security_scanner.py --path ./environments/production

# JSON output for CI pipelines
python scripts/tf_security_scanner.py --path . --format json

# Recursive analysis of all modules
python scripts/tf_module_analyzer.py --path . --recursive
```

## Tools overview

### tf_module_analyzer.py

This script checks Terraform modules for complexity, structure, dependencies, and documentation quality.

| Feature | Description |
|---------|-------------|
| Complexity scoring | Scores modules by resource count, variable count, and nesting |
| Dependency mapping | Maps module dependencies and data source usage |
| Variable analysis | Checks for missing types, defaults, and descriptions |
| Output completeness | Validates output documentation and coverage |
| Naming conventions | Checks resource and variable naming patterns |

### tf_security_scanner.py

This script scans Terraform configurations for security misconfigurations and compliance violations.

| Feature | Description |
|---------|-------------|
| Open ports | Detects 0.0.0.0/0 CIDR in security groups |
| Public access | Flags public S3 buckets, databases, and instances |
| Encryption gaps | Checks for missing encryption at rest and in transit |
| IAM overreach | Identifies wildcard actions and overly broad policies |
| Logging gaps | Verifies CloudTrail, flow logs, and access logging |

## Workflows

### Security review workflow

1. **Scan.** Run `tf_security_scanner.py` across all environments.
2. **Triage.** Handle the critical findings first, such as public data and open access.
3. **Remediate.** Apply the recommended fix for each finding.
4. **Verify.** Scan again to confirm the fixes resolved the issues.
5. **Gate.** Add the scanner to PR checks so every change is held to the same rules.

### Module quality workflow

1. **Analyze.** Run `tf_module_analyzer.py` on each module.
2. **Score.** Review the complexity scores and find the modules above the threshold.
3. **Refactor.** Break down any module whose complexity score is above 70/100.
4. **Document.** Fill in missing variable and output descriptions.
5. **Standardize.** Apply consistent naming and file organization.

### CI integration

```bash
# Security gate
python scripts/tf_security_scanner.py --path . --format json --min-severity high
if [ $? -ne 0 ]; then
  echo "Security scan failed - blocking merge"
  exit 1
fi

# Module quality check
python scripts/tf_module_analyzer.py --path . --recursive --format json
```

## Reference documentation

- [Terraform practices reference](references/terraform-practices.md) covers module design, state management, and naming conventions.

## Common patterns quick reference

### Module structure
```
modules/vpc/
  main.tf           # Primary resources
  variables.tf      # Input variables with descriptions
  outputs.tf        # Module outputs
  versions.tf       # Required providers and versions
  locals.tf         # Local values and computed expressions
```

### Security checklist
| Resource | Check | Rule |
|----------|-------|------|
| Security Groups | No 0.0.0.0/0 ingress | Restrict to known CIDRs |
| S3 Buckets | No public ACLs | Use bucket policies instead |
| RDS | No public access | Set publicly_accessible = false |
| EBS/S3/RDS | Encryption enabled | Add encryption configuration |
| IAM | No wildcard actions | Use least-privilege policies |
| CloudTrail | Enabled in all regions | is_multi_region_trail = true |
| VPC | Flow logs enabled | Create flow log resources |

### Complexity scoring
| Score | Rating | Action |
|-------|--------|--------|
| 0-30 | Low | No action needed |
| 31-60 | Medium | Consider splitting |
| 61-80 | High | Should refactor |
| 81-100 | Critical | Must refactor |
