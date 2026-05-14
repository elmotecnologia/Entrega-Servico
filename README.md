# 🚀 Proxmox Automation Suite

Conjunto de scripts para automação de provisionamento de redes (VNet) e serviços (DBaaS e KaaS) em clusters Proxmox VE.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Bash](https://img.shields.io/badge/shell-bash-green.svg)](https://www.gnu.org/software/bash/)
[![Proxmox](https://img.shields.io/badge/Proxmox-VE-orange.svg)](https://www.proxmox.com/)

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Pré-requisitos](#pré-requisitos)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Fluxo de Trabalho](#fluxo-de-trabalho)
- [Scripts](#scripts)
  - [1. criar_vnet.sh - Criação de Redes](#1-criar_vnetsh---criação-de-redes)
  - [2. criar_usuario.sh - Provisionamento de Clientes](#2-criar_usuariosh---provisionamento-de-clientes)
  - [3. clonar_para_vnet.sh - Deployment de Serviços](#3-clonar_para_vnetsh---deployment-de-serviços)
- [Exemplos de Uso](#exemplos-de-uso)
- [Logs e Estado](#logs-e-estado)
- [Segurança](#segurança)
- [Solução de Problemas](#solução-de-problemas)
- [Roadmap](#roadmap)

## 🎯 Visão Geral

Esta suíte de automação foi desenvolvida para ambientes de provedores de serviços que utilizam Proxmox VE como plataforma de virtualização. Ela permite:

- ✅ Criação automatizada de redes isoladas (VLANs) via SDN do Proxmox
- ✅ Provisionamento de clientes com usuários, pools e permissões granulares
- ✅ Deployment de serviços como Banco de Dados (DBaaS) e Kubernetes (KaaS)
- ✅ Gerenciamento completo de ciclo de vida das VMs
- ✅ Isolamento multi-tenant com VNets dedicadas

## 📦 Pré-requisitos

### Infraestrutura

- Proxmox VE 7.x ou superior
- Template VMs configurados:
  - ID `110` - MariaDB Template
  - ID `111` - PostgreSQL Template
  - ID `120` - Kubernetes Control Plane Template
  - ID `121` - Kubernetes Worker Template
  - ID `122` - HAProxy Template
- SDN configurado no cluster Proxmox

### Machine de Deployment (Onde os scripts serão executados)


# Instalar dependências
sudo apt-get update
sudo apt-get install -y sshpass openssl

# Configurar diretório do projeto
mkdir -p /home/elmotecnologia/projetos/deploy-automacao
cd /home/elmotecnologia/projetos/deploy-automacao


### Permissões no Proxmox

- Usuário `root` ou com privilégios de administrador
- Acesso SSH habilitado
- Templates configurados e funcionando

## 📁 Estrutura do Projeto


deploy-automacao/
│
├── criar_vnet.sh              # Criação de VNets e zonas SDN
├── criar_usuario.sh           # Provisionamento de clientes
├── clonar_para_vnet.sh        # Deployment de serviços
│
├── .state/                    # Estado global do sistema
│   └── vnet_info.json         # Última VNet criada
│
├── logs/                      # Diretório de logs
│   └── clientes/              # Logs específicos por cliente
│       └── {CLIENTE}/         
│           ├── provisionamento_*.log
│           ├── vm_*.conf
│           ├── haproxy.cfg
│           └── state.conf
│
└── vnet_*.conf                # Configurações exportadas


## 🔄 Fluxo de Trabalho

mermaid
graph LR
    A[Criar VNet] --> B[Criar Usuário/Pool]
    B --> C[Deploy Serviço]
    C --> D{Qual Serviço?}
    D -->|DBaaS| E[MariaDB/PostgreSQL]
    D -->|KaaS| F[Cluster Kubernetes]
    F --> G[HAProxy + Control Planes + Workers]


**Sequência recomendada:**
1. Execute `criar_vnet.sh` para criar a rede isolada
2. Execute `criar_usuario.sh` para provisionar o cliente
3. Execute `clonar_para_vnet.sh` para deploy do serviço

## 📜 Scripts

### 1. criar_vnet.sh - Criação de Redes

Cria infraestrutura de rede isolada usando SDN do Proxmox com suporte a VLANs.

**Funcionalidades:**
- ✅ Criação de zonas SDN (VLAN)
- ✅ Criação de VNets com VLAN TAG específica
- ✅ Configuração de sub-redes e gateways
- ✅ Ativação opcional de SNAT para acesso à internet
- ✅ Isolamento de portas (port security)
- ✅ Salva configuração em arquivo para auditoria

**Parâmetros Interativos:**

| Parâmetro | Descrição | Exemplo |
|-----------|-----------|---------|
| IP Proxmox | Endereço do servidor | `192.168.2.200` |
| Usuário SSH | Usuário de acesso | `root` |
| Nome da Zona | Identificador da zona | `DBaaS` |
| Bridge | Bridge física | `vmbr1` |
| Nome da VNet | Identificador da rede | `cliente1` |
| VLAN TAG | ID da VLAN (1-4094) | `101` |
| Sub-rede | Rede no formato CIDR | `10.0.101.0/24` |
| Gateway | Gateway da rede | `10.0.101.1` |

**Exemplo de saída:**

✅ VNET CRIADA COM SUCESSO!
- Zona: DBaaS
- VNet: cliente1 (VLAN: 101)
- Sub-rede: 10.0.101.0/24 (Gateway: 10.0.101.1)
- SNAT: Ativado
- Isolamento de portas: Ativado


### 2. criar_usuario.sh - Provisionamento de Clientes

Cria usuário, pool e configura permissões para um novo cliente.

**Funcionalidades:**
- ✅ Criação de usuário no Proxmox com senha aleatória
- ✅ Criação de pool exclusivo para o cliente
- ✅ Configuração de ACLs granulares:
  - `PVEVMAdmin` no pool (gestão de VMs)
  - `PVEDatastoreUser` no storage (acesso a discos)
  - `PVESDNUser` nas redes (acesso SDN)
- ✅ Geração de log de provisionamento
- ✅ Saída formatada para integração com outros scripts

**Uso:**

# Interativo
./criar_usuario.sh

# Com parâmetro
./criar_usuario.sh "empresa_x"


**Exemplo de saída:**

--------------------------------------------------------
PROVISIONAMENTO CONCLUÍDO!
USUÁRIO: empresa_x@pve
SENHA:   aB3dE5fG7hI9jK1
POOL:    Pool_empresa_x
--------------------------------------------------------


### 3. clonar_para_vnet.sh - Deployment de Serviços

Script principal para deployment de serviços nas VNets criadas.

#### 🗄️ DBaaS (Database as a Service)

Cria VMs de banco de dados a partir de templates otimizados.

**Características:**
- Templates: MariaDB (ID 110) e PostgreSQL (ID 111)
- Faixa de IDs: 100-1999
- Clone FULL (não linked-clone)
- Configurações customizáveis:
  - CPUs (padrão: 2)
  - Memória RAM (padrão: 4096 MB)
  - Disco (redimensionável)
- Conectividade automática à VNet

#### ☸️ KaaS (Kubernetes as a Service)

Cria clusters Kubernetes completos.

**Arquitetura:**

┌─────────────────────────────────────┐
│         CLIENT VNet (VLAN)          │
│                                      │
│  ┌─────────┐  ┌──────────────────┐ │
│  │HAProxy  │  │  Control Planes  │ │
│  │(ID:122) │◄─┤  - cp-1 (ID:120) │ │
│  │Load     │  │  - cp-2 (ID:120) │ │
│  │Balancer │  │  - cp-N (ID:120) │ │
│  └─────────┘  └──────────────────┘ │
│       │               │             │
│       ▼               ▼             │
│  ┌──────────────────────────────┐  │
│  │         Workers              │  │
│  │  - worker-1 (ID:121)         │  │
│  │  - worker-2 (ID:121)         │  │
│  │  - worker-N (ID:121)         │  │
│  └──────────────────────────────┘  │
└─────────────────────────────────────┘


**Características:**
- Template HAProxy: ID 122 (Load Balancer)
- Template Control Plane: ID 120 (K8s master)
- Template Worker: ID 121 (K8s node)
- Faixa exclusiva de IDs: 2000-2999
- Verificação automática de disponibilidade
- Geração de configuração HAProxy
- Suporte a multi-control plane (alta disponibilidade)

**Configurações customizáveis:**
- **Control Planes:**
  - CPUs (padrão: 2)
  - Memória (padrão: 4096 MB)
  - Quantidade (padrão: 1)
- **Workers:**
  - CPUs (padrão: 4)
  - Memória (padrão: 8192 MB)
  - Quantidade (padrão: 2)

## 💡 Exemplos de Uso

### Exemplo 1: Cliente de Banco de Dados


# 1. Criar rede isolada para o cliente
./criar_vnet.sh
# IP: 192.168.2.200
# VNet: cliente_financeiro
# VLAN: 200
# Sub-rede: 10.0.200.0/24

# 2. Criar usuário e pool
./criar_usuario.sh financeiro

# 3. Deploy do PostgreSQL
./clonar_para_vnet.sh
# Opção: 1 (DBaaS)
# Banco: 2 (PostgreSQL)
# CPUs: 4
# Memória: 8192
# Disco: 100G


### Exemplo 2: Cluster Kubernetes para Produção


# 1. Criar rede isolada
./criar_vnet.sh
# VNet: cluster_prod
# VLAN: 300
# Sub-rede: 10.0.300.0/24

# 2. Criar cliente
./criar_usuario.sh producao

# 3. Deploy cluster K8s
./clonar_para_vnet.sh
# Opção: 2 (KaaS)
# Control Planes: 3
# Workers: 5
# CPUs CP: 4
# Memória CP: 8192
# CPUs Worker: 8
# Memória Worker: 16384


## 📊 Logs e Estado

### Estrutura de Logs


logs/clientes/{CLIENTE}/
├── provisionamento_20240101_120000.log  # Log do provisionamento
├── vm_100_20240101_120000.conf          # Configuração da VM
├── haproxy.cfg                           # Config do HAProxy (KaaS)
└── state.conf                            # Estado atual do cliente


### Arquivos de Estado Global


.state/
└── vnet_info.json    # Última VNet criada (para integração)


### Integração com Orquestradores

Os scripts geram saída formatada para captura por outros sistemas:


# Exemplo de captura de saída
./criar_usuario.sh cliente_x | grep "###STATE_OUTPUT###" -A 3


## 🔒 Segurança

### Práticas Implementadas

- ✅ **Isolamento multi-tenant**: Cada cliente possui VNet dedicada com VLAN
- ✅ **Port Isolation**: Prevenção de comunicação entre VMs do mesmo cliente
- ✅ **Senhas aleatórias**: Geradas com `openssl rand -base64`
- ✅ **Permissões granulares**: ACLs específicas por recurso
- ✅ **Logs auditáveis**: Todas as operações são registradas

### Recomendações


# Restringir permissões dos scripts
chmod 700 criar_*.sh clonar_*.sh

# Configurar chave SSH (recomendado sobre senha)
ssh-copy-id root@192.168.2.200

# Armazenar credenciais em cofre (ex: HashiCorp Vault)
# Não versionar arquivos .conf com senhas


## ⚠️ Solução de Problemas

### Problema: Conexão SSH falha


# Verificar conectividade
ping 192.168.2.200

# Testar SSH manualmente
ssh root@192.168.2.200

# Verificar se sshpass está instalado
which sshpass


### Problema: Template não encontrado


# Listar templates disponíveis
ssh root@192.168.2.200 "qm list | grep template"

# Verificar IDs esperados
# 110 - MariaDB
# 111 - PostgreSQL  
# 120 - K8s Control Plane
# 121 - K8s Worker
# 122 - HAProxy


### Problema: Faixa de IDs esgotada (KaaS)


# Verificar VMs existentes na faixa 2000-2999
ssh root@192.168.2.200 "qm list" | grep -E "^2[0-9]{3}"

# Limpar VMs não utilizadas
ssh root@192.168.2.200 "qm destroy <VMID> --purge"


### Problema: VNet não aparece


# Verificar configuração SDN
ssh root@192.168.2.200 "pvesh get /cluster/sdn/status"

# Reaplicar configuração
ssh root@192.168.2.200 "pvesh set /cluster/sdn"


## 🗺️ Roadmap

### Versão 2.0 (Planejado)

- [ ] Suporte a backup automatizado
- [ ] Monitoramento integrado (Prometheus + Grafana)
- [ ] API REST para integração
- [ ] Interface web (React)
- [ ] Suporte a múltiplos storages (Ceph, NFS)
- [ ] Auto-scaling para KaaS
- [ ] Templates de aplicações (WordPress, GitLab, etc.)
- [ ] Disaster Recovery automatizado

## 📄 Licença

MIT License - veja o arquivo [LICENSE](LICENSE) para detalhes.

## 👥 Contribuição

Contribuições são bem-vindas! Por favor:

1. Fork o projeto
2. Crie sua branch (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

## 📞 Suporte

- 📧 Email: emerson@elmotecnologia.com.br
- 📚 Documentação adicional: [Wiki do Projeto](https://emersondominguescmara.substack.com/)
- 🐛 Reportar bugs: [Issues](https://emersondominguescmara.substack.com/)

---

**Desenvolvido com ❤️ para automação de infraestrutura Proxmox**
