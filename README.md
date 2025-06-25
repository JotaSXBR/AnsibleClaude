# Ansible Playbook: Configuração de Servidor Ubuntu 24.04 LTS para Produção

Este projeto contém um playbook Ansible projetado para automatizar a configuração completa e o endurecimento (*hardening*) de um servidor Ubuntu 24.04 LTS. O objetivo é transformar uma instalação limpa em um ambiente de produção seguro e robusto, pronto para hospedar aplicações em contêineres com Docker Swarm, Traefik e Portainer.

---

## 📋 Funcionalidades Detalhadas

O playbook executa um conjunto abrangente de tarefas, divididas nas seguintes categorias:

### 🛡️ Segurança (Hardening)
- **Atualizações do Sistema**: Garante que todos os pacotes estejam na última versão e configura o `unattended-upgrades` para instalar atualizações de segurança automaticamente.
- **Firewall (UFW)**: Configura o `UFW` (Uncomplicated Firewall) para negar todo o tráfego de entrada por padrão, permitindo apenas portas essenciais (SSH, HTTP, HTTPS) e as portas de comunicação interna do Docker Swarm.
- **Fail2Ban**: Instala e configura o `Fail2Ban` para proteger o serviço SSH contra ataques de força bruta, banindo IPs que apresentem comportamento suspeito.
- **Segurança do Kernel**: Aplica um conjunto de configurações de segurança ao kernel via `sysctl` para mitigar vários tipos de ataques de rede e de sistema.
- **Limites do Sistema**: Aumenta os limites de arquivos abertos (`nofile`) and processos (`nproc`) para otimizar o desempenho de aplicações de alta carga.
- **Configuração SSH Segura**:
  - Desabilita o login do usuário `root`.
  - Limita o número máximo de tentativas de autenticação.
  - Adiciona um banner de aviso legal na tela de login.
- **Ferramentas de Segurança**: Instala um conjunto de ferramentas renomadas para verificação e monitoramento de segurança (`rkhunter`, `chkrootkit`, `lynis`, `auditd`, `clamav`).
- **Auditoria de Sistema**: Configura o `auditd` para monitorar o acesso e modificações em arquivos críticos do sistema, como `/etc/passwd` e `/etc/ssh/sshd_config`.

### 🐳 Docker e Orquestração
- **Docker**: Instala a versão mais recente do Docker Engine a partir do repositório oficial do Docker para garantir compatibilidade e atualizações.
- **Docker Compose**: Instala o plugin `docker-compose` para facilitar a definição de aplicações multi-contêiner.
- **Docker Swarm**: Inicializa um cluster Docker Swarm no servidor (modo de nó único), preparando o ambiente para deploy de serviços distribuídos e resilientes.
- **Traefik (Reverse Proxy)**: Implanta o Traefik v3 como um serviço Swarm. Ele atuará como reverse proxy e load balancer, inspecionando o Docker Swarm para descobrir e expor serviços automaticamente.
- **Let's Encrypt (SSL Automático)**: Configura o Traefik com integração nativa ao Let's Encrypt para gerar e renovar certificados SSL/TLS automaticamente para todos os seus serviços.
- **Portainer (Gerenciamento)**: Implanta o Portainer CE como um serviço Swarm, fornecendo uma interface web poderosa para visualizar, gerenciar e solucionar problemas no ambiente Docker.

### ⚙️ Manutenção e Usuários
- **Usuário de Deploy Dedicado**: Cria um usuário (`deploy` por padrão) sem privilégios de root diretos, mas com acesso `sudo`, para executar tarefas administrativas de forma segura. Este usuário também é adicionado ao grupo `docker`.
- **Manutenção Automatizada**: Configura `logrotate` para os logs do Docker e adiciona um script de manutenção (limpeza de imagens, contêineres e volumes não utilizados) que roda semanalmente via `cron`.

---

## 🚀 Pré-requisitos

### Máquina de Controle (Onde você executa o Ansible)
- **Ansible**: `sudo apt update && sudo apt install ansible` ou `pip install ansible`.
- **Git**: Para clonar o repositório.

### Servidor Remoto (O alvo da configuração)
- Um servidor com **Ubuntu 24.04 LTS** recém-instalado.
- Acesso SSH ao servidor com um usuário que tenha privilégios `sudo` (pode ser o `root` para a configuração inicial).

---

## ‼️ Intervenção Manual Obrigatória

Antes de executar o playbook, duas ações manuais são **essenciais**.

### Passo 1: Configurar o Inventário (`inventory.ini`)

1.  **Crie seu arquivo de inventário:** Renomeie o arquivo `inventory_default.ini` para `inventory.ini`. Este arquivo (`inventory.ini`) já está no `.gitignore` para proteger suas credenciais.
2.  **Preencha as variáveis:** Abra o `inventory.ini` e configure **todas** as variáveis de acordo com o seu ambiente. As variáveis são comentadas no arquivo padrão para explicar o que cada uma faz. As mais importantes são:
    - `ansible_host`: O endereço IP do seu servidor.
    - `ansible_user`: O usuário com permissões `sudo`.
    - `traefik_domain` e `portainer_domain`: Seus domínios.
    - `letsencrypt_email`: Seu e-mail para notificações da Let's Encrypt.

### Passo 2: Configurar o DNS

Após definir seus domínios no `inventory.ini`, você **deve** acessar o painel de controle do seu provedor de DNS e criar registros do tipo `A` apontando ambos os domínios (`traefik_domain` e `portainer_domain`) para o endereço IP do seu servidor.

O Traefik não conseguirá gerar os certificados SSL se o DNS não estiver propagado corretamente.

---

## 💡 Como Executar

1.  Clone este repositório para sua máquina local.
2.  Siga os passos de intervenção manual descritos acima.
3.  Execute o playbook a partir do seu terminal:
    ```bash
    ansible-playbook -i inventory.ini playbook.yml
    ```
- **Execução em modo de verificação (Dry Run)**, para ver o que seria feito sem aplicar as mudanças:
  ```bash
  ansible-playbook -i inventory.ini playbook.yml --check
  ```
- **Execução em modo verboso**, para mais detalhes sobre cada passo:
  ```bash
  ansible-playbook -i inventory.ini playbook.yml -v
  ```

---

## ✅ Pós-Instalação e Verificação

### 1. Configuração Inicial do Portainer
O playbook implanta o Portainer, mas a criação do usuário administrador é um passo manual:

1.  Acesse a URL do Portainer que você configurou (ex: `https://portainer.seusite.com`).
2.  **Tela 1 - Criar Usuário:** Crie sua conta de administrador.
3.  **Tela 2 - Assistente de Ambiente:** Após o login, você verá o "Environment Wizard". Ele deve detectar automaticamente seu ambiente Docker Swarm. Apenas clique no botão **"Get Started"** para finalizar a configuração.

> **Nota:** Se por acaso você se deparar com uma tela de "timed out for security purposes", significa que a janela de 5 minutos para a configuração inicial expirou. Para resolver, execute o comando `docker service update --force portainer_portainer` no seu servidor e tente acessar a URL novamente.

### 2. Gerenciamento do `acme.json` (Certificados Let's Encrypt)

O Traefik armazena seus certificados SSL em um arquivo no servidor.

-   **Localização:** `/opt/traefik/acme/acme.json`

Este arquivo é **CRÍTICO**. Se você o perder, perderá seus certificados SSL e poderá ser bloqueado temporariamente pela Let's Encrypt por excesso de solicitações.

**Recomendação:** Faça backup deste arquivo regularmente. Se for migrar o servidor ou reinstalar o Traefik, este é o arquivo que você precisa levar junto para manter seus certificados.

### 3. Comandos de Verificação no Servidor

Conecte-se ao seu servidor via SSH (com o `deploy_user`) e use os seguintes comandos para verificar o estado do sistema:

-   **Versão do Docker**: `docker --version`
-   **Status do Docker Swarm**: `docker node ls`
-   **Stacks em execução**: `docker stack ls`
-   **Serviços detalhados**: `docker stack ps traefik` e `docker stack ps portainer`
-   **Status do Firewall**: `sudo ufw status`
-   **Status do Fail2Ban**: `sudo fail2ban-client status sshd`
-   **Logs do Traefik**: `docker service logs traefik_traefik`
-   **Logs do Portainer**: `docker service logs portainer_portainer`

---

## ⚠️ Nota de Segurança

Este playbook aplica uma série de configurações de segurança robustas. No entanto, segurança é um processo contínuo. **É altamente recomendável que você revise todas as configurações** e as ajuste de acordo com as necessidades específicas do seu ambiente. Considere desabilitar a autenticação por senha no SSH assim que tiver configurado e testado o acesso por chave para o `deploy_user`. 