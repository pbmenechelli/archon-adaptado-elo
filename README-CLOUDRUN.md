# Archon Backend - Cloud Run Deployment

Este documento explica como fazer o deploy dos servi�os backend do Archon no Google Cloud Run.

##  Servi�os a Serem Deployados

### 1. Archon Server (Porta 8080)
- **FastAPI application** com toda l�gica de neg�cio
- **APIs REST** para frontend e integra��es
- **WebSocket support** para real-time updates
- **Endpoints**: /api/* para todas as funcionalidades

### 2. Archon MCP Server (Porta 8080)
- **MCP Protocol server** para integra��o com AI assistants
- **14 tools** para knowledge management e task management
- **SSE transport** para comunica��o com Cursor, Windsurf, etc.

##  Pr�-requisitos

- Conta Google Cloud com billing habilitado
- Projeto GCP configurado
- gcloud CLI instalado e autenticado
- Supabase project criado e configurado

##  Passo a Passo do Deployment

### 1. Configura��o Inicial

`ash
# Clone ou prepare o reposit�rio
git clone <your-repo-url> archon-backend
cd archon-backend

# Configure as vari�veis de ambiente
cp .env.cloudrun .env.production
# Edite .env.production com suas chaves reais
`

### 2. Configura��o do Supabase

`sql
-- Execute no SQL Editor do Supabase
-- File: migration/complete_setup.sql
-- Isso criar� todas as tabelas necess�rias
`

### 3. Build e Deploy do Server

`ash
# Deploy do Archon Server
gcloud run deploy archon-server \
  --source . \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --port 8080 \
  --memory 2Gi \
  --cpu 1 \
  --max-instances 10 \
  --timeout 300 \
  --concurrency 80 \
  --set-env-vars SUPABASE_URL=your-supabase-url \
  --set-env-vars SUPABASE_SERVICE_KEY=your-service-key \
  --set-env-vars LOG_LEVEL=INFO \
  --set-env-vars CORS_ORIGINS=https://your-lovable-domain.lovable.app \
  --dockerfile python/Dockerfile.server
`

### 4. Build e Deploy do MCP Server

`ash
# Deploy do Archon MCP
gcloud run deploy archon-mcp \
  --source . \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --port 8080 \
  --memory 1Gi \
  --cpu 1 \
  --max-instances 5 \
  --timeout 300 \
  --concurrency 80 \
  --set-env-vars SUPABASE_URL=your-supabase-url \
  --set-env-vars SUPABASE_SERVICE_KEY=your-service-key \
  --set-env-vars LOG_LEVEL=INFO \
  --dockerfile python/Dockerfile.mcp
`

##  Configura��o das Vari�veis de Ambiente

### Vari�veis Essenciais
`ash
SUPABASE_URL=https://your-project-ref.supabase.co
SUPABASE_SERVICE_KEY=your-service-role-key-here
LOG_LEVEL=INFO
CORS_ORIGINS=https://your-lovable-domain.lovable.app
`

### Vari�veis Opcionais
`ash
LOGFIRE_TOKEN=your-logfire-token
REDIS_URL=redis://your-redis-instance:6379
DB_POOL_SIZE=10
RATE_LIMIT_REQUESTS=100
`

##  URLs dos Servi�os

Ap�s o deploy, voc� ter�:

- **Archon Server**: https://archon-server-xxxxxxxxxxxxx.run.app
- **Archon MCP**: https://archon-mcp-xxxxxxxxxxxxx.run.app

### Configura��o do Frontend

Atualize o .env do frontend no Lovable:
`env
VITE_ARCHON_SERVER_URL=https://archon-server-xxxxxxxxxxxxx.run.app
VITE_ARCHON_MCP_URL=https://archon-mcp-xxxxxxxxxxxxx.run.app
`

##  Testes P�s-Deploy

### 1. Health Checks
`ash
# Test Server
curl https://archon-server-xxxxxxxxxxxxx.run.app/health

# Test MCP
curl https://archon-mcp-xxxxxxxxxxxxx.run.app/health
`

### 2. Funcionalidades B�sicas
`ash
# Test API endpoints
curl https://archon-server-xxxxxxxxxxxxx.run.app/api/projects

# Test MCP tools (via Cursor ou interface web)
`

##  Monitoramento

### Google Cloud Console
- **Logs**: Cloud Logging para debugging
- **Metrics**: CPU, mem�ria, lat�ncia
- **Traces**: Distributed tracing

### Alertas Recomendados
- CPU > 80% por 5 minutos
- Mem�ria > 90%
- Lat�ncia > 1000ms
- 5xx errors > 1%

##  Troubleshooting

### Problemas Comuns

#### Build Falhando
`ash
# Verificar logs do build
gcloud builds list
gcloud builds log read BUILD_ID
`

#### Servi�o N�o Inicia
`ash
# Verificar logs do Cloud Run
gcloud run services logs read archon-server
`

#### CORS Errors
`ash
# Verificar configura��o CORS
curl -H "Origin: https://your-lovable-domain.lovable.app" \
     https://archon-server-xxxxxxxxxxxxx.run.app/api/projects
`

#### Database Connection
`ash
# Verificar conectividade Supabase
# Logs devem mostrar "Database connected successfully"
`

##  Otimiza��o de Custos

### Configura��es Recomendadas
- **Memory**: 2Gi (server), 1Gi (MCP)
- **CPU**: 1 vCPU
- **Max instances**: 10 (server), 5 (MCP)
- **Concurrency**: 80 requests/instance
- **Timeout**: 300s

### Estrat�gias de Otimiza��o
- **Auto-scaling**: S� paga pelo que usa
- **Spot instances**: Para workloads n�o cr�ticos
- **Caching**: Implemente Redis para reduzir DB queries
- **CDN**: Use Cloud CDN para assets est�ticos

##  Atualiza��es

### Deploy de Novas Vers�es
`ash
# Para atualizar sem downtime
gcloud run deploy archon-server --source . --platform managed --region us-central1

# Traffic splitting para canary deployments
gcloud run services update-traffic archon-server --to-latest --region us-central1
`

##  Suporte

Para problemas espec�ficos do Cloud Run:
- [Documenta��o Cloud Run](https://cloud.google.com/run/docs)
- [Troubleshooting Guide](https://cloud.google.com/run/docs/troubleshooting)
- [Community Forums](https://cloud.google.com/run/docs/support)

Para problemas espec�ficos do Archon:
- [Archon GitHub Issues](https://github.com/coleam00/Archon/issues)
- [Archon Discussions](https://github.com/coleam00/Archon/discussions)
