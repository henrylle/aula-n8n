# 🔒 Security Groups - Arquitetura de Segurança N8N

## 🎯 **Visão Geral**

Os Security Groups funcionam como firewalls virtuais, controlando o tráfego de rede entre os recursos AWS. Nossa arquitetura implementa o princípio do menor privilégio.

## 📊 **Arquitetura de Rede**

```
Internet → ALB → EC2 (ECS) → EFS
                    ↓
                   RDS
```

## 🛡️ **Mapeamento dos Security Groups**

### **1. Application Load Balancer (ALB)**

#### **Security Group: `n8n-alb`**
- **Função**: Recebe tráfego HTTPS da internet
- **Localização**: Subnets públicas

| Protocolo | Porta | Origem | Descrição |
|-----------|-------|--------|-----------|
| TCP | 443 | 0.0.0.0/0 | HTTPS liberado geral |

**Por que esta configuração?**
- Porta 443 é padrão para HTTPS
- `0.0.0.0/0` permite acesso global (necessário para aplicação web)
- Certificado SSL/TLS termina no ALB

---

### **2. Instâncias EC2 (ECS Cluster)**

#### **Security Group: `n8n-ec2`**
- **Função**: Executa containers N8N
- **Localização**: Subnets privadas

| Protocolo | Porta | Origem | Descrição |
|-----------|-------|--------|-----------|
| TCP | 0-65535 | sg-n8n-alb | Acesso vindo do ALB |

**Por que esta configuração?**
- Range amplo de portas devido ao port mapping dinâmico
- Origem restrita apenas ao ALB (segurança)
- Permite que ALB encontre a porta aleatória do container

---

### **3. Elastic File System (EFS)**

#### **Security Group: `n8n-efs`**
- **Função**: Armazenamento persistente de dados
- **Protocolo**: NFS (Network File System)

| Protocolo | Porta | Origem | Descrição |
|-----------|-------|--------|-----------|
| TCP | 2049 | sg-n8n-ec2 | NFS - Acesso vindo do EC2 |

**Por que esta configuração?**
- Porta 2049 é padrão do protocolo NFS
- Acesso restrito apenas às instâncias EC2
- Garante que apenas containers autorizados acessem dados

---

### **4. Relational Database Service (RDS)**

#### **Security Group: `n8n-db`**
- **Função**: Banco PostgreSQL para N8N
- **Localização**: Subnets privadas

| Protocolo | Porta | Origem | Descrição |
|-----------|-------|--------|-----------|
| TCP | 5432 | sg-n8n-ec2 | PostgreSQL - Acesso vindo da EC2 |

**Por que esta configuração?**
- Porta 5432 é padrão do PostgreSQL
- Acesso restrito apenas às instâncias EC2
- Banco isolado da internet (máxima segurança)

## 🔄 **Fluxo de Comunicação Detalhado**

### **1. Usuário → ALB**
```
Internet (HTTPS:443) → sg-n8n-alb
```
- Usuário acessa aplicação via HTTPS
- Certificado SSL validado no ALB
- Tráfego criptografado até o ALB

### **2. ALB → EC2**
```
sg-n8n-alb → sg-n8n-ec2 [TCP:porta-aleatória]
```
- ALB distribui carga entre instâncias
- Target Group identifica instâncias saudáveis
- Porta dinâmica descoberta automaticamente

### **3. EC2 → EFS**
```
sg-n8n-ec2 → sg-n8n-efs [TCP:2049]
```
- Container monta volume EFS
- Dados persistem entre restarts
- Backup automático disponível

### **4. EC2 → RDS**
```
sg-n8n-ec2 → sg-n8n-db [TCP:5432]
```
- Aplicação conecta ao PostgreSQL
- Credenciais via variáveis de ambiente
- Conexões pooling para performance

## 🛡️ **Princípios de Segurança Implementados**

### **1. Menor Privilégio**
- Cada SG permite apenas tráfego necessário
- Portas específicas para cada serviço
- Origens restritas por Security Group ID

### **2. Defesa em Profundidade**
- Múltiplas camadas de segurança
- ALB → EC2 → Banco isolados
- Subnets públicas/privadas separadas

### **3. Referências por Security Group**
- Uso de SG IDs em vez de CIDRs
- Facilita manutenção e auditoria
- Reduz erros de configuração

### **4. Isolamento de Recursos**
- Cada tier tem seu próprio SG
- Comunicação controlada entre camadas
- Facilita troubleshooting

## 📋 **Resumo dos Security Groups**

| Nome | Função | Porta Principal | Acesso |
|------|--------|----------------|---------|
| n8n-alb | Load Balancer | 443 | Internet |
| n8n-ec2 | Aplicação | Dinâmica | ALB apenas |
| n8n-efs | Storage | 2049 | EC2 apenas |
| n8n-db | Database | 5432 | EC2 apenas |

## 🔍 **Troubleshooting Comum**

### **Problema: Aplicação não carrega**
1. Verificar se ALB está healthy
2. Confirmar Target Group com instâncias
3. Validar Security Group do ALB (porta 443)

### **Problema: Dados não persistem**
1. Verificar mount do EFS no container
2. Confirmar Security Group EFS (porta 2049)
3. Validar permissões do filesystem

### **Problema: Erro de conexão com banco**
1. Verificar endpoint RDS nas variáveis
2. Confirmar Security Group RDS (porta 5432)
3. Validar credenciais de acesso

## 🚀 **Benefícios desta Arquitetura**

### **Segurança**
- Tráfego controlado entre todas as camadas
- Banco isolado da internet
- Certificados SSL/TLS gerenciados

### **Escalabilidade**
- ALB distribui carga automaticamente
- Port mapping dinâmico permite múltiplas instâncias
- EFS compartilhado entre containers

### **Manutenibilidade**
- Security Groups bem documentados
- Referências claras entre recursos
- Facilita auditoria e compliance

---

**💡 Dica**: Esta configuração segue as melhores práticas da AWS para aplicações web em produção.
