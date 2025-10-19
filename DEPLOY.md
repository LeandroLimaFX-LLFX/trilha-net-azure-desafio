# Guia de Deploy no Azure

## Pré-requisitos
- Conta Azure ativa
- Azure CLI instalado

## Passo 1: Criar recursos no Azure

### 1.1 Login no Azure
```bash
az login
```

### 1.2 Criar Resource Group
```bash
az group create --name rg-trilha-net-rh --location brazilsouth
```

### 1.3 Criar SQL Database
```bash
# Criar SQL Server
az sql server create \
  --name sql-trilha-net-rh \
  --resource-group rg-trilha-net-rh \
  --location brazilsouth \
  --admin-user sqladmin \
  --admin-password "SuaSenhaForte123!"

# Criar Database
az sql db create \
  --resource-group rg-trilha-net-rh \
  --server sql-trilha-net-rh \
  --name db-rh \
  --service-objective S0

# Permitir acesso do Azure
az sql server firewall-rule create \
  --resource-group rg-trilha-net-rh \
  --server sql-trilha-net-rh \
  --name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0
```

### 1.4 Criar Storage Account
```bash
az storage account create \
  --name strgtrilhanet \
  --resource-group rg-trilha-net-rh \
  --location brazilsouth \
  --sku Standard_LRS

# Criar Table
az storage table create \
  --name FuncionarioLogs \
  --account-name strgtrilhanet
```

### 1.5 Criar App Service
```bash
# Criar App Service Plan
az appservice plan create \
  --name plan-trilha-net-rh \
  --resource-group rg-trilha-net-rh \
  --sku B1 \
  --is-linux

# Criar Web App
az webapp create \
  --resource-group rg-trilha-net-rh \
  --plan plan-trilha-net-rh \
  --name trilha-net-azure-rh \
  --runtime "DOTNETCORE:6.0"
```

## Passo 2: Configurar Connection Strings

### 2.1 Obter Connection Strings
```bash
# SQL Connection String
az sql db show-connection-string \
  --client ado.net \
  --server sql-trilha-net-rh \
  --name db-rh

# Storage Connection String
az storage account show-connection-string \
  --name strgtrilhanet \
  --resource-group rg-trilha-net-rh
```

### 2.2 Configurar no App Service
```bash
# SQL Database
az webapp config connection-string set \
  --resource-group rg-trilha-net-rh \
  --name trilha-net-azure-rh \
  --settings DefaultConnection="Server=tcp:sql-trilha-net-rh.database.windows.net,1433;Database=db-rh;User ID=sqladmin;Password=SuaSenhaForte123!;Encrypt=true;" \
  --connection-string-type SQLAzure

# Storage Account
az webapp config appsettings set \
  --resource-group rg-trilha-net-rh \
  --name trilha-net-azure-rh \
  --settings ConnectionStrings__SAConnectionString="COLE_AQUI_A_CONNECTION_STRING_DO_STORAGE" \
              ConnectionStrings__AzureTableName="FuncionarioLogs"
```

## Passo 3: Deploy da Aplicação

### Opção A: Deploy via Azure CLI
```bash
cd workspace_repo
dotnet publish -c Release -o ./publish
cd publish
zip -r deploy.zip .
az webapp deployment source config-zip \
  --resource-group rg-trilha-net-rh \
  --name trilha-net-azure-rh \
  --src deploy.zip
```

### Opção B: Deploy via GitHub Actions
1. No portal Azure, vá até o App Service
2. Em "Deployment Center", selecione "GitHub"
3. Autorize e selecione o repositório
4. O workflow será criado automaticamente

## Passo 4: Executar Migrations

```bash
# Obter publish profile
az webapp deployment list-publishing-profiles \
  --name trilha-net-azure-rh \
  --resource-group rg-trilha-net-rh

# Ou executar migrations localmente apontando para o Azure SQL
dotnet ef database update
```

## Passo 5: Testar

Acesse: `https://trilha-net-azure-rh.azurewebsites.net/swagger`

### Endpoints disponíveis:
- GET /Funcionario/{id}
- POST /Funcionario
- PUT /Funcionario/{id}
- DELETE /Funcionario/{id}

## Custos Estimados (mensais)
- App Service (B1): ~R$ 50-80
- SQL Database (S0): ~R$ 60-100
- Storage Account: ~R$ 5-10
- **Total estimado**: R$ 115-190/mês

## Limpeza de Recursos
```bash
az group delete --name rg-trilha-net-rh --yes
```

## Troubleshooting

### Erro de conexão com SQL
- Verifique firewall rules do SQL Server
- Adicione seu IP: `az sql server firewall-rule create --resource-group rg-trilha-net-rh --server sql-trilha-net-rh --name MyIP --start-ip-address SEU_IP --end-ip-address SEU_IP`

### Erro ao salvar logs
- Verifique se a connection string do Storage está correta
- Confirme que a tabela "FuncionarioLogs" foi criada

### App não inicia
- Verifique logs: `az webapp log tail --name trilha-net-azure-rh --resource-group rg-trilha-net-rh`
