# 📋 Task Definition - Configuração do Container N8N

## 🎯 **Visão Geral**

A Task Definition é o "blueprint" que define como o container N8N será executado no ECS. É equivalente ao `docker-compose.yml` mas para AWS.

## 🔧 **Configuração Completa**

```json
{
  "cpu": 1024,
  "memoryReservation": 819,
  "image": "n8nio/n8n:1.114.3",
  "essential": true,
  "environment": [
    {
      "name": "WEBHOOK_URL",
      "value": "https://SEU_DOMINIO/"
    },
    {
      "name": "DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED",
      "value": "false"
    },
    {
      "name": "DB_POSTGRESDB_DATABASE",
      "value": "n8n"
    },
    {
      "name": "DB_POSTGRESDB_PASSWORD",
      "value": "SENHA_DO_POSTGRES"
    },
    {
      "name": "DB_POSTGRESDB_PORT",
      "value": "5432"
    },
    {
      "name": "N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS",
      "value": "true"
    },
    {
      "name": "N8N_HOST",
      "value": "SEU_DOMINIO"
    },
    {
      "name": "N8N_BLOCK_ENV_ACCESS_IN_NODE",
      "value": "true"
    },
    {
      "name": "DB_POSTGRESDB_HOST",
      "value": "ENDPOINT_DO_RDS"
    },
    {
      "name": "DB_TYPE",
      "value": "postgresdb"
    },
    {
      "name": "EXECUTION_MODE",
      "value": "regular"
    },
    {
      "name": "DB_POSTGRESDB_USER",
      "value": "postgres"
    },
    {
      "name": "N8N_RUNNERS_ENABLED",
      "value": "true"
    },
    {
      "name": "GENERIC_TIMEZONE",
      "value": "America/Sao_Paulo"
    },
    {
      "name": "N8N_GIT_NODE_DISABLE_BARE_REPOS",
      "value": "true"
    }
  ],
  "portMappings": [
    {
      "containerPort": 5678,
      "hostPort": 0,
      "protocol": "tcp",
      "name": "porta-0-to-5678",
      "appProtocol": "http"
    }
  ],
  "mountPoints": [
    {
      "sourceVolume": "n8n_efs",
      "containerPath": "/home/node/.n8n",
      "readOnly": false
    }
  ],
  "logConfiguration": {
    "logDriver": "awslogs",
    "options": {
      "awslogs-group": "/ecs/task-def-n8n",
      "awslogs-create-group": "true",
      "awslogs-region": "us-east-2",
      "awslogs-stream-prefix": "ecs"
    }
  },
  "volumes": [
    {
      "name": "n8n_efs",
      "efsVolumeConfiguration": {
        "fileSystemId": "EFS_FILE_SYSTEM_ID"
      }
    }
  ]
}
```

## 🔍 **Detalhamento das Configurações**

### **Recursos Computacionais**
- **CPU**: 1024 unidades (1 vCPU)
- **Memória**: 819 MB reservados
- **Imagem**: `n8nio/n8n:1.114.3` (versão estável)

### **Variáveis de Ambiente Críticas**

#### **Conexão com Banco**
```json
"DB_TYPE": "postgresdb"
"DB_POSTGRESDB_HOST": "ENDPOINT_DO_RDS"
"DB_POSTGRESDB_DATABASE": "n8n"
"DB_POSTGRESDB_USER": "postgres"
"DB_POSTGRESDB_PASSWORD": "SENHA_DO_POSTGRES"
"DB_POSTGRESDB_PORT": "5432"
```

#### **Configurações de Segurança**
```json
"N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS": "true"
"N8N_BLOCK_ENV_ACCESS_IN_NODE": "true"
"N8N_GIT_NODE_DISABLE_BARE_REPOS": "true"
```

#### **Configurações de Rede**
```json
"WEBHOOK_URL": "https://SEU_DOMINIO/"
"N8N_HOST": "SEU_DOMINIO"
```

### **Port Mapping**
- **Container Port**: 5678 (porta padrão do N8N)
- **Host Port**: 0 (porta aleatória - permite múltiplas instâncias)
- **Protocolo**: TCP/HTTP

### **Persistência de Dados**
- **Volume**: EFS montado em `/home/node/.n8n`
- **Função**: Armazena workflows, credenciais e configurações
- **Benefício**: Dados persistem mesmo com restart do container

### **Logs Centralizados**
- **Driver**: AWS CloudWatch Logs
- **Log Group**: `/ecs/task-def-n8n`
- **Região**: us-east-2
- **Auto-criação**: Habilitada

## 🚀 **Vantagens desta Configuração**

### **Escalabilidade**
- Port mapping dinâmico permite múltiplas instâncias
- CPU/Memória otimizados para workload do N8N

### **Segurança**
- Variáveis de ambiente isoladas
- Permissões de arquivo controladas
- Acesso a repositórios Git restrito

### **Observabilidade**
- Logs centralizados no CloudWatch
- Métricas de container disponíveis
- Facilita troubleshooting

### **Persistência**
- Dados críticos no EFS
- Backup automático disponível
- Alta disponibilidade dos dados

## 📝 **Boas Práticas Implementadas**

1. **Versão específica da imagem** (não `latest`)
2. **Recursos limitados** (evita consumo excessivo)
3. **Logs estruturados** (facilita análise)
4. **Dados persistentes** (EFS para workflows)
5. **Configuração por variáveis** (flexibilidade)

---

**💡 Dica**: Esta configuração é otimizada para produção, balanceando performance, segurança e custos.
