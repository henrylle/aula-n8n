# ☁️ Recursos AWS - Infraestrutura N8N

## 🎯 **Visão Geral**

Esta documentação detalha todos os recursos AWS utilizados na implantação do N8N, incluindo configurações, custos estimados e justificativas técnicas.

## 🏗️ **Arquitetura Completa**

```
Internet → Route 53 → CloudFront → ALB → EC2 (ECS) → EFS
                                            ↓
                                          RDS
```

## 📋 **Lista Completa de Recursos**

### **1. Elastic Container Service (ECS)**

#### **ECS Cluster**
- **Tipo**: EC2 (não Fargate)
- **Instâncias**: t3.micro
- **Auto Scaling**: Habilitado
- **Capacity Provider**: EC2

**Por que EC2 em vez de Fargate?**
- Menor custo para workloads contínuos
- Maior controle sobre instâncias
- Melhor para aplicações stateful como N8N

#### **ECS Service**
- **Desired Count**: 1
- **Deployment Type**: Rolling update
- **Health Check**: ALB Target Group
- **Network Mode**: Bridge
- **Auto Recovery**: Habilitado

#### **Task Definition**
- **CPU**: 1024 unidades (1 vCPU)
- **Memória**: 819 MB
- **Imagem**: n8nio/n8n:1.114.3
- **Volumes**: EFS montado

---

### **2. Application Load Balancer (ALB)**

#### **Configuração**
- **Tipo**: Application Load Balancer
- **Scheme**: Internet-facing
- **Subnets**: 2 AZs (us-east-2a, us-east-2b)
- **Security Group**: n8n-alb

#### **Target Group**
- **Tipo**: Instance
- **Protocolo**: HTTP
- **Porta**: Dinâmica (0-65535)
- **Health Check**: HTTP:/ (porta do container)

#### **Listeners**
- **HTTPS:443**: Certificado SSL/TLS
- **Redirect HTTP→HTTPS**: Automático

---

### **3. Elastic File System (EFS)**

#### **Configuração**
- **Performance Mode**: General Purpose
- **Throughput Mode**: Bursting
- **Criptografia**: Habilitada (KMS)
- **Backup**: Automático habilitado

#### **Mount Targets**
- **Subnets**: Todas as AZs do cluster
- **Security Group**: n8n-efs
- **Porta**: 2049 (NFS)

**Por que EFS?**
- Dados persistem entre restarts
- Compartilhado entre múltiplas instâncias
- Backup automático
- Escalabilidade automática

---

### **4. Relational Database Service (RDS)**

#### **Configuração**
- **Engine**: PostgreSQL 15
- **Classe**: db.t3.micro
- **Storage**: 20 GB GP2
- **Multi-AZ**: Não (para reduzir custos)

#### **Segurança**
- **Subnet Group**: Subnets privadas
- **Security Group**: n8n-db
- **Criptografia**: Habilitada
- **Backup**: 7 dias de retenção

**Por que PostgreSQL?**
- Recomendado pelo N8N
- Melhor performance que SQLite
- Suporte a JSON nativo
- ACID compliance

---

### **5. Elastic Compute Cloud (EC2)**

#### **Instâncias**
- **Tipo**: t3.micro
- **AMI**: Amazon Linux 2023 ECS-optimized
- **Storage**: 30 GB GP3
- **IAM Role**: ecsInstanceRole

#### **Auto Scaling Group**
- **Min**: 1 instância
- **Max**: 1 instância
- **Desired**: 1 instância
- **Health Check**: ECS + EC2
- **Função**: Auto recuperação em caso de falha

#### **Launch Template**
- **User Data**: ECS agent configuration
- **Security Group**: n8n-ec2
- **Key Pair**: Para acesso SSH (opcional)

---

### **6. Virtual Private Cloud (VPC)**

#### **Configuração**
- **CIDR**: 172.31.0.0/16 (VPC padrão)
- **Subnets Públicas**: 2 AZs
- **Subnets Privadas**: 2 AZs
- **Internet Gateway**: Habilitado

#### **Route Tables**
- **Pública**: 0.0.0.0/0 → Internet Gateway
- **Privada**: Tráfego local apenas

---

### **7. CloudWatch Logs**

#### **Log Groups**
- **Nome**: /ecs/task-def-n8n
- **Retenção**: 30 dias
- **Criptografia**: Habilitada

#### **Métricas**
- **ECS**: CPU, Memória, Network
- **ALB**: Request count, Latency, Errors
- **RDS**: Connections, CPU, Storage

---

### **8. Identity and Access Management (IAM)**

#### **Roles Necessárias**
- **ecsTaskExecutionRole**: Execução de tasks
- **ecsInstanceRole**: Instâncias EC2
- **ecsServiceRole**: Gerenciamento do service

#### **Políticas**
- **AmazonECSTaskExecutionRolePolicy**
- **AmazonEC2ContainerServiceforEC2Role**
- **CloudWatchLogsFullAccess**

---

## 🔧 **Configurações Específicas**

### **ECS Task Definition**
```json
{
  "networkMode": "bridge",
  "requiresCompatibilities": ["EC2"],
  "cpu": "1024",
  "memory": "819",
  "executionRoleArn": "arn:aws:iam::ACCOUNT:role/ecsTaskExecutionRole"
}
```

### **ALB Target Group**
```json
{
  "targetType": "instance",
  "protocol": "HTTP",
  "healthCheckPath": "/",
  "healthCheckIntervalSeconds": 30,
  "healthyThresholdCount": 2
}
```

### **EFS Mount Target**
```json
{
  "fileSystemId": "fs-xxxxxxxxx",
  "subnetId": "subnet-xxxxxxxxx",
  "securityGroups": ["sg-n8n-efs"]
}
```

## 🚀 **Benefícios da Arquitetura**

### **Alta Disponibilidade**
- Multi-AZ deployment
- Auto recuperação de instâncias
- Health checks automáticos
- Backup automático do RDS

### **Segurança**
- Subnets privadas para aplicação
- Security Groups restritivos
- Criptografia em repouso e trânsito
- IAM roles com menor privilégio

### **Persistência de Dados**
- EFS para dados do N8N
- RDS com backup automático
- Dados sobrevivem a falhas de instância
- Recuperação automática do serviço

### **Observabilidade**
- Logs centralizados
- Métricas detalhadas
- Alertas configuráveis
- Dashboards personalizados


---

**💡 Dica**: Esta arquitetura é otimizada para produção, balanceando custo, performance e segurança. Para ambientes de desenvolvimento, considere usar Fargate para simplificar o gerenciamento.
