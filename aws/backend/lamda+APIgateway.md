# AWS Lambda + API Gateway backend deployment

### Architecture
Client (Browser / Postman)
        |
API Gateway (URL)
        |
AWS Lambda (Node.js code)

Amazon CLI 

```
aws --version
aws sts get-caller-identity
node -v
```
`export AWS_REGION=us-east-1`
Check:echo $AWS_REGION
expected :us-east-1

### step1 Create a project folder + Lambda code (no uploads)
Create project folder:
```
mkdir lambda-api
cd lambda-api
```
Create Lambda Function Code:
Create index.js:

```
cat > index.js << 'EOF'
exports.handler = async (event) => {
  return {
    statusCode: 200,
    headers: { "content-type": "application/json" },
    body: JSON.stringify({
      message: "Hello from Lambda API Backend",
      time: new Date().toISOString()
    })
  };
};
EOF 
```

Zip it:
`zip function.zip index.js`

### step2 Create an IAM Role for Lambda (CLI)
Create a trust policy file:
```
cat > trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "lambda.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```

### STEP 3: Zip Lambda Code + Create IAM Role
*Zip the Lambda code*:
`zip function.zip index.js`

check :ls
expected:index.js
function.zip

 *Create IAM Role for Lambda*
Create trust policy file
```
cat > trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "lambda.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```
*Create IAM role:*
```
aws iam create-role \
  --role-name lambda-basic-exec-role \
  --assume-role-policy-document file://trust-policy.json
```

*Attach logging permission*
```
aws iam attach-role-policy \
  --role-name lambda-basic-exec-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

*Get Role ARN:*
```
aws iam get-role \
  --role-name lambda-basic-exec-role \
  --query "Role.Arn" \
  --output text
```
📌 Copy the ARN value you see (starts with arn:aws:iam::).

### STEP 4: Create the Lambda Function:

replace arn value 
```
aws lambda create-function \
  --function-name backend-lambda \
  --runtime nodejs20.x \
  --handler index.handler \
  --role arn:aws:iam::066712929553:role/lambda-basic-exec-role \
  --zip-file fileb://function.zip \
  --region us-east-1
```

### STEP 5: Test Lambda Function
```
aws lambda invoke \
  --function-name backend-lambda \
  --payload '{}' \
  response.json
```

### STEP 6: Create API Gateway (HTTP API)
*Create HTTP API:*
```
aws apigatewayv2 create-api \
  --name backend-http-api \
  --protocol-type HTTP
```

*Connect API Gateway to Lambda*

save your API ID into a variable (replace id )
`echo $API_ID`

*Get Lambda ARN*
```
LAMBDA_ARN=$(aws lambda get-function \
  --function-name backend-lambda \
  --query "Configuration.FunctionArn" \
  --output text)
```

*Create API → Lambda integration*(replace arn id and intergarion id )
```
INTEGRATION_ID=$(aws apigatewayv2 create-integration \
  --api-id bc1zk930fb \
  --integration-type AWS_PROXY \
  --integration-uri arn:aws:lambda:us-east-1:066712929553:function:backend-lambda \
  --payload-format-version "2.0" \
  --query "IntegrationId" \
  --output text)
  ```
check for integration id : `echo $INTEGRATION_ID`

*Create route*(replace integration id)
```
aws apigatewayv2 create-route \
  --api-id bc1zk930fb \
  --route-key "GET /hello" \
  --target integrations/$INTEGRATION_ID
  
```

### STEP 8: Allow API Gateway to Invoke Lambda

*Add permission to Lambda:(replace account id)
```
aws lambda add-permission \
  --function-name backend-lambda \
  --statement-id apigw-permission \
  --action lambda:InvokeFunction \
  --principal apigateway.amazonaws.com \
  --source-arn "arn:aws:execute-api:us-east-1:$ACCOUNT_ID:bc1zk930fb/*/*"
```

### Step 9 in AWS API Gateway: Create Stage → Deploy → Test URL

create stage :







     
