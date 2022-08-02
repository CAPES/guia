- [Visão Geral](#visão-geral)
- [Como Acessar um Servidor via SSH](#como-acessar-um-servidor-via-ssh)
  - [via Mobaxterm no Windows](#via-mobaxterm-no-windows)
  - [via Terminal (CLI)](#via-terminal-cli)
  - [via Terminal (CLI) com Script](#via-terminal-cli-com-script)
  - [via Terminal (CLI) com configuração no `ssh_config`](#via-terminal-cli-com-configuração-no-ssh_config)
- [Notas e Referências](#notas-e-referências)

# Visão Geral
Para um controle nos acessos SSH aos servidores, foi implantada a solução **[FreeIPA](https://xpto.com/cgii/gerenciamento-de-identidade/freeipa)** (acesso restrito) que gerencia as permissões e o controle de acessos dos usuários aos grupos de servidores.

<br><br>

# Como Acessar um Servidor via SSH
## via Mobaxterm no Windows
Utilizando a ferramenta [Mobaxterm](https://mobaxterm.mobatek.net/download-home-edition.html):

1. Clique sobre o botão "SSH".
2. Na sequência pressione o botão "Network settings".
3. Preencha o campo "Gateway SSH server": `ssh.capes.gov.br`
4. Porta padrão: `22`
5. Defina o usuário que acessará os servidores. Exemplo:  `login`  ou  `login@fc.capes.gov.br`
   * Utilizar a conta **adm.xxx** sempre que possível.
6. Defina o servidor que deseja acessar "Remote host".


![Acesso-SSH-Mobaxterm](Acesso-ssh-mobaxterm.png)

<br><br>

## via Terminal (CLI)
Em um terminal (Linux, MacOS ou Windows pelo MobaXterm, digite o comando, alterando o `<login>` e o `<servidor_destino>`, conforme necesidade.

```
ssh <login>@fc.capes.gov.br@<servidor_destino>.capes.gov.br -J <login>@ssh.capes.gov.br
```

<br><br>

## via Terminal (CLI) com Script
Para agilizar a digitação, um script pode ser usado, para usar o freeipa como gateway de acesso ssh 
> Execute na sua estação trabalho linux, MacOS ou no MobaXterm. Altere o `<login>` com sua conta de Rede CAPES.

* Crie o arquivo com o conteúdo abaixo:
```
vi /usr/bin/gw
```


```bash
 #!/bin/bash
        if [ -z "$1" ]; then
                ssh -A <login>@fc.capes.gov.br@ipa.hom.capes.gov.br
        else
                ssh -A <login>@fc.capes.gov.br@ipa.hom.capes.gov.br -t "ssh -o StrictHostKeyChecking=no $1"
       fi
```

* Aplique permissão de execução

```
chmod a+x /usr/bin/gw
```

* Agora basta executar 

```
gw <servidor>
```

<br><br>

## via Terminal (CLI) com configuração no `ssh_config`
O arquivo `~/.ssh/config`[^man-ssh-config] é o arquivo de configuração que pode ser utilizado para simplificar as configurações de acesso a _hosts_, como as necessárias para utilizar o FreeIPA como `ProxyJump`[^man-ssh-config] que o argumento `-J`[^man-ssh] indica.


---
**Pré-requisitos**:
```bash
# A pasta ~/.ssh provavelmente já existe...
mkdir -p ~/.ssh && chmod 600 ~/.ssh
# O arquivo ~/.ssh/config talvez já exista
touch ~/.ssh/config
```
---

Adicione o conteúdo a seguir no arquivo `~/.ssh/config`, substituido `<login>` pelo login de rede a ser utilizado

```ssh_config
Host xpto.com
    User git
Host ssh.capes.gov.br
    User <login>
Host *.capes.gov.br !ssh.capes.gov.br !xpto.com
    User <login>
    ProxyJump ssh.capes.gov.br
```

Com essas configuração é possível fazer simplesmente:

```bash
ssh <servidor_destino>.capes.gov.br
```

<br><br>

# Notas e Referências

[^man-ssh-config]: Vide `man ssh_config` - https://linux.die.net/man/5/ssh_config
[^man-ssh]: Vide `man ssh` - https://linux.die.net/man/1/ssh
