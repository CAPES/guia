[[_TOC_]]

# Introdução

Este guia tem por objetivo auxílio aos desenvolvedores nas tarefas do cotidiano no uso do git para fácil referência.

# Instalação

## Git para todas as plataformas
[https://git-scm.com/downloads](https://git-scm.com/downloads)

## Clientes de Interface Gráfica

[https://git-scm.com/downloads/guis](https://git-scm.com/downloads/guis)

# Principais comandos

>NOTA: Este agrupará os comandos por afinidade e não por ordem de execução

## Configuração

Configurando Informações do Usuário Utilizadas em Todos os Repositórios Locais

- ```git config --global user.name “[firstname lastname]”```
  > Defina um nome identificável para crédito ao revisar o histórico de versões


- ```git config --global user.email “[valid-email]”```
  > Defina um endereço de email que será associado a cada marcador do histórico


- ```git config --global color.ui auto```
  > Defina a coloração automática da linha de comando para o Git para facilitar a revisão
  
  
## Iniciar um projeto
Configurando Informações do Usuário, Inicializando e Clonando Repositórios

- ```git init```
   > Inicializar um diretório existente como um repositório Git
   

## Clonando um projeto   

- ```git clone [url]```
  > Ex: ```git clone https://xpto.com/cgs/csae/freire/freire.git```
  
## Branches

- ```git branch```
  > Lista todos os branches locais no repositório atual 

- ```git branch [nome-do-branch]```
  > Cria um novo branch 

- ```git checkout -b [nome-do-branch]```
  > Cria um novo branch e faz o switch para ela

- ```git checkout -b [nome-do-branch] <remote>/[nome-do-branch]```
  > Cria um novo branch local a partir da branch remota e faz o switch para ela

- ```git checkout [nome-do-branch]```
  > Muda para o branch específico e atualiza o diretório de trabalho 

- ```git merge [branch]```
  > Combina o histórico do branch específico com o branch atual 

- ```git branch -d [nome-do-branch]```
  > Exclui o branch específico
  
## Interagindo com servidor

- ```git push <remote> <branch>```
  > Especifica uma branch remota para receber commits feitos na branch e torna essa branch o default para o push

- ```git push```
  > Efetua o envio dos commits feitos para o servidor 

- ```git fetch```
  > Efetua o download do conteúdo do repositório do servidor para a cópia local, considerado mais seguro, pois não modifica sua cópia local (branches), o processo de merge deve ser feito logo em seguida.

- ```git pull```
  > Efetua o download do conteúdo e atualiza imediatamente a cópia local. Caso tenha commits não enviados que estão em conflito, inicia o fluxo de mesclagem.
   
## Revisão de Código

- ```git status```
  > Lista todos os arquivos novos ou modificados para serem commitados

- ```git diff```
  > Mostra diferenças no arquivo que não foram realizadas

- ```git add [arquivo]```
  > Faz o snapshot de um arquivo na preparação para versionamento

- ```git diff --staged```
  > Mostra a diferença entre arquivos selecionados e a suas últimas versões

- ```git reset [arquivo]```
  > Desseleciona o arquivo, mas preserva seu conteúdo



# Dicas

## Ignorando arquivos
Impedindo o _staging_ não intencional ou confirmação de arquivos

### .gitignore
Salve um arquivo com os padrões desejados como .gitignore com uma sequência direta
correspondências ou wildcard globs

**Ex: .gitignore**
```
logs/
*.notes
pattern*/
```

### Git config
- ```git config --global core.excludesfile [arquivo]```
  > todo o sistema ignora o padrão para todos os repositórios locais
