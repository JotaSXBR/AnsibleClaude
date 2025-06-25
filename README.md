# Projeto Ansible para Servidor de Produção com Docker Swarm, Traefik e Portainer

Este repositório contém um playbook Ansible para configurar um servidor Ubuntu 24.04 do zero, transformando-o em um ambiente de produção robusto e seguro, pronto para hospedar aplicações em contêineres.

O playbook automatiza a instalação e configuração de Docker, Docker Swarm, Traefik como reverse proxy com SSL automático (Let's Encrypt), e Portainer para gerenciamento de contêineres.

## Funcionalidades

- **Segurança:** Configura UFW, Fail2Ban, atualizações automáticas de segurança e hardening do kernel.
- **Docker e Swarm:** Instala o Docker e inicializa um cluster Swarm, deixando o ambiente pronto para deploy de serviços.
- **Traefik:** Implanta o Traefik v3 como um serviço Swarm para atuar como reverse proxy e load balancer, com integração Let's Encrypt para certificados SSL automáticos.
- **Portainer:** Implanta o Portainer CE para facilitar o gerenciamento do ambiente Docker via interface web.
- **Manutenção:** Adiciona um script de limpeza e um cron job para manutenção periódica do sistema.

---

## ‼️ Intervenção Manual Obrigatória

Antes de executar o playbook, duas ações manuais são **essenciais**.

### Passo 1: Configurar o Inventário (`inventory.ini`)

Abra o arquivo `inventory.ini` e configure **todas** as variáveis a seguir de acordo com o seu ambiente:

- `ansible_host`: O endereço IP do seu servidor.
- `ansible_user`: O usuário com permissões `sudo` que o Ansible usará para se conectar (ex: `root` ou outro usuário).
- `deploy_user`: O nome do usuário de deploy que será criado (ex: `deploy`).
- `fail2ban_bantime`: Tempo de banimento do Fail2Ban (ex: `1h` para 1 hora).
- `fail2ban_maxretry`: Número de tentativas de login antes do banimento.
- `traefik_version`: A versão do Traefik a ser usada (ex: `v3.0`).
- `portainer_version`: A versão do Portainer CE a ser usada (ex: `latest`).
- `traefik_domain`: O domínio que você usará para o dashboard do Traefik (ex: `traefik.seusite.com`).
- `portainer_domain`: O domínio que você usará para a interface do Portainer (ex: `portainer.seusite.com`).
- `letsencrypt_email`: Seu endereço de e-mail, usado pela Let's Encrypt para notificações sobre seus certificados.

### Passo 2: Configurar o DNS

Após definir seus domínios no `inventory.ini`, você **deve** acessar o painel de controle do seu provedor de DNS e criar registros do tipo `A` apontando ambos os domínios (`traefik_domain` e `portainer_domain`) para o endereço IP do seu servidor.

O Traefik não conseguirá gerar os certificados SSL se o DNS não estiver propagado corretamente.

---

## Como Executar

1.  Certifique-se de que o Ansible está instalado na sua máquina local.
2.  Preencha as variáveis no arquivo `inventory.ini` conforme descrito acima.
3.  Execute o playbook a partir do seu terminal:

    ```bash
    ansible-playbook -i inventory.ini playbook.yml
    ```

---

## Pós-Instalação: Primeiros Passos

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

**Recomendação:** Faça backup deste arquivo regularmente. Se for migrar o servidor, este é o arquivo que você precisa levar junto para manter seus certificados.

---

## Verificação e Solução de Problemas

Para verificar se os serviços estão rodando corretamente, conecte-se ao seu servidor via SSH e use os seguintes comandos:

-   **Verificar todos os serviços do stack:**
    -   `docker stack ps traefik`
    -   `docker stack ps portainer`
-   **Ver logs do Traefik:**
    -   `docker service logs traefik_traefik`
-   **Ver logs do Portainer:**
    -   `docker service logs portainer_portainer` 