
# Visão Geral
Para um controle nos acessos SSH aos servidores, foi implantada a solução **[FreeIPA](https://git.capes.gov.br/cgii/gerenciamento-de-identidade/freeipa)** (acesso restrito) que gerencia as permissões e o controle de acessos dos usuários aos grupos de servidores.

<br><br>

# Como Acessar um Servidor via SSH
## Com Mobaxterm 
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

## Com Terminal Linux
Para simplificar o acesso, pode ser utilizado o script abaixo.

Crie o arquivo `gw` na sua estação de trabalho (linux).
```
vi /usr/bin/gw
```

Insira o seguinte conteúdo:
> Troque o `seu_login_de_rede` pelo seu login da Rede Capes.

```bash
 #!/bin/bash
        if [ -z "$1" ]; then
                ssh -A (seu_login_de_rede)@fc.capes.gov.br@ssh.capes.gov.br
        else
                ssh -A (seu_login_de_rede)@fc.capes.gov.br@ssh.capes.gov.br -t "ssh -o StrictHostKeyChecking=no $1"
       fi
```

Aplique permissão de execução

```
chmod a+x /usr/bin/gw
```

Agora basta executar 

```
gw <servidor>
```