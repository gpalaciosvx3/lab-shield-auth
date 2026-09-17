# se-shield-auth — CDK

Infraestructura AWS del proyecto `Shield Auth`, definida con AWS CDK (TypeScript).

---

## Índice

- [Estructura](#estructura)
- [Requisitos](#requisitos)
- [Instalación](#instalación)
- [Desarrollo en LocalStack](#desarrollo-en-localstack)
- [Despliegue en AWS](#despliegue-en-aws)
- [Comandos de referencia](#comandos-de-referencia)
- [Recursos desplegados](#recursos-desplegados)
- [Outputs del stack](#outputs-del-stack)

---

## Estructura

```
cdk/
  bin/
    app.ts              # Entry point — resuelve stage → config → stack
  lib/
    app.stack.ts        # Stack principal
    constructs/
      api-gateway/      # REST API v1 con rutas auth
      cloudwatch/       # Log groups por Lambda
      dynamodb/         # Tablas Users y RefreshTokens
      elasticache/      # Cluster Redis (blacklist + rate-limit)
      iam/              # Rol de ejecución compartido
      lambda/
        auth/           # Lambda auth-service (login / refresh / logout)
        authorizer/     # Lambda Authorizer (validación JWT + blacklist)
        shared/         # bundling.config.ts compartido
      vpc/              # VPC con subnets privadas + VPC Endpoints
  common/
    constants/          # NamingConstants, ResourceConstants, InfraConstants
  docker-compose.yml    # LocalStack Pro para desarrollo local
```

---

## Requisitos

- Node.js 20+
- Docker (para LocalStack)
- `LOCALSTACK_AUTH_TOKEN` en `.env` (LocalStack Pro)

---

## Instalación

```bash
# Instalar dependencias CDK
cd cdk && npm install

# Instalar CLI global (una sola vez)
npm install -g aws-cdk aws-cdk-local
pip install awscli-local
```

---

## Desarrollo en LocalStack

> Copiar `.env.example` a `.env` y completar `LOCALSTACK_AUTH_TOKEN`.

```bash
# Levantar LocalStack (desde la raíz del proyecto)
docker compose up -d

# Bootstrap (una vez por contenedor)
cdklocal bootstrap

# Deploy
cdklocal deploy --require-approval never

# Preview de cambios
cdklocal diff

# Destruir
cdklocal destroy --force
```

### Scripts disponibles

```bash
npm run setup:local      # bootstrap + deploy
npm run deploy:local     # solo deploy
npm run diff:local       # diff
npm run destroy:local    # destruir stack
```

---

## Despliegue en AWS

```bash
# Bootstrap (una vez por cuenta/región)
cdk bootstrap aws://<ACCOUNT_ID>/us-east-1

# Preview
cdk diff

# Deploy
cdk deploy --require-approval never
```

---

## Comandos de referencia

### Verificar recursos en LocalStack

```bash
# API Gateway
awslocal apigateway get-rest-apis
awslocal apigateway get-stages --rest-api-id <api-id>

# Lambda
awslocal lambda list-functions --query 'Functions[*].FunctionName'

# Lambda — listar variables de entorno (auth-service)
awslocal lambda get-function-configuration \
  --function-name UE1SHIELDAUTHLMB001 \
  --query 'Environment.Variables'

# Lambda — listar variables de entorno (authorizer)
awslocal lambda get-function-configuration \
  --function-name UE1SHIELDAUTHLMB002 \
  --query 'Environment.Variables'

# DynamoDB — listar tablas
awslocal dynamodb list-tables

# DynamoDB — listar usuarios
awslocal dynamodb scan --table-name UE1SHIELDAUTHDDB001

# DynamoDB — listar refresh tokens
awslocal dynamodb scan --table-name UE1SHIELDAUTHDDB002

# ElastiCache
awslocal elasticache describe-cache-clusters
```

---

## Recursos desplegados

| Recurso | Construct | Nombre físico |
|---|---|---|
| Lambda auth-service | `AuthFnConstruct` | `UE1SHIELDAUTHLMB001` |
| Lambda Authorizer | `AuthorizerFnConstruct` | `UE1SHIELDAUTHLMB002` |
| API Gateway REST | `HttpApiConstruct` | `UE1SHIELDAUTHGTW001` |
| IAM Role | `WorkerRoleConstruct` | `UE1SHIELDAUTHROL001` |
| VPC | `ShieldVpcConstruct` | `UE1SHIELDAUTHVPC001` |
| Security Group Redis | `RedisClusterConstruct` | `UE1SHIELDAUTHSGP001` |
| Security Group Lambda | `ShieldVpcConstruct` | `UE1SHIELDAUTHSGP002` |
| ElastiCache Redis | `RedisClusterConstruct` | `UE1SHIELDAUTHECC001` |
| DynamoDB Users | `AuthTablesConstruct` | `UE1SHIELDAUTHDDB001` |
| DynamoDB RefreshTokens | `AuthTablesConstruct` | `UE1SHIELDAUTHDDB002` |

### Rutas expuestas

| Método | Ruta | Autorización |
|---|---|---|
| `POST` | `/v1/auth/login` | Ninguna |
| `POST` | `/v1/auth/refresh` | Ninguna |
| `POST` | `/v1/auth/logout` | Ninguna |

---

## Outputs del stack

| Output | Descripción |
|---|---|
| `ApiUrl` | URL base del API Gateway |
| `AuthorizerFunctionArn` | ARN del Lambda Authorizer — exportado como `shield-auth-authorizer-arn-{stage}` para uso en otros API Gateways |
