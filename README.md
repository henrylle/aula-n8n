# 🚀 Aula N8N - Implantação na AWS

## 📋 **Conteúdo da Aula**

Esta pasta contém a documentação completa da implantação do N8N na AWS usando ECS, demonstrando uma arquitetura robusta e escalável.

## 📁 **Arquivos Disponíveis**

### 1. **[Task Definition](./01-task-definition.md)**
- Configuração completa do container N8N
- Variáveis de ambiente necessárias
- Configuração de recursos (CPU/Memória)
- Mount points e volumes EFS
- Configuração de logs CloudWatch

### 2. **[Security Groups](./02-security-groups.md)**
- Mapeamento completo dos Security Groups
- Fluxo de comunicação entre recursos
- Regras de entrada (inbound) detalhadas
- Análise de segurança da arquitetura

### 3. **[Recursos AWS](./03-recursos-aws.md)**
- Lista completa dos recursos utilizados
- Configurações específicas de cada serviço
- Custos estimados da infraestrutura
- Boas práticas implementadas

## 🏗️ **Arquitetura Implementada**

```
Internet → CloudFront → ALB → EC2 (ECS) → EFS
                                  ↓
                                 RDS
```

---

**Formação AWS 5.0** | **Henrylle Maia** | **Novembro 2025**
