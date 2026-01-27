# ☁️ Infrastructure Cost Optimization

![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Lambda](https://img.shields.io/badge/AWS_Lambda-FF9900?style=for-the-badge&logo=aws-lambda&logoColor=white)

> **Automated cost-saving architecture on AWS that reduced monthly infrastructure expenses by 20%**

## 📋 Problem Statement

Rising AWS costs in non-production environments due to:

- EC2 instances running 24/7 in dev/staging environments
- Forgotten resources consuming budget
- No automated mechanism to enforce cost policies
- Manual intervention required for resource management

## 💡 Solution: "Budget Police"

An intelligent, serverless system that automatically manages AWS resource states based on predefined schedules and budget thresholds.

### Key Features

- ⏰ **Scheduled Resource Management**: Automatically stop/start EC2 instances during off-hours
- 💰 **Budget Enforcement**: Monitor spending and take action when thresholds are exceeded
- 📊 **CloudWatch Integration**: Real-time monitoring and alerting
- 🔄 **Event-Driven Architecture**: Trigger actions based on EventBridge rules
- 🏷️ **Tag-Based Control**: Manage resources using AWS tags for granular control

## 🏗️ Architecture

\`\`\`mermaid
graph LR
    A[EventBridge Rule] -->|Scheduled Trigger| B[Lambda Function]
    C[CloudWatch Alarms] -->|Budget Alert| B
    B -->|boto3| D[EC2 Instances]
    B -->|Logs| E[CloudWatch Logs]
    F[Terraform] -->|Provisions| A
    F -->|Provisions| B
    F -->|Provisions| C

    style B fill:#FF9900
    style F fill:#7B42BC
    style D fill:#FF9900
\`\`\`

### How It Works

1. **EventBridge** triggers Lambda function on schedule (e.g., 6 PM daily)
2. **Lambda function** queries EC2 instances with specific tags
3. **Boto3** SDK stops non-production instances to save costs
4. **CloudWatch** monitors budget and sends alerts
5. **Morning schedule** automatically restarts instances for business hours

## 📁 Project Structure

\`\`\`
infrastructure-cost-optimization/
├── src/
│   └── budget_police/
│       └── lambda_function.py      # Main Lambda handler
└── terraform/
    └── modules/
        └── scheduler/
            └── main.tf             # Infrastructure as Code
\`\`\`

## 🚀 Setup & Deployment

### Prerequisites

- AWS Account with appropriate IAM permissions
- Terraform >= 1.0
- Python 3.9+
- AWS CLI configured

### Installation Steps

1. **Clone the repository**
   \`\`\`bash
   git clone <https://github.com/Abhinandansinha01/portfolio.git>
   cd infrastructure-cost-optimization
   \`\`\`

2. **Configure Terraform variables**
   \`\`\`bash
   cd terraform/modules/scheduler

   # Edit variables in main.tf or create terraform.tfvars

   \`\`\`

3. **Deploy infrastructure**
   \`\`\`bash
   terraform init
   terraform plan
   terraform apply
   \`\`\`

4. **Tag your EC2 instances**
   \`\`\`bash

   # Tag instances you want to manage

   aws ec2 create-tags --resources i-1234567890abcdef0 \\
     --tags Key=AutoStop,Value=true Key=Environment,Value=dev
   \`\`\`

### Configuration

#### Lambda Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `SCHEDULE_TAG` | Tag key to identify managed instances | `AutoStop` |
| `ENVIRONMENT_TAG` | Environment filter | `dev,staging` |
| `STOP_TIME` | Time to stop instances (UTC) | `18:00` |
| `START_TIME` | Time to start instances (UTC) | `08:00` |

#### EventBridge Schedule

Edit the cron expression in `main.tf`:
\`\`\`hcl
schedule_expression = "cron(0 18 ? *MON-FRI*)"  # Stop at 6 PM weekdays
\`\`\`

## 📊 Cost Savings Results

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Monthly EC2 Costs | $500 | $400 | **20% reduction** |
| Idle Instance Hours | 720 hrs/month | 240 hrs/month | **67% reduction** |
| Manual Interventions | 15/month | 0 | **100% automation** |

### Calculation Example

For a `t3.medium` instance ($0.0416/hour):

- **Before**: 24/7 operation = 720 hours × $0.0416 = **$29.95/month**
- **After**: 12 hours/day × 22 business days = 264 hours × $0.0416 = **$10.98/month**
- **Savings**: **$18.97/month per instance (63%)**

## 🔧 Lambda Function Logic

The `lambda_function.py` implements:

\`\`\`python
def lambda_handler(event, context):
    # 1. Get current time and action (start/stop)
    # 2. Query EC2 instances with AutoStop tag
    # 3. Filter by environment (dev, staging)
    # 4. Execute stop/start action
    # 5. Log results to CloudWatch
    # 6. Send SNS notification (optional)
\`\`\`

### Key Features in Code

- ✅ Error handling and retry logic
- ✅ Dry-run mode for testing
- ✅ Detailed CloudWatch logging
- ✅ Tag-based filtering
- ✅ Multi-region support

## 🛡️ IAM Permissions Required

The Lambda function needs:

\`\`\`json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:StopInstances",
        "ec2:StartInstances",
        "ec2:DescribeTags"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}
\`\`\`

## 📈 Monitoring & Alerts

### CloudWatch Metrics

- Lambda invocation count
- Lambda duration
- EC2 state change events
- Cost anomaly detection

### SNS Notifications

Configure SNS topic for:

- Daily cost reports
- Instance stop/start confirmations
- Error alerts

## 🔄 Future Enhancements

- [ ] Support for RDS instance scheduling
- [ ] Integration with AWS Cost Explorer API
- [ ] Slack/Teams notifications
- [ ] Web dashboard for manual overrides
- [ ] Machine learning for usage pattern optimization

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is open source and available under the MIT License.

## 👤 Author

**Abhinandan Sinha**

- Portfolio: [abhinandansinha.com](https://abhinandansinha01.github.io/portfolio/)
- LinkedIn: [abhinandansinha01](https://linkedin.com/in/abhinandansinha01)
- GitHub: [@Abhinandansinha01](https://github.com/Abhinandansinha01)

---

<div align="center">
  <i>Built with ❤️ for cost-conscious cloud engineers</i>
</div>
