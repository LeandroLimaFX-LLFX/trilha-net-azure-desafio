# DIO - Trilha .NET - Nuvem com Microsoft Azure

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com)

## 📋 Sobre o Projeto

Sistema de RH desenvolvido como desafio da trilha .NET da DIO, implementando um CRUD completo de funcionários com armazenamento de logs no Azure Table Storage.

**Status**: ✅ Desafio concluído - Todos os TODOs implementados

## 🚀 Tecnologias Utilizadas

- .NET 6.0
- ASP.NET Core Web API
- Entity Framework Core
- Azure SQL Database
- Azure Table Storage
- Azure App Service

## ✨ Funcionalidades Implementadas

- ✅ Cadastro de funcionários (POST)
- ✅ Consulta por ID (GET)
- ✅ Atualização completa de dados (PUT)
- ✅ Exclusão de funcionários (DELETE)
- ✅ Log automático de todas as operações no Azure Table Storage
- ✅ Validação de dados com Entity Framework
- ✅ Documentação Swagger automática

## 🏗️ Arquitetura Azure

```
┌─────────────────┐
│  Azure App      │
│  Service        │
│  (Web API)      │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼────┐ ┌──▼──────────┐
│ Azure  │ │   Azure     │
│ SQL DB │ │   Table     │
│ (EF)   │ │   Storage   │
└────────┘ └─────────────┘
```

## 📦 Modelo de Dados

### Funcionario
```json
{
  "id": 0,
  "nome": "João Silva",
  "endereco": "Rua A, 123",
  "ramal": "1234",
  "emailProfissional": "joao@empresa.com",
  "departamento": "TI",
  "salario": 5000,
  "dataAdmissao": "2025-10-18T00:00:00Z"
}
```

### FuncionarioLog (Azure Table)
- **PartitionKey**: Departamento do funcionário
- **RowKey**: GUID único
- **TipoAcao**: Inclusao | Atualizacao | Remocao
- **JSON**: Snapshot completo do funcionário
- **Timestamp**: Data/hora automática

## 🔧 Configuração Local

### Pré-requisitos
- .NET 6.0 SDK
- SQL Server (LocalDB ou Azure)
- Conta Azure (para Table Storage)

### appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=RH;Trusted_Connection=True;",
    "SAConnectionString": "DefaultEndpointsProtocol=https;AccountName=SUA_CONTA;AccountKey=SUA_CHAVE;",
    "AzureTableName": "FuncionarioLogs"
  }
}
```

### Executar Migrations

```bash
dotnet ef database update
```

### Executar a API

```bash
dotnet run
```

Acesse: `https://localhost:5001/swagger`

## �� Deploy no Azure

📖 **Guia completo**: [DEPLOY.md](DEPLOY.md)

### Deploy rápido (5 minutos):

```bash
# 1. Login
az login

# 2. Criar recursos
az group create --name rg-trilha-net-rh --location brazilsouth

# 3. Deploy
az webapp up --name trilha-net-azure-rh --resource-group rg-trilha-net-rh --runtime "DOTNETCORE:6.0"
```

## 📚 API Endpoints

| Método | Endpoint | Descrição | Body |
|--------|----------|-----------|------|
| GET | `/Funcionario/{id}` | Busca funcionário | - |
| POST | `/Funcionario` | Cria funcionário | JSON |
| PUT | `/Funcionario/{id}` | Atualiza funcionário | JSON |
| DELETE | `/Funcionario/{id}` | Remove funcionário | - |

### Exemplo de uso:

```bash
# Criar funcionário
curl -X POST "https://trilha-net-azure-rh.azurewebsites.net/Funcionario" \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "João Silva",
    "endereco": "Rua A, 123",
    "ramal": "1234",
    "emailProfissional": "joao@empresa.com",
    "departamento": "TI",
    "salario": 5000,
    "dataAdmissao": "2025-10-18"
  }'

# Buscar funcionário
curl "https://trilha-net-azure-rh.azurewebsites.net/Funcionario/1"
```

## 🔍 Sistema de Logs

Todos os logs são **automaticamente** armazenados no Azure Table Storage:

- **Inclusao**: Quando um funcionário é criado
- **Atualizacao**: Quando dados são modificados
- **Remocao**: Quando um funcionário é deletado

Cada log contém:
- Snapshot completo do funcionário (JSON)
- Timestamp preciso
- Tipo da operação
- Departamento (para particionamento eficiente)

## 🧪 Testes Locais

### Usar Azurite (Storage Emulator)

```bash
# Instalar
npm install -g azurite

# Executar
azurite --silent --location ./azurite

# Connection string local
"DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;TableEndpoint=http://127.0.0.1:10002/devstoreaccount1;"
```

## 📊 Custos Estimados (Azure)

| Recurso | Tier | Custo/mês (estimado) |
|---------|------|---------------------|
| App Service | B1 | R$ 50-80 |
| SQL Database | S0 | R$ 60-100 |
| Storage Account | Standard | R$ 5-10 |
| **Total** | | **R$ 115-190** |

💡 **Dica**: Use o tier gratuito (F1) do App Service para testes!

## 🎯 Implementações Técnicas

### TODOs Completados:
- ✅ `_context.SaveChanges()` após Add/Update/Remove
- ✅ Propriedades completas no método `Atualizar()`
- ✅ `tableClient.UpsertEntity()` para logs
- ✅ Tratamento de valores null (`??` operator)
- ✅ Atributos `[FromBody]` nos parâmetros

### Código adicionado:
```csharp
// Criar
_context.SaveChanges();
tableClient.UpsertEntity(funcionarioLog);

// Atualizar
funcionarioBanco.Nome = funcionario.Nome;
funcionarioBanco.Ramal = funcionario.Ramal;
// ... todas as propriedades
_context.Funcionarios.Update(funcionarioBanco);
_context.SaveChanges();
tableClient.UpsertEntity(funcionarioLog);

// Deletar
_context.Funcionarios.Remove(funcionarioBanco);
_context.SaveChanges();
tableClient.UpsertEntity(funcionarioLog);
```

## 📝 Estrutura do Projeto

```
trilha-net-azure-desafio/
├── Controllers/
│   └── FuncionarioController.cs  ✅ Implementado
├── Context/
│   └── RHContext.cs
├── Models/
│   ├── Funcionario.cs
│   ├── FuncionarioLog.cs
│   └── TipoAcao.cs
├── Migrations/
├── .github/
│   └── workflows/
│       └── azure-deploy.yml      ✅ CI/CD
├── DEPLOY.md                      ✅ Guia de deploy
└── README.md                      ✅ Este arquivo
```

## 🚀 GitHub Actions (CI/CD)

Deploy automático configurado! Cada push na branch `main` dispara:
1. Build do projeto
2. Execução de testes
3. Deploy no Azure App Service

Veja: [.github/workflows/azure-deploy.yml](.github/workflows/azure-deploy.yml)

## 🐛 Troubleshooting

### Erro: "Connection timeout" (SQL)
```bash
# Adicionar seu IP ao firewall
az sql server firewall-rule create \
  --resource-group rg-trilha-net-rh \
  --server sql-trilha-net-rh \
  --name MeuIP \
  --start-ip-address SEU_IP \
  --end-ip-address SEU_IP
```

### Erro: "Table não encontrada"
```bash
# Criar tabela manualmente
az storage table create --name FuncionarioLogs --account-name SUA_CONTA
```

### Ver logs da aplicação
```bash
az webapp log tail --name trilha-net-azure-rh --resource-group rg-trilha-net-rh
```

## 🎓 Aprendizados

- ✅ Deploy de Web API .NET no Azure
- ✅ Integração com Azure SQL Database
- ✅ Uso de Azure Table Storage para logs
- ✅ Entity Framework Core migrations
- ✅ CI/CD com GitHub Actions
- ✅ Boas práticas de API REST

## 📚 Recursos Úteis

- [Documentação Azure App Service](https://learn.microsoft.com/azure/app-service/)
- [Azure Table Storage](https://learn.microsoft.com/azure/storage/tables/)
- [Entity Framework Core](https://learn.microsoft.com/ef/core/)
- [ASP.NET Core Web API](https://learn.microsoft.com/aspnet/core/web-api/)

## 📄 Licença

Projeto desenvolvido para fins educacionais - [Digital Innovation One](https://www.dio.me/)

## 👤 Autor

**Leandro Lima**
- GitHub: [@LeandroLimaFX-LLFX](https://github.com/LeandroLimaFX-LLFX)
- LinkedIn: [Conecte-se comigo](https://www.linkedin.com/in/leandrolima)

## 🙏 Agradecimentos

- **Digital Innovation One** - Bootcamp e desafio
- **Microsoft** - Plataforma Azure
- **Comunidade .NET** - Suporte e conhecimento

---

⭐ **Se este projeto te ajudou, deixe uma estrela!**

💬 **Dúvidas?** Abra uma [issue](https://github.com/LeandroLimaFX-LLFX/trilha-net-azure-desafio/issues)
