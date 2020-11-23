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
> pip install awslocal
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
      - "4510-4511:4510-4511"
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
