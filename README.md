# AWS Bedrock AgentCore Multimodal RAG Tutorial

This repository contains a complete hands-on example of building a Retrieval-Augmented Generation (RAG) system using AWS Bedrock AgentCore with multimodal support (text and images). The agent can retrieve and process both text documents and images from a knowledge base.

## Overview

This project demonstrates how to:
- Set up an AWS Bedrock Knowledge Base with multimodal data sources
- Create an AgentCore runtime with a custom LangChain agent
- Build a RAG agent that handles both text and image retrieval
- Deploy and invoke the agent via REST API

## Architecture

The solution consists of:
- **AWS Bedrock Knowledge Base**: Stores and indexes documents (text and images) using S3 as the data source
- **S3 Vectors**: Vector storage for embeddings
- **Bedrock AgentCore Runtime**: Hosts the custom agent code
- **Cognito User Pool**: Provides authentication for API access
- **Custom LangChain Agent**: Implements the RAG workflow with multimodal document handling

## Prerequisites

Before you begin, ensure you have:

1. **AWS Account** with appropriate permissions for:
   - Bedrock AgentCore
   - Bedrock Knowledge Bases
   - S3 and S3 Vectors
   - Cognito
   - ECR
   - IAM

2. **Local Tools**:
   - Python 3.12+ with virtual environment support
   - Terraform >= 1.14.3
   - AWS CLI configured
   - Docker (for building and pushing agent images)

3. **S3 Bucket**: An existing S3 bucket containing your documents (text files, PDFs, images, etc.)

## Setup Instructions

### 1. Clone the Repository

```bash
git clone <repository-url>
cd agentcore-blog
```

### 2. Configure Terraform Variables

Copy the example Terraform variables file and update it with your values:

```bash
cd infra
cp example.tfvars terraform.tfvars
```

Edit `terraform.tfvars`:

```hcl
region  = "us-east-1"
profile = "" # Leave empty to use default AWS credentials
tags = {
  project = "agentcore-blog"
}
data_source_bucket_arn = "arn:aws:s3:::your-bucket-name"
ecr_repository_name    = "your-ecr-repository-name"
```

### 3. Initialize and Apply Terraform

```bash
terraform init
terraform plan
terraform apply
```

This will create:
- Bedrock Knowledge Base with multimodal parsing
- S3 Vector bucket and index
- Bedrock AgentCore runtime
- Cognito user pool and client
- ECR repository for agent code
- IAM roles and policies

After successful deployment, note the outputs:
- `agentcore_runtime_id`
- `cognito_client_id`
- `ecr_repository_name`

### 4. Prepare Agent Environment

```bash
cd ../agent
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 5. Build and Push Agent to ECR

Use the provided script to build and push the Docker image:

```bash
cd ../scripts
cp env.agent.template .env.agent # fill out the ecr repository name
chmod +x upload-agent-to-ecr.sh
./upload-agent-to-ecr.sh
```

## Key Features

### Multimodal Document Handling

The `RetrieverTool.py` includes logic to handle both text and image documents:

- **Text documents**: Returns the full `page_content`
- **Image documents**: Extracts the image description from metadata (`x-amz-bedrock-kb-description`) instead of raw image data

This allows the agent to work with both document types seamlessly in the RAG pipeline.

### Agent Workflow

The agent follows a three-step workflow:

1. **Generate Query**: The LLM analyzes the user query and determines if knowledge base retrieval is needed
2. **Retrieve**: If needed, searches the knowledge base and retrieves relevant documents
3. **Generate Answer**: Uses the retrieved context to generate a comprehensive answer

## Project Structure

```
agentcore-blog/
├── agent/                 # Agent code
│   ├── app.py            # AgentCore entrypoint
│   ├── RetrieverAgent.py # LangGraph agent definition
│   ├── RetrieverTool.py  # Knowledge base retrieval tool
│   ├── llm_model.py      # Bedrock LLM configuration
│   └── prompts.py        # System prompts
├── infra/                # Terraform infrastructure
│   ├── bedrock_kb.tf     # Knowledge base resources
│   ├── agentcore_runtime.tf # Runtime resources
│   └── cognito.tf        # Authentication
├── kb/                   # Sample documents
└── scripts/              # Deployment scripts
```

## Cleanup

To remove all resources:

```bash
cd infra
terraform destroy
```

**Note**: This will delete all resources including the knowledge base, runtime, and associated data. Make sure you have backups if needed.
