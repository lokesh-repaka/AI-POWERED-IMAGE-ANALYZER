# VISIO — AI-Powered Image Analyzer

An enterprise-grade, serverless image analysis web application that combines **AWS Rekognition** for classical computer vision with **AWS Bedrock** (Mistral model) for generative AI-powered image descriptions. The entire infrastructure is provisioned and managed using **Terraform**.

## Project Overview

VISIO demonstrates a production-ready architecture that leverages serverless AWS services to build a scalable, cost-effective AI application. Users upload images through a beautifully designed web interface, which triggers a Lambda function that analyzes the image using Rekognition and Bedrock.

### Architecture Highlights

- **Frontend**: Modern, responsive HTML5 single-page application hosted on AWS S3
- **Backend**: Serverless Lambda function for image processing and analysis
- **APIs**: API Gateway for secure HTTPS endpoint exposure
- **AI Services**: AWS Rekognition (object detection) + AWS Bedrock/Mistral (generative descriptions)
- **Infrastructure**: Fully automated with Terraform (Infrastructure as Code)

## Key Features

✨ **Dual AI Processing**
- AWS Rekognition extracts precise object labels and detected items
- AWS Bedrock (Mistral model) generates natural language descriptions

🚀 **Serverless Architecture**
- Zero server management
- Auto-scaling on demand
- Pay only for what you use

🛠️ **Infrastructure as Code**
- Complete infrastructure defined in Terraform
- Reproducible deployments across environments
- Version-controlled infrastructure

🎨 **Premium UI/UX**
- Modern, animated interface with glassmorphism design
- Real-time loading states
- Responsive design for all devices
- Accessibility-first approach

🔒 **Security Best Practices**
- Least-privilege IAM roles
- CORS support for frontend-backend communication
- Secure image processing without persistence

## Project Structure

```
├── frontend/
│   └── index.html                 # Responsive web UI (VISIO)
├── lambda/
│   └── image_analyzer.py          # Lambda function handler
├── terraform/
│   ├── main.tf                    # Primary infrastructure definitions
│   ├── variables.tf               # Input variables & defaults
│   └── outputs.tf                 # Output values
└── README.md                      # This file
```

## Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Frontend** | HTML5, CSS3, Vanilla JavaScript | User interface & image upload |
| **Backend** | Python 3.9, AWS Lambda | Image processing logic |
| **Computer Vision** | AWS Rekognition | Object/label detection |
| **Generative AI** | AWS Bedrock (Mistral) | Image description generation |
| **API Layer** | AWS API Gateway | REST endpoint exposure |
| **Storage** | AWS S3 | Frontend hosting |
| **Infrastructure** | Terraform | Infrastructure as Code |
| **IAM** | AWS IAM | Access control & permissions |

## Prerequisites

Before deploying, ensure you have:

1. **AWS Account** with appropriate permissions
   - Ability to create Lambda, API Gateway, S3, IAM roles
   - Access to Bedrock (may require service enablement in your region)
   - Access to Rekognition

2. **Local Tools**
   - Terraform >= 1.0
   - AWS CLI v2
   - Python 3.9+

3. **AWS Credentials**
   ```bash
   aws configure
   ```
   Configure your AWS credentials with a profile that has administrative permissions

4. **AWS Bedrock Access**
   - Bedrock is available in specific regions
   - Verify Mistral model access in your chosen region
   - You may need to request model access in the Bedrock console

## Deployment Guide

### Step 1: Clone/Navigate to Project

```bash
cd /path/to/ai-image-analyzer
```

### Step 2: Initialize Terraform

```bash
cd terraform
terraform init
```

This downloads required providers and initializes the Terraform working directory.

### Step 3: Review Terraform Plan

```bash
terraform plan
```

Review the resources that will be created:
- IAM roles and policies
- Lambda function
- API Gateway and deployment
- S3 bucket for frontend
- CORS configuration

### Step 4: Deploy Infrastructure

```bash
terraform apply
```

Type `yes` when prompted. Terraform will:
1. Create IAM execution role for Lambda
2. Create policies for Rekognition and Bedrock access
3. Package and deploy Lambda function
4. Create API Gateway with `/analyze` endpoint
5. Create S3 bucket for frontend hosting
6. Configure CORS and public access

**Deployment time**: ~5-10 minutes

### Step 5: Get Deployment Outputs

```bash
terraform output
```

You'll receive:
- **api_gateway_url**: The API endpoint for image analysis
- **frontend_website_endpoint**: S3 website URL
- **lambda_function_name**: Name of deployed function
- **frontend_bucket_name**: S3 bucket name

Example output:
```
api_gateway_url = "https://abc123def456.execute-api.us-east-1.amazonaws.com/v1"
frontend_bucket_name = "ai-image-analyzer-frontend-a1b2c3d4e5f6g7h8"
frontend_website_endpoint = "ai-image-analyzer-frontend-a1b2c3d4e5f6g7h8.s3-website-us-east-1.amazonaws.com"
lambda_function_name = "ai-image-analyzer-function"
```

### Step 6: Deploy Frontend

Update the API endpoint in `frontend/index.html` and upload to S3:

```bash
# Get the API Gateway URL from terraform output
API_URL=$(terraform output -raw api_gateway_url)
BUCKET=$(terraform output -raw frontend_bucket_name)

# Update the frontend with the API endpoint
# (You may need to modify index.html to include the API_URL)

# Upload to S3
aws s3 cp ../frontend/index.html s3://$BUCKET/index.html --content-type "text/html"
```

### Step 7: Access Your Application

Visit the frontend website endpoint in your browser:
```
http://ai-image-analyzer-frontend-[suffix].s3-website-us-east-1.amazonaws.com
```

## How It Works

### User Flow

1. **Upload**: User selects an image (PNG/JPEG) through the VISIO web interface
2. **Preview**: Image is previewed in the browser
3. **Submit**: Click "Analyze Image" to send to backend
4. **Lambda Processing**:
   - Image is base64-encoded and sent via API
   - Lambda receives image data
   - **Rekognition**: Detects up to 10 labels with 80%+ confidence
   - **Bedrock**: Generates natural description from detected labels
5. **Response**: Labels and description returned to frontend
6. **Display**: Results shown with animated label tags and description text

### Technical Flow

```
Frontend (S3)
    ↓ [Image Upload - Base64]
API Gateway (/analyze POST)
    ↓
Lambda Function (image_analyzer.py)
    ├→ Decode base64 image
    ├→ AWS Rekognition.detect_labels()
    │   └→ Returns: [Label1, Label2, ...]
    ├→ AWS Bedrock.invoke_model()
    │   └→ Prompt: "Based on these labels: ... generate a description"
    │   └→ Model: Mistral Small 2402
    └→ Return: {labels: [...], description: "..."}
    ↓ [JSON Response]
Frontend (S3)
    ↓ [Display Results]
User sees labels and AI description
```

## Configuration

### Environment Variables (Terraform)

Customize deployment via `terraform/variables.tf`:

```hcl
variable "aws_region" {
  default = "us-east-1"  # Change to your region
}

variable "project_name" {
  default = "ai-image-analyzer"  # Custom prefix for all resources
}

variable "environment" {
  default = "dev"  # dev, staging, prod
}
```

Deploy with custom values:
```bash
terraform apply -var="aws_region=us-west-2" -var="environment=prod"
```

### Lambda Configuration

Edit `lambda/image_analyzer.py` to customize:

```python
# Rekognition settings
MaxLabels=10,              # Number of labels to detect
MinConfidence=80           # Confidence threshold (0-100)

# Bedrock settings
"max_tokens": 100,         # Description length
"temperature": 0.5,        # Creativity (0=deterministic, 1=creative)
"top_p": 0.9              # Diversity control
```

After changes, redeploy:
```bash
cd terraform
terraform apply
```

## API Documentation

### Endpoint
```
POST /analyze
```

### Request

```bash
curl -X POST https://your-api-gateway-url/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "image": "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
  }'
```

**Request Body**:
```json
{
  "image": "base64-encoded-image-string"
}
```

### Response (Success)

```json
{
  "labels": ["Dog", "Pet", "Animal", "Canine"],
  "description": "A brown and white dog sitting in a grassy field with yellow flowers under a blue sky."
}
```

**Status Code**: `200 OK`

### Response (Error)

```json
{
  "error": "No image provided"
}
```

**Status Code**: `400 Bad Request` or `500 Internal Server Error`

### CORS Headers

All responses include:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: OPTIONS,POST
Access-Control-Allow-Headers: Content-Type
```

## Security Considerations

### IAM Permissions

The Lambda function is granted **least-privilege** access:

- **Rekognition**: `rekognition:DetectLabels` only
- **Bedrock**: `bedrock:InvokeModel` only
- **Logging**: CloudWatch Logs creation and writing

Resource policies are scoped to `*` for these services (no specific resource ARNs available).

### API Security

- **No Authentication**: Currently open to public (CORS enabled)
- **Optional Enhancement**: Add API Key or AWS SigV4 authentication
- **CORS Configuration**: Set in API Gateway

### Data Privacy

- Images are **not persisted** to storage
- Images exist only in Lambda memory during processing
- No logs contain image data
- Bedrock/Rekognition responses are transient

### Recommended Enhancements

```hcl
# Add API Key requirement
resource "aws_api_gateway_api_key" "api_key" {
  name = "${var.project_name}-api-key"
}

resource "aws_api_gateway_usage_plan" "usage_plan" {
  name       = "${var.project_name}-usage-plan"
  api_stages {
    api_id = aws_api_gateway_rest_api.api.id
    stage  = aws_api_gateway_stage.stage.stage_name
  }
}
```

## Monitoring & Troubleshooting

### View Lambda Logs

```bash
# List recent logs
aws logs tail /aws/lambda/ai-image-analyzer-function --follow

# View specific invocation
aws logs filter-log-events \
  --log-group-name /aws/lambda/ai-image-analyzer-function \
  --start-time $(date -d '1 hour ago' +%s)000
```

### Test Lambda Directly

```bash
# Create test event (base64-encoded test image)
aws lambda invoke \
  --function-name ai-image-analyzer-function \
  --payload '{"body": "{\"image\": \"base64imagestring\"}"}' \
  response.json

cat response.json
```

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| "No labels detected" | Image too ambiguous or quality too low | Use clear, well-lit images |
| Bedrock timeout | Model not enabled in region | Enable Mistral in Bedrock console |
| 403 Forbidden | Insufficient IAM permissions | Verify Lambda execution role policy |
| CORS errors | Frontend not loading from S3 properly | Check S3 bucket CORS and API Gateway configuration |
| Image too large | Base64 payload exceeds Lambda limit | Compress image before upload |

## Cost Estimation

Monthly costs for typical usage (1000 analyses):

| Service | Usage | Cost |
|---------|-------|------|
| **Lambda** | 1000 invocations × 2 sec | ~$0.20 |
| **Rekognition** | 1000 detect_labels calls | ~$1.00 |
| **Bedrock** | 1000 Mistral invocations | ~$0.50 |
| **API Gateway** | 1000 requests | ~$0.35 |
| **S3** | Frontend hosting | ~$0.50 |
| **Total** | | **~$2.55** |

**Note**: Costs scale linearly with usage. First 1 million Lambda invocations per month include generous free tier.

## Development & Testing

### Local Testing

```bash
# Test the Lambda function locally
cd lambda
python3 -c "
import json
import base64
from image_analyzer import lambda_handler

# Mock event with base64 image
event = {
    'body': json.dumps({
        'image': 'your-base64-string'
    })
}

response = lambda_handler(event, None)
print(json.dumps(json.loads(response['body']), indent=2))
"
```

### Update Frontend

Edit `frontend/index.html` and update the API endpoint:

```javascript
// In index.html, find the API call section and update:
const API_ENDPOINT = "https://your-api-gateway-url/analyze";
```

Then redeploy to S3:

```bash
aws s3 cp frontend/index.html s3://$(terraform output -raw frontend_bucket_name)/index.html
```

## Cleanup & Destruction

To remove all AWS resources and stop incurring costs:

```bash
cd terraform
terraform destroy
```

**Warning**: This will:
- Delete S3 bucket (force_destroy = true)
- Delete Lambda function
- Delete API Gateway
- Delete IAM roles and policies

Confirm by typing `yes` when prompted.

## Learning Outcomes

By deploying and using this project, you'll understand:

✅ **AWS Serverless Architecture**: Lambda, API Gateway, S3
✅ **Infrastructure as Code**: Terraform best practices
✅ **IAM Security**: Least-privilege access control
✅ **AI/ML Integration**: Rekognition + Bedrock
✅ **Frontend-Backend Decoupling**: Serverless patterns
✅ **CORS & API Design**: Modern API best practices
✅ **Cost Optimization**: Serverless cost efficiency
✅ **Production Deployment**: Reproducible infrastructure

## Future Enhancements

- [ ] Add authentication (API Key / OAuth)
- [ ] Implement image storage with S3/DynamoDB
- [ ] Add rate limiting and usage analytics
- [ ] Support batch image processing
- [ ] Multi-model support (Anthropic, LLaMA)
- [ ] CloudFront CDN for frontend
- [ ] Lambda image optimization
- [ ] API versioning and deprecation support

## References

- [AWS Rekognition Documentation](https://docs.aws.amazon.com/rekognition/)
- [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [API Gateway CORS Documentation](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html)

## License

Open source - use freely for educational and commercial purposes.

## Support

For issues or questions:
1. Check [Troubleshooting](#monitoring--troubleshooting) section
2. Review AWS CloudWatch Logs
3. Verify IAM permissions
4. Check Bedrock/Rekognition service limits in your region

---

**Built with** ☁️ AWS, 🏗️ Terraform, 🤖 AI/ML • **Made for** Cloud Architects & DevOps Engineers
