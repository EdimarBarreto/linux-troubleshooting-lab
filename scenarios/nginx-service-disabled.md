# Gerenciamento de Serviços: Nginx-Service-Disabled/Enabled?

Neste tópico, exploramos o funcionamento dos serviços gerenciados pelo **systemd/systemctl** e como realizar diagnósticos eficazes.

---

## Ambiente de Testes
Para realizar estes testes, foi utilizada a seguinte infraestrutura:

| Item | Descrição |
| :--- | :--- |
| **Infraestrutura** | VM EC2 da AWS |
| **Sistema Operacional** | Ubuntu 24.04 |
| **Instalação** | Nginx via gerenciador de pacotes APT (padrão Debian/Ubuntu) |
| **Init System** | systemd nativo do sistema |

---

## Alguns Esclarecimentos

### O que é o systemd?
> Ele é um gerenciador de serviços, responsável por inicializar os serviços instalados em seu sistema operacional de forma automática. Quando o S.O. está terminando de subir, o systemd entra em ação, subindo serviços relacionados à rede, firewall e outros componentes essenciais.

### O que é o Nginx?
> É um Servidor Web open source de alto desempenho. Com ele, conseguimos recursos como proxy reverso, balanceador de carga, cache HTTP, entre outros. É uma das peças fundamentais da internet hoje para otimizar desempenho e segurança.

### O que é o Troubleshooting?
> É um processo sistemático de identificar, analisar e resolver problemas em sistemas e equipamentos. Usa lógica e métodos estruturados para restaurar a funcionalidade, minimizando tempo e custos, além de documentar soluções para referência futura.

---

## O Teste Realizado

### 1. Parando o serviço
Realizamos a parada do serviço Nginx instalado no sistema:
```bash
systemctl stop nginx.service
```

### 2. Desabilitando o serviço
Aqui, desativamos o gerenciamento automático (inicialização no boot) pelo systemd:
```bash
systemctl disable nginx.service
```

### 3. Iniciando o serviço
Executamos o serviço novamente de forma manual:
```bash
systemctl start nginx.service
```

> **Cenário Atual:** Temos um serviço Nginx funcionando corretamente agora. Porém, no momento em que este servidor for reiniciado, o Nginx **não subirá** junto ao sistema. Como analisar e resolver isso?

---

## O Troubleshooting

### Como realizar?
Partimos do princípio da investigação do **Serviço**, fazendo as perguntas certas:
* O serviço está ativo? 
* O serviço está inativo? 
* O serviço está com problema? 
* O que os logs mostram?

Para essas validações, utilizaremos o `systemctl` e o `journalctl`.

### Analisando os logs
Utilizamos o `journalctl` para visualizar o histórico do serviço:
```bash
# Para ver o log estático
journalctl -u nginx.service

# Para acompanhar os logs em tempo real
journalctl -u nginx.service -f
```
*Nota: O `-f` (follow) serve para "ouvir" os logs continuamente em vez de apenas mostrar um bloco e encerrar.*

### Analisando o status do serviço
Para ver o estado atual, usamos o comando `status`:
```bash
systemctl status nginx.service
```

Se o serviço estiver parado, iniciamos e checamos o status novamente para ver o diagnóstico completo:
```bash
systemctl start nginx.service
systemctl status nginx.service
```

#### Identificando o Problema
Ao rodar o status, observe atentamente a saída:

```text
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: enabled)
     Active: active (running) since Mon 2026-02-02 21:58:23 -03; 2s ago
     ...
```

Repare na palavra **disabled**. Isso indica que o serviço não possui mais o link simbólico ativo para os *targets* de boot, pois o desabilitamos anteriormente. Para corrigir:

```bash
systemctl enable nginx.service
```
**Pronto! Problema resolvido!**

---

## Active x Enabled no systemd

| Estado | Descrição |
| :--- | :--- |
| **Active (running)** | Indica que o serviço está em execução no momento. |
| **Enabled** | Indica que o serviço está configurado para iniciar automaticamente no boot. |

Um serviço pode estar ativo e ainda assim estar desabilitado (caso tenha sido iniciado manualmente). O comando `systemctl enable` garante a execução automática após reinicializações.

---

# Obrigado por ler até aqui! 
Nos vemos nos próximos conteúdos dessa jornada!