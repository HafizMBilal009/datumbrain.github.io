---
layout: post
title: Localstack / AWS Services / Go
author: Fahad Siddiqui
authorUrl: https://github.com/fahadsiddiqui
date: "2020-11-20 18:56:41"
---

## Pre-requisites

Before following the article, you should have

1. `python` & `pip`
2. `docker.io`
3. `docker-compose`
4. `go`
5. AWS CLI or `awslocal`

Installed on your system.

> The reason why `awslocal` makes it a lot easier to play with AWS commands is, you don't have to specify `--endpoint-url` each and everytime you need to
> access, invoke or create a AWS resource.
>
> Thanks to [localstack.cloud](https://localstack.cloud) for creating this utility for us.
>
> ```
> pip install awscli-local
> ```

## Example Project

### Lambda Function

Let's write Lambda function that responds with a JSON response.

```go
package main

import (
	"context"
	"encoding/json"
	"github.com/aws/aws-lambda-go/events"
	"github.com/aws/aws-lambda-go/lambda"
	"log"
	"time"
)

type response struct {
	UTC time.Time `json:"utc"`
}

func handleRequest(ctx context.Context, request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
	now := time.Now()
	resp := &response{
		UTC: now.UTC(),
	}
	body, err := json.Marshal(resp)
	if err != nil {
		return events.APIGatewayProxyResponse{}, err
	}
	return events.APIGatewayProxyResponse{Body: string(body), StatusCode: 200}, nil
}

func main() {
	lambda.Start(handleRequest)
}
```

You can probably put this code into a `main.go` and then build your project.

With Go modules, it is even simpler i.e; you don't have to manage `GOPATH` separately.

```bash
mkdir mylambda/
cd mylambda/
go mod init mylambda
touch main.go         # and put the code above, in that file
go mod download       # will fetch all dependencies
```

### Localstack

#### Install Localstack

```bash
pip install localstack
```

#### Run Localstack

```bash
localstack start
```

If you need services like AWS Lambda, AWS API Gateway, or may be S3 - you may pass an environment variable

```bash
SERVICES=lambda,apigateway,s3 localstack start
```

#### Run Localstack Using Docker Compose

You can define your own docker-compose file to run localstack

```yml
version: "3.3"

services:
  mylocalstack:
    container_name: mylocalstack
    image: localstack/localstack
    network_mode: bridge
    ports:
      - "4566:4566"
      - "4571:4571"
      - "${PORT_WEB_UI-8080}:${PORT_WEB_UI-8080}"
    environment:
      - DATA_DIR=${DATA_DIR- }
      - PORT_WEB_UI=${PORT_WEB_UI- }
      - LAMBDA_EXECUTOR=${LAMBDA_EXECUTOR- }
      - KINESIS_ERROR_PROBABILITY=${KINESIS_ERROR_PROBABILITY- }
      - DOCKER_HOST=unix:///var/run/docker.sock
      - HOST_TMP_FOLDER=${TMPDIR}
      - LOCALSTACK_API_KEY=${LOCALSTACK_API_KEY}
      - DEBUG=${DEBUG- }
      - SERVICES=${SERVICES- }
    volumes:
      - "${TMPDIR:-/tmp/localstack}:/tmp/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
```

You can now run the container

```bash
docker-compose up -d
```

> In macOS, you will have to run it using
>
> ```bash
> TMPDIR=/private$TMPDIR docker-compose up -d
> ```
>
> Otherwise it won't work.

### Installing AWS CLI

```bash
pip install awscli-local
```

### Setup AWS Infrastructure Locally

```bash
#!/usr/bin/env bash

GRACE_TIME="20s"
REGION="us-east-1"
STAGE="test"
API_NAME=api_$(date +"%Y%m%d_%H%M%S")
DOCKER_FILE="docker-compose.yml"
ENV_FILE="./.env"
LOCALSTACK_ENDPOINT="http://localhost:4566"

function fail() {
    echo $2
    exit $1
}

if [[ -f $ENV_FILE ]]; then
    echo "Sourcing environment variables..."
    source $ENV_FILE
else
    fail 9 "$ENV_FILE not present..."
fi

echo "Building lambda..."
GOOS=linux go build -o api ./ && \
    zip api.zip api && \
    rm -rf api

echo "Removing old containers..."
docker-compose -f ${DOCKER_FILE} down --remove-orphans

echo "Building new localstack environment..."
docker-compose -f ${DOCKER_FILE} up -d

echo "Grace time for $GRACE_TIME..."
sleep $GRACE_TIME && echo "Grace time ended..."

echo "Generated API name: $API_NAME"

awslocal elasticache create-cache-cluster \
    --cache-cluster-id "testcenters_cache" \
    --engine redis \
    --cache-node-type cache.m5.large \
    --num-cache-nodes 1

echo "Creating lambda function..."
awslocal lambda create-function \
    --region ${REGION} \
    --function-name ${API_NAME} \
    --runtime go1.x \
    --handler api \
    --timeout 30 \
    --memory-size 512 \
    --zip-file fileb://api.zip \
    --role arn:aws:iam::123456:role/irrelevant

[ $? == 0 ] || fail 1 "Failed: AWS / lambda / create-function"

LAMBDA_ARN=$(awslocal lambda list-functions --query "Functions[?FunctionName==\`${API_NAME}\`].FunctionArn" --output text --region ${REGION})

echo "Creating REST API..."
awslocal apigateway create-rest-api \
    --region ${REGION} \
    --name ${API_NAME}

[ $? == 0 ] || fail 2 "Failed: AWS / apigateway / create-rest-api"

API_ID=$(awslocal apigateway get-rest-apis --query "items[?name==\`${API_NAME}\`].id" --output text --region ${REGION})
PARENT_RESOURCE_ID=$(awslocal apigateway get-resources --rest-api-id ${API_ID} --query 'items[?path==`/`].id' --output text --region ${REGION})

awslocal apigateway create-resource \
    --region ${REGION} \
    --rest-api-id ${API_ID} \
    --parent-id ${PARENT_RESOURCE_ID} \
    --path-part "{somethingId}"

[ $? == 0 ] || fail 3 "Failed: AWS / apigateway / create-resource"

RESOURCE_ID=$(awslocal apigateway get-resources --rest-api-id ${API_ID} --query 'items[?path==`/{somethingId}`].id' --output text --region ${REGION})

awslocal apigateway put-method \
    --region ${REGION} \
    --rest-api-id ${API_ID} \
    --resource-id ${RESOURCE_ID} \
    --http-method GET \
    --request-parameters "method.request.path.somethingId=true" \
    --authorization-type "NONE" \

[ $? == 0 ] || fail 4 "Failed: AWS / apigateway / put-method"

echo "Creating REST API integration..."
awslocal apigateway put-integration \
    --region ${REGION} \
    --rest-api-id ${API_ID} \
    --resource-id ${RESOURCE_ID} \
    --http-method GET \
    --type AWS_PROXY \
    --integration-http-method POST \
    --uri arn:aws:apigateway:${REGION}:lambda:path/2015-03-31/functions/${LAMBDA_ARN}/invocations \
    --passthrough-behavior WHEN_NO_MATCH \

[ $? == 0 ] || fail 5 "Failed: AWS / apigateway / put-integration"

echo "Creating REST API deployment..."
awslocal apigateway create-deployment \
    --region ${REGION} \
    --rest-api-id ${API_ID} \
    --stage-name ${STAGE} \

[ $? == 0 ] || fail 6 "Failed: AWS / apigateway / create-deployment"

ENDPOINT=$LOCALSTACK_ENDPOINT/restapis/${API_ID}/${STAGE}/_user_request_

echo "API endpoint:"
echo ${ENDPOINT}

echo -e "\nAll good..."

```

## Resources

1. Thanks to this amazing [gist](https://gist.github.com/crypticmind/c75db15fd774fe8f53282c3ccbe3d7ad)
2. [Localstack's AWS CLI](https://github.com/localstack/awscli-local)
3. AWS documentation
