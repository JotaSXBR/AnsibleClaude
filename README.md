# Ansible Playbook: Configuração de Servidor Ubuntu 24.04 LTS

Este projeto contém um playbook Ansible projetado para automatizar a configuração e o endurecimento de um servidor Ubuntu 24.04 LTS. O playbook instala e configura um ambiente seguro e pronto para produção com Docker, Docker Swarm, Traefik e Portainer.

## 📋 Funcionalidades

O playbook realiza as seguintes tarefas:

### 🛡️ Segurança (Hardening)
- **Atualizações do Sistema**: Atualiza todos os pacotes e configura atualizações automáticas de segurança.
- **Firewall (UFW)**: Configura o `UFW` para negar todo o tráfego de entrada por padrão, permitindo apenas portas essenciais (SSH, HTTP, HTTPS) e as necessárias para o Docker Swarm.
- **Fail2Ban**: Instala e configura o `Fail2Ban` para proteger o serviço SSH contra ataques de força bruta.
- **Segurança do Kernel**: Aplica configurações de segurança ao kernel via `sysctl` para mitigar vários tipos de ataques.
- **Limites do Sistema**: Aumenta os limites de `nofile` e `nproc` para melhor desempenho de aplicações de alta carga.
- **Configuração SSH Segura**:
  - Desabilita o login do usuário `root`.
  - Limita o número de tentativas de autenticação.
  - Cria um usuário de `deploy` dedicado com privilégios `sudo`.
  - Adiciona um banner de aviso no login.
- **Ferramentas de Segurança**: Instala um conjunto de ferramentas para verificação e monitoramento de segurança (`rkhunter`, `chkrootkit`, `lynis`, `auditd`, `clamav`).
- **Auditoria**: Configura o `auditd` para monitorar o acesso a arquivos críticos do sistema.

### 🐳 Docker e Orquestração
- **Docker**: Instala a versão mais recente do Docker a partir do repositório oficial.
- **Docker Compose**: Instala o Docker Compose.
- **Docker Swarm**: Inicializa um cluster Docker Swarm no servidor (modo de nó único).
- **Traefik**: Implanta o Traefik como um serviço Swarm para atuar como reverse proxy e load balancer, com integração Let's Encrypt para certificados SSL automáticos.
- **Portainer**: Implanta o Portainer CE como um serviço Swarm para facilitar a gestão do ambiente Docker.

### ⚙️ Manutenção e Usuários
- **Usuário de Deploy**: Cria um usuário dedicado (`deploy` por padrão) para gerenciar o servidor, adicionando-o aos grupos `sudo` e `docker`.
- **Manutenção Automatizada**: Configura scripts de manutenção (`logrotate`, limpeza do sistema) que rodam periodicamente via `cron`.

## 🚀 Pré-requisitos

### Máquina de Controle (Onde você executa o Ansible)
- **Ansible**: Certifique-se de ter o Ansible instalado.
  ```bash
  sudo apt update
  sudo apt install ansible
  ```
  ou
  ```bash
  pip install ansible
  ```
- **Git**: Para clonar o repositório.

### Servidor Remoto (O alvo da configuração)
- Um servidor com **Ubuntu 24.04 LTS** recém-instalado.
- Acesso SSH ao servidor com um usuário que tenha privilégios `sudo` (pode ser o `root` para a configuração inicial).

## 💡 Como Usar

### 1. Clone o Repositório
Clone este projeto para a sua máquina local:
```bash
git clone <URL_DO_REPOSITORIO>
cd <NOME_DO_REPOSITORIO>
```

### 2. Configure o Inventário
Edite o arquivo `inventory.ini` para corresponder ao seu ambiente.

**Exemplo de configuração com senha:**
```ini
[servers]
meu_servidor ansible_host=SEU_IP_OU_DOMINIO ansible_user=seu_usuario ansible_ssh_pass=sua_senha
```

**Exemplo com chave SSH (recomendado):**
```ini
[servers]
meu_servidor ansible_host=SEU_IP_OU_DOMINIO ansible_user=seu_usuario ansible_ssh_private_key_file=~/.ssh/sua_chave_privada
```

Substitua `SEU_IP_OU_DOMINIO`, `seu_usuario`, `sua_senha` e o caminho para a sua chave SSH.

### 3. Personalize as Variáveis
No final do arquivo `inventory.ini`, na seção `[servers:vars]`, você pode personalizar as variáveis do projeto:
```ini
[servers:vars]
# Usuário que será criado para deploy
deploy_user=deploy

# Domínios para os serviços (configure seu DNS adequadamente)
traefik_domain=traefik.seu-domino.com
portainer_domain=portainer.seu-domino.com

# Email para os certificados Let's Encrypt
letsencrypt_email=seu-email@seu-domino.com
```

### 4. Execute o Playbook
Execute o seguinte comando no seu terminal:
```bash
ansible-playbook -i inventory.ini playbook.yml
```
O Ansible se conectará ao seu servidor e executará todas as tarefas definidas.

**Comandos úteis:**
- **Modo "Dry Run"** (verifica o que seria feito sem executar as mudanças):
  ```bash
  ansible-playbook -i inventory.ini playbook.yml --check
  ```
- **Modo Verboso** (mostra mais detalhes da execução):
  ```bash
  ansible-playbook -i inventory.ini playbook.yml -v
  ```

## ✅ Verificação Pós-instalação

Após a conclusão do playbook, você pode se conectar ao servidor (usando o `deploy_user` criado) e verificar se tudo está funcionando:

- **Versão do Docker**:
  ```bash
  docker --version
  ```
- **Status do Docker Swarm**:
  ```bash
  docker node ls
  ```
- **Serviços em execução (Stacks)**:
  ```bash
  docker stack ls
  ```
- **Status do Firewall**:
  ```bash
  sudo ufw status
  ```
- **Status do Fail2Ban**:
  ```bash
  sudo fail2ban-client status sshd
  ```

## 🌐 Acesso aos Serviços

Para acessar as interfaces web do Traefik e do Portainer, você precisa apontar os domínios configurados (`traefik_domain` e `portainer_domain`) para o IP do seu servidor no seu provedor de DNS.

- **Dashboard do Traefik**: `https://traefik.seu-domino.com`
- **Dashboard do Portainer**: `https://portainer.seu-domino.com`

Na primeira vez que acessar o Portainer, você precisará criar um usuário administrador.

## ⚠️ Nota de Segurança

Este playbook aplica uma série de configurações de segurança robustas. No entanto, segurança é um processo contínuo. **É altamente recomendável que você revise todas as configurações** e as ajuste de acordo com as necessidades específicas do seu ambiente. Desabilite a autenticação por senha no SSH assim que tiver configurado o acesso por chave para o `deploy_user`.

## 📄 Licença

Este projeto é de código aberto. Sinta-se à vontade para usá-lo e modificá-lo. 