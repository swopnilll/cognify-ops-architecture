# Jira Clone Application - Core Architecture

## Overview

This document provides an overview of the core architecture for the Jira clone application with an integrated custom-trained foundation model.

## Architecture Components

### Frontend Layer

- **Technology**: React
- **Features**:
  - Main UI for managing tasks, assignments, and project updates
  - ChatGPT-like frontend component for AI-based assistance

### Backend Components

- **Technology**: Node.js / Express / Next.js
- **Function**: Serves as the application server layer handling business logic and API endpoints

### Data Storage

- **Technology**: PostgreSQL deployed on AWS RDS
- **Function**: Persistent storage for all project-related data

## Foundation Model Custom Training Flow

### Training Trigger Points and Data Flow

The custom training process is integrated with user interactions and follows a structured pipeline for data processing.

#### 1. Data Capture Phase

- Users interact with the frontend to manage project-related content (tasks, assignments, status updates)
- Interaction data is processed by the backend services
- All project data is persistently stored in PostgreSQL (AWS RDS)
- This stored data serves as the raw material for AI model training

#### 2. Data Transfer to Training Pipeline

- New or updated project data in PostgreSQL triggers a data transfer to **Amazon SQS**
- **Amazon SQS** acts as an asynchronous message queue to decouple core application and AI workload
- This ensures application performance remains unaffected while AI training runs independently

#### 3. Custom Training Initiation

- **AWS Lambda functions** consume messages from SQS
- These functions process raw application data into training-ready formats
- Processed data is stored in structured formats (JSONL files) in **Amazon S3**
- The **Custom Data Train** component in AWS Bedrock initiates model training using the stored data

## Potential Issues & Challenges

### 1. Data Latency in Model Training

- The AI model training process is asynchronous, meaning there is a delay between user interactions and model updates.
- If frequent updates are required, the model might lag behind real-time data.

### 2. Cost Considerations

- **AWS Bedrock, S3, Lambda, and RDS** can incur significant costs, especially with high data volume and frequent training cycles.
- **Amazon SQS** and **Lambda executions** add additional costs based on message throughput.

### 3. Scalability Challenges in AI Training

- If project data grows significantly, **Lambda functions** processing the training data may hit execution time limits (max 15 minutes).
- **S3 storage** could become inefficient without proper data management (e.g., old training data retention policies).

### 4. Consistency Issues Between Core Application and AI Models

- Since the training process is independent, AI predictions may be outdated if the model is not retrained frequently.
- There’s no version control mentioned for AI models, meaning old models might still be used even if new data is available.

### 5. No Real-Time AI Integration

- The architecture supports **batch training**, but it lacks real-time AI inference.
- If AI-powered features (like smart task recommendations) are needed in real time, **AWS Bedrock’s inference endpoint** should be integrated.

### 6. Potential Bottleneck in SQS Processing

- If the application generates a large number of updates, SQS messages may queue up, causing delays in AI training.
- **Dead-letter queues (DLQs)** should be implemented to handle failed processing attempts.

### 7. Security and Data Privacy Concerns

- Sensitive project-related data is sent to **AWS Bedrock** for AI training—this could raise privacy concerns.
- Proper **data anonymization and encryption** should be implemented before storing training data in S3.

## Possible Solutions & Enhancements

✅ Implement **incremental model training** instead of full retraining to reduce costs and latency.
✅ Introduce **real-time AI inference** using a Bedrock model API.
✅ Use **a dedicated ETL pipeline (AWS Glue)** for better data transformation before training.
✅ Monitor **SQS message queue length** and optimize Lambda concurrency to avoid bottlenecks.
✅ Apply **model versioning** to track different AI model versions for rollback if needed.

## Summary

This architecture ensures seamless integration of AI-based assistance within a Jira-like project management application while maintaining application performance and scalability. The use of AWS services enables efficient data processing and model customization workflows.
