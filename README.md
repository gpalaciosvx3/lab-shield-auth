<p align="center">
  <a href="https://gustavopalacios.dev">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/gpalaciosvx3/gpalaciosvx3/master/assets/brand/logo-dark.svg">
      <img src="https://raw.githubusercontent.com/gpalaciosvx3/gpalaciosvx3/master/assets/brand/logo-light.svg" alt="Gustavo Palacios" height="64">
    </picture>
  </a>
</p>

<p align="center">
  <a href="https://gustavopalacios.dev"><img src="https://img.shields.io/badge/web-gustavopalacios.dev-17a267?logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCA2NCA2NCI%2BPGRlZnM%2BPG1hc2sgaWQ9Im0iPjxwYXRoIGQ9Ik0zMiAyIEw1OCAxNyBMNTggNDcgTDMyIDYyIEw2IDQ3IEw2IDE3IFoiIGZpbGw9IiNmZmYiLz48cGF0aCBkPSJNMjggMTQgTDE3LjUgNTAgTTI4IDE0IEw0Ni41IDUwIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iNC44IiBzdHJva2UtbGluZWNhcD0icm91bmQiLz48Y2lyY2xlIGN4PSIyOCIgY3k9IjE0IiByPSI0LjYiIGZpbGw9IiMwMDAiLz48L21hc2s%2BPC9kZWZzPjxyZWN0IHdpZHRoPSI2NCIgaGVpZ2h0PSI2NCIgZmlsbD0id2hpdGUiIG1hc2s9InVybCgjbSkiLz48L3N2Zz4%3D" alt="Web"></a>
  <a href="https://www.npmjs.com/org/gpkit"><img src="https://img.shields.io/badge/npm-%40gpkit-CB3837?logo=npm&logoColor=white" alt="npm @gpkit"></a>
  <a href="https://www.linkedin.com/in/gustavopalaciosv"><img src="https://img.shields.io/badge/LinkedIn-gustavopalaciosv-0A66C2?logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0id2hpdGUiPjxwYXRoIGQ9Ik0yMC40NSAyMC40NWgtMy41NnYtNS41N2MwLTEuMzMtLjAzLTMuMDQtMS44NS0zLjA0LTEuODYgMC0yLjE0IDEuNDUtMi4xNCAyLjk0djUuNjdIOS4zNVY5aDMuNDF2MS41NmguMDVjLjQ4LS45IDEuNjQtMS44NSAzLjM3LTEuODUgMy42IDAgNC4yNyAyLjM3IDQuMjcgNS40NnY2LjI4ek01LjM0IDcuNDNhMi4wNiAyLjA2IDAgMSAxIDAtNC4xMyAyLjA2IDIuMDYgMCAwIDEgMCA0LjEzek03LjEyIDIwLjQ1SDMuNTZWOWgzLjU2djExLjQ1eiIvPjwvc3ZnPg%3D%3D" alt="LinkedIn"></a>
  <a href="https://github.com/gpalaciosvx3"><img src="https://img.shields.io/badge/GitHub-gpalaciosvx3-181717?logo=github&logoColor=white" alt="GitHub"></a>
</p>

# lab-shield-auth

Servicio centralizado de autenticación y autorización. Expone una API HTTP para login, refresh y logout de usuarios, y un Lambda Authorizer que valida tokens JWT en el API Gateway antes de que lleguen a otros servicios.

---

## Índice

- [Arquitectura](#arquitectura)
- [API Reference](#api-reference)
  - [Endpoints](#endpoints)
  - [Códigos de error](#códigos-de-error)
- [Instalación y desarrollo local](#instalación-y-desarrollo-local)
- [CI/CD](#cicd)
  - [Pipelines](#pipelines)
  - [Secretos requeridos](#secretos-requeridos)

---

## Arquitectura

![Arquitectura](./docs/architecture.png)

### Recursos AWS

| Recurso | Nombre | Descripción |
|---|---|---|
| API Gateway HTTP | `UE1SHIELDAUTHGTW001` | Entry point HTTP |
| Lambda `auth-service` | `UE1SHIELDAUTHLMB001` | Login / Refresh / Logout |
| Lambda `authorizer` | `UE1SHIELDAUTHLMB002` | Validación JWT + blacklist |
| DynamoDB `users` | `UE1SHIELDAUTHDDB001` | Tabla de usuarios |
| DynamoDB `refresh-tokens` | `UE1SHIELDAUTHDDB002` | Refresh tokens activos |
| ElastiCache Redis | — | Rate limiting + blacklist de tokens |
| IAM Role | `UE1SHIELDAUTHROL001` | Rol de ejecución compartido |

---

## API Reference

### Endpoints

#### POST `/auth/login`

Autentica un usuario y devuelve un par de tokens.

**Body:**
```json
{ "email": "user@example.com", "password": "secret" }
```

**Response `200`:**
```json
{
  "data": {
    "accessToken": "<jwt>",
    "refreshToken": "<uuid>",
    "expiresIn": 900
  }
}
```

---

#### POST `/auth/refresh`

Rota el refresh token y devuelve un nuevo par.

**Body:**
```json
{ "refreshToken": "<uuid>" }
```

**Response `200`:**
```json
{
  "data": {
    "accessToken": "<jwt>",
    "refreshToken": "<uuid>",
    "expiresIn": 900
  }
}
```

---

#### POST `/auth/logout`

Revoca el refresh token y añade el access token a la blacklist.

**Headers:** `Authorization: Bearer <accessToken>`

**Body:**
```json
{ "refreshToken": "<uuid>" }
```

**Response `200`:**
```json
{
  "data": { "message": "Sesión cerrada correctamente" }
}
```

---

### Códigos de error

| Código | HTTP | Descripción |
|---|---|---|
| `APP-001` | 500 | Error inesperado |
| `APP-002` | 500 | Variable de entorno faltante |
| `APP-003` | 400 | Body de request inválido |
| `AUTH-001` | 401 | Credenciales inválidas |
| `AUTH-002` | 429 | Demasiados intentos fallidos |
| `AUTH-003` | 401 | Token inválido o expirado |
| `AUTH-004` | 401 | Refresh token ya fue rotado |
| `AUTH-005` | 401 | Usuario asociado al token no existe |

---

## Instalación y desarrollo local

```bash
# Instalar dependencias
npm install

# Tests unitarios
npm test
```

---

## Desarrollo en LocalStack

> Copiar `.env.example` a `.env` y completar `LOCALSTACK_AUTH_TOKEN`.

```bash
# Levantar LocalStack + Redis
docker compose up -d

# Instalar CLI (una sola vez)
npm install -g aws-cdk aws-cdk-local
pip install awscli-local

# Bootstrap + deploy
cd cdk
cdklocal bootstrap
cdklocal deploy --require-approval never
```

---

## Despliegue en AWS

```bash
# Bootstrap (una vez por cuenta/región)
cdk bootstrap aws://<ACCOUNT_ID>/us-east-1

# Preview de cambios
cdk diff

# Deploy
cdk deploy --require-approval never

# Destruir el stack
cdk destroy
```

> URL del endpoint: `https://{id}.execute-api.us-east-1.amazonaws.com/prod/v1/{path}`

---

## CI/CD

### Pipelines

| Archivo | Trigger | Acción |
|---|---|---|
| `deploy.yml` | `pull_request` / `push` a `master` | Build + Deploy en AWS |

### Secretos requeridos

Configurar en GitHub → Settings → Environments:

**`deployer`:**
```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
CDK_DEFAULT_ACCOUNT
AWS_DEFAULT_REGION
JWT_SECRET
```

**`deployer-local`:**
```
LOCALSTACK_AUTH_TOKEN
```

---