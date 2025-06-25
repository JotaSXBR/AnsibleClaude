# 🤖 Automatize seu Servidor Ubuntu 24.04 LTS em 15 minutos

> Uma receita Ansible que atualiza, protege e prepara seu servidor para hospedar aplicações em contêineres – sem dor de cabeça.

---

## 1. O que é isto?
Este projeto é um **playbook Ansible** (um roteiro em YAML) que executa automaticamente:

1. Atualização e endurecimento de segurança (firewall, Fail2Ban, ajustes de kernel…).
2. Instalação do Docker + Docker Swarm.
3. Implantação do **Traefik** (proxy reverso com HTTPS automático via Let's Encrypt).
4. Implantação do **Portainer** (painel web para gerenciar o Docker).

Quando terminar, você terá um servidor limpo, seguro e pronto para hospedar stacks Docker com **TLS válido**.

---

## 2. Pré-requisitos (resumido)

| Você precisa de | Detalhes |
|-----------------|----------|
| 🖥️ **Máquina de controle** | Qualquer PC com Linux, macOS ou Windows + WSL. Precisa ter **Ansible** e **Git** instalados. |
| 🌐 **Servidor remoto** | VPS/nuvem rodando **Ubuntu 24.04 LTS** (instalação padrão) e acesso SSH com usuário que possua `sudo`. |
| 🔑 **Domínios** | Dois registros **A** apontando para o IP do servidor, ex.: `traefik.seudominio.com` e `portainer.seudominio.com`. |

> Dica: Se não tem domínio, pode usar um subdomínio gratuito tipo DuckDNS.<br>
> Sem DNS apontado, o HTTPS automático NÃO funcionará.

---

## 3. Instalação em 5 passos

```bash
# 1. Clone este repositório
 git clone https://github.com/JotaSXBR/AnsibleClaude.git
 cd AnsibleClaude

# 2. Ajuste o inventário
 cp inventory.ini inventory.backup   # opcional, para ter um backup
 nano inventory.ini  # ou seu editor favorito
 #  – troque IP, usuário, domínios e e-mail

# 3. Teste conexão (opcional mas recomendado)
 ansible -i inventory.ini all -m ping

# 4. Execute o playbook
 ansible-playbook playbook.yml -i inventory.ini --ask-pass --ask-become-pass -v

# 5. Aguarde ±5 min e acesse
 https://traefik.seudominio.com  # dashboard traefik
 https://portainer.seudominio.com # crie o usuário admin
```

> • **--ask-pass**: solicita a senha SSH do usuário remoto (caso você não use chave privada).
> 
> • **--ask-become-pass**: pede a senha do `sudo` para executar tarefas como administrador.
> 
> • **-v**: ativa o modo verboso, exibindo detalhes extras durante a execução.

Ficou curioso sobre o que será feito? Rode em **modo simulação** primeiro:
```bash
ansible-playbook -i inventory.ini playbook.yml --check
```

---

## 4. Depois da execução

1. Abra o Portainer, crie o usuário administrador e clique em **Get Started**.
2. Se receber erro de certificado, verifique se o DNS já propagou (pode levar alguns minutos).
3. Faça backup de `/opt/traefik/acme/acme.json` – lá vivem seus certificados.

---

## 5. Estrutura de arquivos

| Arquivo | Para que serve? |
|---------|-----------------|
| `playbook.yml` | Receita principal com todas as tarefas. |
| `inventory.ini` | Aqui você coloca informações do(s) servidor(es). |
| `README.md` | Este guia. |

---

## 6. Perguntas frequentes

**O playbook altera a senha root?**  
Não. Ele desabilita o login direto como root por segurança, mas a senha permanece.

**Posso adicionar mais de um servidor?**  
Sim! Basta criar novas linhas em `[servers]` no `inventory.ini`.

**Quanto custa o Traefik/Portainer?**  
Ambos são gratuitos nas versões usadas aqui.

**Quero remover o que foi instalado.**  
Você pode destruir os stacks (Docker) ou rodar outro playbook de rollback. Consulte a documentação oficial do Docker para remover Swarm.

---

## 7. Troubleshooting rápido

| Sintoma | Possível causa | Comando de verificação |
|---------|---------------|-------------------------|
| Porta 443 não abre | Firewall | `sudo ufw status` |
| Login SSH bloqueado | Fail2Ban baniu seu IP | `sudo fail2ban-client status sshd` |
| HTTPS inválido | DNS não propagou | `nslookup traefik.seu.domínio` |
| Stacks parados | Serviço caiu | `docker service ls` |

---

## 8. Contribuindo
Achou bug, tem sugestão ou quer melhorar algo? Abra uma *issue* ou envie um *pull request* – a comunidade (você!) faz o projeto crescer.

---

## 9. Licença
Distribuído sob a licença MIT. Use à vontade, mas sem garantias. 😉