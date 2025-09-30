# GitHub Actions - Configuração de Secrets

Este diretório contém os workflows de CI/CD para deploy automático do Archon no Google Cloud Run.

## 🔐 Secrets Necessários

Configure os seguintes secrets no GitHub (Settings → Secrets and variables → Actions):

### 1. **GCP_PROJECT_ID**
- **Descrição**: ID do projeto Google Cloud
- **Como obter**: 
  ```bash
  gcloud config get-value project
  ```
- **Exemplo**: `elo-microservices-prod`

### 2. **GCP_SA_KEY**
- **Descrição**: Chave JSON da Service Account com permissões para Cloud Run
- **Como criar**:
  ```bash
  # 1. Criar service account
  gcloud iam service-accounts create archon-deployer \
    --display-name="Archon Cloud Run Deployer"
  
  # 2. Dar permissões necessárias
  gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:archon-deployer@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/run.admin"
  
  gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:archon-deployer@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/iam.serviceAccountUser"
  
  # 3. Criar e baixar chave
  gcloud iam service-accounts keys create key.json \
    --iam-account=archon-deployer@PROJECT_ID.iam.gserviceaccount.com
  
  # 4. Copiar conteúdo do key.json e adicionar como secret no GitHub
  ```

### 3. **SUPABASE_URL**
- **Descrição**: URL do projeto Supabase
- **Como obter**: Supabase Dashboard → Project Settings → API → Project URL
- **Exemplo**: `https://xxxxxxxxxxxxx.supabase.co`

### 4. **SUPABASE_SERVICE_KEY**
- **Descrição**: Service Role Key do Supabase
- **Como obter**: Supabase Dashboard → Project Settings → API → service_role key
- **⚠️ Atenção**: Nunca compartilhe essa chave!

### 5. **CORS_ORIGINS** (Opcional)
- **Descrição**: Domínios permitidos para CORS
- **Exemplo**: `https://your-lovable-domain.lovable.app`
- **Nota**: Configurar após deploy do frontend no Lovable

---

## 🚀 Workflows Disponíveis

### 1. `deploy-server.yml` - Deploy do Archon Server
- **Trigger**: Push na main com mudanças em `python/**`
- **Serviço**: `archon-server`
- **Recursos**: 2Gi RAM, 1 CPU, max 10 instances
- **Porta**: 8080

### 2. `deploy-mcp.yml` - Deploy do Archon MCP
- **Trigger**: Push na main com mudanças em `python/**`
- **Serviço**: `archon-mcp`
- **Recursos**: 1Gi RAM, 1 CPU, max 5 instances
- **Porta**: 8080

---

## 📋 Checklist de Configuração

- [ ] Criar projeto no Google Cloud (se ainda não existir)
- [ ] Habilitar APIs necessárias:
  ```bash
  gcloud services enable run.googleapis.com
  gcloud services enable cloudbuild.googleapis.com
  gcloud services enable artifactregistry.googleapis.com
  ```
- [ ] Criar Service Account e chave JSON
- [ ] Adicionar todos os secrets no GitHub
- [ ] Testar workflows com push manual ou workflow_dispatch

---

## 🧪 Testar Deploy Manual

Para testar localmente antes do CI/CD:

```bash
# 1. Autenticar no GCP
gcloud auth login

# 2. Configurar projeto
gcloud config set project YOUR_PROJECT_ID

# 3. Deploy manual do server
gcloud run deploy archon-server \
  --source . \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated

# 4. Deploy manual do MCP
gcloud run deploy archon-mcp \
  --source . \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

---

## 📊 Monitoramento

Após o deploy, monitore os serviços:

- **Cloud Run Console**: https://console.cloud.google.com/run
- **Logs**: Cloud Logging com label `service=archon`
- **Metrics**: CPU, memória, latência, requests/s

---

## 🔄 Atualizações

Para fazer deploy de novas versões:

1. Faça commit e push na branch `main`
2. Os workflows serão acionados automaticamente
3. Acompanhe o progresso em: Actions tab do GitHub
4. URLs dos serviços serão exibidas nos logs do workflow

---

## 🆘 Troubleshooting

### Build Falhando
- Verifique se todos os secrets estão configurados
- Confirme que a Service Account tem as permissões corretas
- Revise os logs do workflow no GitHub Actions

### Deploy Falhando
- Verifique se as APIs do GCP estão habilitadas
- Confirme que o projeto GCP está correto
- Revise os logs do Cloud Build

### Health Check Falhando
- Verifique se o endpoint `/health` está implementado
- Confirme que a porta 8080 está sendo usada
- Revise os logs do Cloud Run
