# Estratégias de _Deploy_/Entrega

É importante saber discriminar as estratégias de implantação e liberação, por isso vamos comentar algumas, mas é importante que as pessoas dos times de desenvolvimento estudem e entendam esses conceitos, para saber quando utilizar a _Feature Flags_ para permiter tais estratégias.

## Canary[^canary-release]

Quando uma nova versão de código está sendo implantada ela é um risco. Vamos imaginar que o ideial seria que ela só ficasse disponível para um percentual de requisições no início (digamos 5%). Nesse momento a _feature flag_ ainda está "`off`". Isso serve pra ver se o deploy foi "okay". Se mesmo com a feature desligada nenhum problema ocorreu com esses usuários, então chega a vez dos _beta tests_.

## Alpha->Beta->Release

Essa sequência, é o que controla o ciclo de entrega em muitos aspectos. Podemos abstrair que o que está em Alpha é o que está interno (DHT). O que está em Beta deveria estar em produção sendo servido/disponibilizado por um conjunto reduzido de _users_. Quando finalmente a funcionalidade é liberada ("_released_") é que ela foi disponibilizada para o público total da aplicação (dada suas permissões).

## _Feature Toggle/Flags_ vs Permissões

Imaginemos que uma nova funcionalidade esteja sendo desenvolvida para um determinado perfil (por exemplo Moderadores), mas será testada apenas pelo "_beta users_" dessa categoria. Os "programas de _beta users_" afetam os usuários meio que ortogonalmente a seus perfis. Imaginemos que tenhamos coisas como assinaturas. Na CAPES não é bem o caso, mas existe uma vasta gama de necessidades negociais diferentes na CAPES.

## "A/B _Testers_" vs "_Beta Users_"

Aqui a ideia fundamental parece ser que "_Beta Users_" (ou "_Beta Testers_") estão cientes disso. Enquanto "A/B _Testers_", não estão concientes disso. Imagine o seguinte cenário: uma nova "funcionalidade" tem 2 versões (A e B), o exemplo mais clássico é introduzir um novo botão em 2 cores diferentes. Claro que isso roda ainda na fase Beta. Para estabelecer esse teste da cor do novo botão, digamos a _feature_ do novo botão em si continuaria indisponível para 80% dos usuários, mas dos 20% restantes metade (10%) vai ver a cor 01 (digamos laranja) e a outra metade vai ver a cor 02 (digamos roxa). (Outras opções como o lado em que o botão fica - a direita ou a esquerda - poderiam ser testadas com "A/B").

> Na CAPES não parece haver muito sentido de "A/B _Testing_". Não há ideias de conversão (algo virar dinheiro). Claro que poderiam ser feitos experimentos para por exemplo reduzir erros, aumentar a velocidade com que os usuários fazem algo, ou até o caso de 2 algoritmos diferentes para a mesma nova funcionalidade, mas me parece extremamente rara a necessidade. _Beta Testting_ parece ser suficiente para os cenários gerais.

## _Blue Green Deployments_

Acho que vale a nota, isso faria mais sentido para quando temos de fazer manutenção. Do tipo: enquanto atualiza o cluster K8s na CAPES, o cluster na RNP ficaria respondendo pelas aplicações. Quando concluir essa atualização, o cluster na CAPES voltaria a responder, e a mesma atualização poderia ser feita no cluster na RNP. Após isso o balanceamento padrão ocorreria.

# Notas e Referências

[^canary-release]: CanaryRelease - https://martinfowler.com/bliki/CanaryRelease.html
