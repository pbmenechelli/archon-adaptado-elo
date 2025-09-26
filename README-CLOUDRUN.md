# Archon Backend - Cloud Run Deployment

Este documento explica como fazer o deploy dos serviços backend do Archon no Google Cloud Run.

##  Serviços a Serem Deployados

### 1. Archon Server (Porta 8080)
- **FastAPI application** com toda lógica de negócio
- **APIs REST** para frontend e integrações
- **WebSocket support** para real-time updates
- **Endpoints**: /api/* para todas as funcionalidades

### 2. Archon MCP Server (Porta 8080)
- **MCP Protocol server** para integração com AI assistants
- **14 tools** para knowledge management e task management
- **SSE transport** para comunicação com Cursor, Windsurf, etc.

##  Pré-requisitos

- Conta Google Cloud com billing habilitado
- Projeto GCP configurado
- gcloud CLI instalado e autenticado
- Supabase project criado e configurado

##  Passo a Passo do Deployment

### 1. Configuração Inicial

`ash
# Clone ou prepare o repositório
git clone <your-repo-url> archon-backend
cd archon-backend

# Configure as variáveis de ambiente
cp .env.cloudrun .env.production
# Edite .env.production com suas chaves reais
`

### 2. Configuração do Supabase

`sql
-- Execute no SQL Editor do Supabase
-- File: migration/complete_setup.sql
-- Isso criará todas as tabelas necessárias
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

##  Configuração das Variáveis de Ambiente

### Variáveis Essenciais
`ash
SUPABASE_URL=https://your-project-ref.supabase.co
SUPABASE_SERVICE_KEY=your-service-role-key-here
LOG_LEVEL=INFO
CORS_ORIGINS=https://your-lovable-domain.lovable.app
`

### Variáveis Opcionais
`ash
LOGFIRE_TOKEN=your-logfire-token
REDIS_URL=redis://your-redis-instance:6379
DB_POOL_SIZE=10
RATE_LIMIT_REQUESTS=100
`

##  URLs dos Serviços

Após o deploy, você terá:

- **Archon Server**: https://archon-server-xxxxxxxxxxxxx.run.app
- **Archon MCP**: https://archon-mcp-xxxxxxxxxxxxx.run.app

### Configuração do Frontend

Atualize o .env do frontend no Lovable:
`env
VITE_ARCHON_SERVER_URL=https://archon-server-xxxxxxxxxxxxx.run.app
VITE_ARCHON_MCP_URL=https://archon-mcp-xxxxxxxxxxxxx.run.app
`

##  Testes Pós-Deploy

### 1. Health Checks
`ash
# Test Server
curl https://archon-server-xxxxxxxxxxxxx.run.app/health

# Test MCP
curl https://archon-mcp-xxxxxxxxxxxxx.run.app/health
`

### 2. Funcionalidades Básicas
`ash
# Test API endpoints
curl https://archon-server-xxxxxxxxxxxxx.run.app/api/projects

# Test MCP tools (via Cursor ou interface web)
`

##  Monitoramento

### Google Cloud Console
- **Logs**: Cloud Logging para debugging
- **Metrics**: CPU, memória, latência
- **Traces**: Distributed tracing

### Alertas Recomendados
- CPU > 80% por 5 minutos
- Memória > 90%
- Latência > 1000ms
- 5xx errors > 1%

##  Troubleshooting

### Problemas Comuns

#### Build Falhando
`ash
# Verificar logs do build
gcloud builds list
gcloud builds log read BUILD_ID
`

#### Serviço Não Inicia
`ash
# Verificar logs do Cloud Run
gcloud run services logs read archon-server
`

#### CORS Errors
`ash
# Verificar configuração CORS
curl -H "Origin: https://your-lovable-domain.lovable.app" \
     https://archon-server-xxxxxxxxxxxxx.run.app/api/projects
`

#### Database Connection
`ash
# Verificar conectividade Supabase
# Logs devem mostrar "Database connected successfully"
`

##  Otimização de Custos

### Configurações Recomendadas
- **Memory**: 2Gi (server), 1Gi (MCP)
- **CPU**: 1 vCPU
- **Max instances**: 10 (server), 5 (MCP)
- **Concurrency**: 80 requests/instance
- **Timeout**: 300s

### Estratégias de Otimização
- **Auto-scaling**: Só paga pelo que usa
- **Spot instances**: Para workloads não críticos
- **Caching**: Implemente Redis para reduzir DB queries
- **CDN**: Use Cloud CDN para assets estáticos

##  Atualizações

### Deploy de Novas Versões
`ash
# Para atualizar sem downtime
gcloud run deploy archon-server --source . --platform managed --region us-central1

# Traffic splitting para canary deployments
gcloud run services update-traffic archon-server --to-latest --region us-central1
`

##  Suporte

Para problemas específicos do Cloud Run:
- [Documentação Cloud Run](https://cloud.google.com/run/docs)
- [Troubleshooting Guide](https://cloud.google.com/run/docs/troubleshooting)
- [Community Forums](https://cloud.google.com/run/docs/support)

Para problemas específicos do Archon:
- [Archon GitHub Issues](https://github.com/coleam00/Archon/issues)
- [Archon Discussions](https://github.com/coleam00/Archon/discussions)
