# AWS Lambda - Serverless Compute

## 📖 ¿Qué es Lambda?

**AWS Lambda** permite ejecutar código sin aprovisionar o gestionar servidores. Pagas solo por el tiempo de cómputo que consumes.

```
Lambda = Function as a Service (FaaS)
- No servers to manage
- Scales automatically
- Pay per execution
- Event-driven
```

**Analogía:**
```
EC2: Rentas un apartamento completo (24/7)
Lambda: Pagas por habitación de hotel (solo cuando la usas)
```

---

## 🎯 Conceptos Fundamentales

### Function

Unidad básica de deployment.

```yaml
Function: process-image
Runtime: Python 3.11
Handler: lambda_function.lambda_handler
Memory: 512 MB
Timeout: 30 seconds
Role: lambda-execution-role

Code: Uploaded as ZIP or container image
```

### Invocation

Ejecución de la función.

```
Trigger → Lambda Function → Output

Invocation types:
1. Synchronous (request-response)
2. Asynchronous (fire-and-forget)
3. Event source mapping (streams)
```

### Runtime

Entorno de ejecución para tu código.

```yaml
Runtimes soportados:
- Node.js: 18.x, 20.x
- Python: 3.9, 3.10, 3.11, 3.12
- Java: 8, 11, 17, 21
- .NET: 6, 8
- Go: 1.x
- Ruby: 3.2, 3.3
- Custom Runtime: Cualquier lenguaje (usando Runtime API)
- Container Image: Cualquier lenguaje en Docker
```

---

## 💻 Crear Function

### Via Console

```
1. Lambda Console → Create function

2. Author from scratch
   Function name: hello-lambda
   Runtime: Python 3.11
   
3. Permissions
   Execution role: Create new role with basic permissions
   
4. Code source (inline editor)

5. Deploy

6. Test → Create test event
```

### Code Example - Python

```python
# lambda_function.py
import json

def lambda_handler(event, context):
    """
    Lambda handler function
    
    Args:
        event: Input event data
        context: Runtime information
    
    Returns:
        Response object
    """
    print(f"Event: {json.dumps(event)}")
    
    # Extract data from event
    name = event.get('name', 'World')
    
    # Business logic
    message = f"Hello, {name}!"
    
    # Return response
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': message
        })
    }
```

### Code Example - Node.js

```javascript
// index.js
exports.handler = async (event) => {
    console.log('Event:', JSON.stringify(event));
    
    const name = event.name || 'World';
    const message = `Hello, ${name}!`;
    
    return {
        statusCode: 200,
        body: JSON.stringify({
            message: message
        })
    };
};
```

### Via CLI

```bash
# Create function
aws lambda create-function \
  --function-name hello-lambda \
  --runtime python3.11 \
  --role arn:aws:iam::123456789:role/lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip

# Invoke function
aws lambda invoke \
  --function-name hello-lambda \
  --payload '{"name":"Claude"}' \
  response.json

# View response
cat response.json
```

---

## 🔧 Configuration

### Memory

```yaml
Memory: 128 MB - 10,240 MB (10 GB)
Increment: 1 MB

Memory determines:
- RAM available
- CPU power (proportional)
- Network bandwidth (proportional)
- Ephemeral storage (/tmp)

Example:
  128 MB → 0.08 vCPU equivalent
  1,792 MB → 1 full vCPU
  10,240 MB → ~6 vCPUs
```

### Timeout

```yaml
Timeout: 1 second - 15 minutes (900 seconds)

Default: 3 seconds

⚠️ If function runs longer → terminated
```

### Environment Variables

```yaml
Environment variables:
  DATABASE_URL: postgres://...
  API_KEY: abc123xyz  # ⚠️ Use Secrets Manager instead
  REGION: us-east-1
  LOG_LEVEL: INFO

# Access in code (Python)
import os
db_url = os.environ['DATABASE_URL']

# Access in code (Node.js)
const dbUrl = process.env.DATABASE_URL;
```

### Ephemeral Storage (/tmp)

```yaml
Ephemeral storage: 512 MB - 10,240 MB

Location: /tmp
Lifecycle: Per execution environment
Reuse: Possible between invocations (warm start)

Use cases:
- Download files
- Cache data
- Temporary processing
```

---

## 🎪 Triggers (Event Sources)

### API Gateway

HTTP(S) requests trigger Lambda.

```yaml
API Gateway → Lambda

Example:
  GET /users → list-users-function
  POST /users → create-user-function
  GET /users/{id} → get-user-function
```

```python
def lambda_handler(event, context):
    # API Gateway event structure
    http_method = event['requestContext']['http']['method']
    path = event['requestContext']['http']['path']
    headers = event['headers']
    body = json.loads(event['body']) if event.get('body') else {}
    
    if http_method == 'GET':
        return {
            'statusCode': 200,
            'body': json.dumps({'users': get_users()})
        }
```

### S3 Events

Object operations trigger Lambda.

```yaml
S3 Bucket: my-photos

Events:
  - s3:ObjectCreated:* → process-image-lambda
  - s3:ObjectRemoved:* → cleanup-lambda

Filter:
  Prefix: uploads/
  Suffix: .jpg
```

```python
def lambda_handler(event, context):
    # Extract S3 info
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    print(f"Processing {key} from {bucket}")
    
    # Download and process image
    s3 = boto3.client('s3')
    response = s3.get_object(Bucket=bucket, Key=key)
    image_data = response['Body'].read()
    
    # Process image...
    thumbnail = create_thumbnail(image_data)
    
    # Upload thumbnail
    s3.put_object(
        Bucket=bucket,
        Key=f"thumbnails/{key}",
        Body=thumbnail
    )
```

### DynamoDB Streams

Database changes trigger Lambda.

```yaml
DynamoDB Table: orders

Stream: Enabled (New and old images)

Lambda: process-order
  Batch size: 100
  Starting position: LATEST
```

```python
def lambda_handler(event, context):
    for record in event['Records']:
        if record['eventName'] == 'INSERT':
            new_item = record['dynamodb']['NewImage']
            order_id = new_item['orderId']['S']
            
            # Process new order
            send_confirmation_email(order_id)
```

### SQS Queue

Messages from queue trigger Lambda.

```yaml
SQS Queue: tasks-queue

Lambda: process-task
  Batch size: 10
  Batch window: 5 seconds
  Concurrent batches: 5
```

```python
def lambda_handler(event, context):
    for record in event['Records']:
        message_body = json.loads(record['body'])
        task_id = message_body['taskId']
        
        # Process task
        result = process_task(task_id)
        
        # Message is auto-deleted if function succeeds
```

### EventBridge (CloudWatch Events)

Scheduled or event-driven triggers.

```yaml
EventBridge Rule: daily-backup

Schedule: cron(0 2 * * ? *)  # 2 AM daily

Target: backup-lambda
```

```python
def lambda_handler(event, context):
    # Scheduled event
    print("Running daily backup")
    
    backup_databases()
    cleanup_old_backups()
    
    return {'status': 'success'}
```

### Other Triggers

```yaml
- SNS (Simple Notification Service)
- Kinesis Streams
- Application Load Balancer
- CloudFront (Lambda@Edge)
- Cognito
- Alexa Skills Kit
- IoT Rules
- CloudFormation
```

---

## 🔐 Permissions

### Execution Role

IAM role que Lambda asume para acceder a otros servicios.

```yaml
Execution Role: lambda-s3-dynamo-role

Trust policy: (Quién puede asumir el role)
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "lambda.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }]
}

Permissions policy: (Qué puede hacer)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:*:*:table/MyTable"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

### Resource-based Policy

Control quién puede invocar tu Lambda.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "s3.amazonaws.com"
    },
    "Action": "lambda:InvokeFunction",
    "Resource": "arn:aws:lambda:us-east-1:123456789:function:my-function",
    "Condition": {
      "StringEquals": {
        "AWS:SourceAccount": "123456789"
      },
      "ArnLike": {
        "AWS:SourceArn": "arn:aws:s3:::my-bucket"
      }
    }
  }]
}
```

---

## 📦 Packaging

### ZIP Archive

```bash
# Simple function (single file)
zip function.zip lambda_function.py

# With dependencies
pip install requests -t .
zip -r function.zip .

# Upload
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip
```

### Layers

Reusable code/libraries shared across functions.

```yaml
Layer: numpy-pandas-layer

Content:
  python/
    lib/
      python3.11/
        site-packages/
          numpy/
          pandas/

Version: 1
Size: 150 MB
```

```bash
# Create layer
zip -r layer.zip python/
aws lambda publish-layer-version \
  --layer-name numpy-pandas \
  --zip-file fileb://layer.zip \
  --compatible-runtimes python3.11

# Attach to function
aws lambda update-function-configuration \
  --function-name my-function \
  --layers arn:aws:lambda:us-east-1:123456:layer:numpy-pandas:1
```

**Benefits:**
- Reduce deployment package size
- Share code across functions
- Separate dependencies from function code
- AWS managed layers (boto3, numpy, etc.)

### Container Images

```dockerfile
# Dockerfile
FROM public.ecr.aws/lambda/python:3.11

# Copy function code
COPY lambda_function.py ${LAMBDA_TASK_ROOT}

# Install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"

# Set handler
CMD ["lambda_function.lambda_handler"]
```

```bash
# Build and push
docker build -t my-lambda .
docker tag my-lambda:latest 123456.dkr.ecr.us-east-1.amazonaws.com/my-lambda:latest
docker push 123456.dkr.ecr.us-east-1.amazonaws.com/my-lambda:latest

# Create function
aws lambda create-function \
  --function-name my-function \
  --package-type Image \
  --code ImageUri=123456.dkr.ecr.us-east-1.amazonaws.com/my-lambda:latest \
  --role arn:aws:iam::123456:role/lambda-role
```

**Max sizes:**
- ZIP: 50 MB (compressed), 250 MB (uncompressed)
- Container: 10 GB

---

## ⚡ Performance

### Cold Start vs Warm Start

```yaml
Cold Start:
  - First invocation or after idle period
  - AWS provisions execution environment
  - Loads code and dependencies
  - Runs initialization code
  - Latency: 100ms - several seconds

Warm Start:
  - Reuses existing execution environment
  - Skips initialization
  - Latency: <10ms

Factors affecting cold start:
  - Runtime (Interpreted > Compiled)
  - Package size
  - VPC configuration
  - Memory allocation
```

**Minimize cold starts:**
```python
# Initialize outside handler (runs once per container)
import boto3

# ✅ Initialize here (cold start only)
s3 = boto3.client('s3')
db_connection = create_db_connection()

def lambda_handler(event, context):
    # ❌ Don't initialize here (every invocation)
    
    # Reuse connections
    s3.get_object(Bucket='my-bucket', Key='file.txt')
    db_connection.execute(query)
```

### Provisioned Concurrency

Pre-warmed execution environments.

```yaml
Provisioned Concurrency: 10

Benefits:
- No cold starts
- Consistent latency
- Always ready

Cost: $0.015/hour per concurrent execution
      + invocation costs

Use case: Latency-sensitive apps (APIs, gaming)
```

### Memory and CPU

```
More memory = More CPU = Faster execution = Lower duration cost

Example processing 1,000 images:
  128 MB → 300 seconds → $0.000629
  1,024 MB → 50 seconds → $0.000835
  3,008 MB → 20 seconds → $0.001003
  
Sweet spot: Often around 1,024-1,792 MB
```

---

## 🔄 Concurrency

### Concurrent Executions

```yaml
Account limit: 1,000 concurrent executions (default)
  Can request increase

Function level:
  - Reserved concurrency: Guarantee for this function
  - Unreserved pool: Shared across all functions

Example:
  Total: 1,000
  - Function A reserved: 100
  - Function B reserved: 200
  - Unreserved pool: 700 (for all other functions)
```

### Scaling

```
Lambda auto-scales:
- First minute: +3,000 concurrent executions
- After: +500/minute

Example burst:
  Second 1: 1,000 invocations
  Second 2: 4,000 invocations (scales +3,000)
  Minute 2: 8,500 invocations (scales +500/min)
```

### Throttling

```yaml
Throttle occurs when:
- Exceed concurrent execution limit
- Reserved concurrency limit reached

Response:
  - Synchronous: 429 TooManyRequestsException
  - Asynchronous: Retry automatically (2x)
  - Stream: Retry until data expires
```

---

## 📊 Monitoring

### CloudWatch Logs

```python
import json

def lambda_handler(event, context):
    # Automatic: Request ID, duration, memory used
    
    # Custom logs
    print("Processing order")  # → CloudWatch Logs
    print(json.dumps({'orderId': 123, 'status': 'processing'}))
    
    # Structured logging (recommended)
    import logging
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    logger.info('Order processed', extra={'orderId': 123})
```

### CloudWatch Metrics

```yaml
Default metrics:
  - Invocations: Count
  - Duration: Milliseconds
  - Errors: Count
  - Throttles: Count
  - ConcurrentExecutions: Count
  - UnreservedConcurrentExecutions: Count

# View metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=my-function \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z \
  --period 3600 \
  --statistics Sum
```

### X-Ray Tracing

```yaml
Active tracing: Enabled

Traces:
  - Function execution
  - AWS SDK calls
  - HTTP requests
  - SQL queries

# In code (Python)
from aws_xray_sdk.core import xray_recorder

@xray_recorder.capture('process_order')
def process_order(order_id):
    # Traced automatically
    pass

def lambda_handler(event, context):
    process_order(event['orderId'])
```

---

## 💰 Pricing

```yaml
Pricing dimensions:
1. Number of requests
2. Duration (GB-seconds)

Requests:
  First 1M/month: Free
  After: $0.20 per 1M requests

Duration:
  First 400,000 GB-seconds/month: Free
  After: $0.00001667 per GB-second

GB-second = Memory (GB) × Duration (seconds)
```

**Examples:**
```yaml
Example 1: Simple API
  Memory: 128 MB (0.125 GB)
  Duration: 200 ms (0.2 seconds)
  Invocations: 5 million/month
  
  Requests: 5M × $0.20/1M = $1.00
  Duration: (5M - 1M) × 0.125 × 0.2 × $0.00001667 = $1.67
  Total: $2.67/month

Example 2: Image Processing
  Memory: 1,024 MB (1 GB)
  Duration: 5 seconds
  Invocations: 100,000/month
  
  Requests: 100K × $0.20/1M = $0.02
  Duration: 100K × 1 × 5 × $0.00001667 = $8.34
  Total: $8.36/month

Example 3: Data Processing
  Memory: 3,008 MB (3 GB)
  Duration: 30 seconds
  Invocations: 10,000/month
  
  Requests: 10K × $0.20/1M = $0.002
  Duration: 10K × 3 × 30 × $0.00001667 = $15.00
  Total: $15.00/month
```

---

## 📋 Best Practices

### Code

```python
✅ Initialize outside handler
✅ Reuse connections
✅ Use environment variables
✅ Implement idempotency
✅ Handle partial failures
✅ Use structured logging

# Good
import boto3
s3 = boto3.client('s3')  # Initialize once

def lambda_handler(event, context):
    s3.get_object(...)  # Reuse client

# Bad
def lambda_handler(event, context):
    import boto3  # ❌ Every invocation
    s3 = boto3.client('s3')
```

### Performance

```yaml
✅ Right-size memory
✅ Minimize cold starts
✅ Use Provisioned Concurrency for critical
✅ Optimize dependencies (layers)
✅ Consider container images for large deps
✅ Avoid VPC if not needed (cold start penalty)
```

### Security

```yaml
✅ Least privilege IAM roles
✅ Don't hardcode secrets (use Secrets Manager/Parameter Store)
✅ Encrypt environment variables
✅ Enable X-Ray tracing
✅ Regular dependency updates
✅ Input validation
```

### Reliability

```yaml
✅ Set appropriate timeout
✅ Dead Letter Queue (DLQ) for async
✅ Retry logic for external APIs
✅ Circuit breaker pattern
✅ Idempotency (handle duplicate invocations)
```

---

## 🐛 Troubleshooting

### Function Timeout

```python
# Problem: Function times out

# Solution 1: Increase timeout
aws lambda update-function-configuration \
  --function-name my-function \
  --timeout 60

# Solution 2: Optimize code
# - Add indexes to database
# - Increase memory (more CPU)
# - Async processing for long tasks

# Solution 3: Split into smaller functions
```

### Out of Memory

```bash
# Error: Runtime exited with error: signal: killed
# Memory: 128 MB → Not enough

# Solution: Increase memory
aws lambda update-function-configuration \
  --function-name my-function \
  --memory-size 512
```

### Cold Start Latency

```yaml
# Solutions:
1. Provisioned Concurrency
2. Increase memory (faster initialization)
3. Reduce package size
4. Remove VPC if not needed
5. Keep function warm (scheduled ping)
```

---

## 📚 Próximos Pasos

Con Lambda dominado:

1. **Step Functions** → Orchestrate Lambdas
2. **API Gateway** → HTTP APIs + Lambda
3. **EventBridge** → Event-driven architecture
4. **SAM / Serverless Framework** → IaC for serverless

---

[⬅️ Volver: RDS](./rds.md) | [➡️ Siguiente: IAM](./iam.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
