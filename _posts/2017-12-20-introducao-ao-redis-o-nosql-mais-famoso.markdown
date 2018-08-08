---
layout: post
title: Introdução ao Redis, o NoSQL chave-valor mais famoso
date: '2017-12-20 11:34:00'
tags:
- nosql
- redis
---

Antes de falarmos de Redis, o que é NoSQL? Creio que muitos já sabem o que é, mas alguns ainda possuem aquela dúvida. NoSQL (referência de non SQL), é o termo utilizado para descrever os bancos de dados não relacionais. Os bancos NoSQL, diferente dos bancos tradicionais como Oracle, MySQL, SQL Server e outros, não são necessariamente do mesmo tipo, pois cada um adota um ou mais modelo de dados. Estes modelos de dados podem ser documentos, chave e valor, colunar ou gráfo. 

Assim como o modelo de dados para resolver cada tipo de problema, os bancos NoSQL no geral possuem diferenças comparando aos bancos relacionais. Para começar, podemos dizer que os bancos NoSQL não possui um schema, ou seja, não é da mesma forma como os bancos convencionais são representados, através de uma normalização de dados por meio de tabelas, colunas e linhas. Outra diferença é a forma como escalamos os bancos NoSQL, onde quanto maior o nosso cluster, maior o desempenho, sendo mais fácil realizar o escalonamento horizontal, diferente dos bancos relacionais que são mais fácil realizar o escalonamento vertical, adicionando mais recursos a uma máquina, como memória e CPU. Essas são apenas algumas diferenças entre os bancos NoSQL e relacionais.

Mas onde o Redis se encaixa nisso? O Redis é o banco de dados NoSQL do tipo chave-valor mais famoso/conhecido. No mercado podemos encontrar diversas outras soluções que também são mecanismos de armazenamento baseado em chave-valor, incluindo Memcached e Hazelcast, mas vamos falar especificamente do Redis.

Para iniciarmos, o que seria um mecanismo de armazenamento do tipo chave-valor? Simples, é uma forma de armazenarmos um valor vinculado a uma chave, assim como estruturas encontradas em linguagens de programação, por exemplo, o [Map](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html) do Java. Para simplificar, podemos pensar em algo como várias gavetas, onde cada gaveta possui um nome, que não pode se repetir e dentro de cada gaveta possui um valor que representa a mesma. Por exemplo, cada gaveta representa o nome de uma pessoa e dentro dela possui o valor da idade, então se eu quero saber a idade do Lucas, vou até a gaveta "Lucas" e pego o valor, no caso, 22. Simples, não é? Este exemplo foi exatamente a estrutura mais simples de armazenamento do Redis, onde temos uma chave e um valor, podendo ser uma letra, palavra, texto ou número. Só que o Redis vai além disso, pois o valor da chave não precisa ser necessariamente algo tão simples como o exemplo anterior, o valor pode ser uma estrutura de dados um pouco mais complexa, como uma lista, conjunto ou hash.

Tudo bem, falamos o que é o Redis, mostramos o conceito básico dele e os tipos de dados, mas qual o seu uso no dia a dia do desenvolvedor? Alguns.

Acredito que quando ouvimos falar em chave-valor, estrutura de Map em Java ou em outras linguagens, a primeira coisa que vem na mente é cache e sim, um dos seus usos é para isso mesmo. Mas ok, vamos dizer que seja uma aplicação web e você me pergunte: Lucas, não posso fazer a mesma coisa com a sessão? Sim, podemos, mas se tivermos mais de um servidor executando nossa aplicação? O que é muito comum. Hora que o usuário acessar o servidor1 e salvar algo na sessão, depois acessar o servidor2, será que a informação estará lá também? Não, pois como estamos armazenando na sessão, isso ficará armazenado na memória do servidor e quando tentar acessar o outro servidor, não estará lá. É claro que quando fazemos um cluster, alguns servidores fornecem soluções, como o JBoss, fazendo a replicação de sessão através de UDP ou outras maneiras, mas também podemos utilizar o Redis de forma centralizada, onde qualquer informação que precisamos salvar ou consultar relacionada a sessão ou cache, vamos diretamente nele, assim, se o usuário estiver no servidor1 ou no servidor 2, ele terá a mesma informação.

Um outro caso que é possível utilizar o Redis é quando na página inicial de um site exibe totalizadores ou um ranking de games que muitas vezes não são dados que atualizam a todo momento e muitas vezes ao tentar recuperar estas informações diretamente de uma base relacional e realizar uma soma, média ou qualquer que seja a operação, demanda uma certa quantidade de recursos, prejudicando ainda mais a performance caso tenha um alto volume de acesso a estas informações. Sendo assim, para economizar recursos e consequentemente retornar estas informações com maior velocidade, é possível armazenar estes totalizadores diretamente no Redis, assim, quando o usuário acessar a página, não terá que recuperar uma quantidade de registros, depois fazer uma operação e retornar, mas já poderá recuperar estes valores diretamente. Consegue imaginar a economia de recurso e tempo? Para demonstrar esse exemplo no dia dia, podemos imaginar um dashboard ou apenas uma tela que mostra o totalizadores de itens vendidos ou o valor total de vendas do dia anterior, assim não precisamos mais recuperar todas as vendas do dia anterior e somar, apenas é enviado um comando para o Redis pedindo o total de vendas do dia anterior que o retorno já será o valor totalizado.

Para entender, nada melhor que a prática e para acompanhar os exemplos pode ser tanto por uma instalação local do Redis, realizando o download através [da página oficial](https://redis.io/download) ou caso não queira baixar ou instalar alguma coisa, o Redis disponibiliza um playground para brincar com ele e que pode ser acessado por [este link](https://try.redis.io). Como comentado anteriormente, a estrutura mais simples de chave e valor no Redis é a String, sendo da seguinte forma para inserir e recuperar um valor:

Inserir um valor
```
> SET chave valor
OK
```

Inserir múltiplas chaves com valores
```
> MSET chave1 valor1 chave2 valor2 chave3 valor3
OK
```

Recuperar
```
> GET chave
"valor"
```

Recuperar múltiplas chaves
```
> MGET chave1 chave2 chave3
1) "valor1"
2) "valor2"
3) "valor3"
```

Se quisermos armazenar uma lista, existem duas formas:

Inserir no lado esquerdo (no início)
```
> LPUSH lista valor1
(integer) 1
```

Inserir no lado direito (no fim)
```
> RPUSH lista valor2
(integer) 2
```

Mas e para recuperar as informações de uma lista? Deve utilizar o comando LRANGE e passar três parâmetros básicos para o mesmo, sendo a chave da lista, posição inicial e final.
```
> LRANGE lista 0 -1
1) "valor1"
2) "valor2"
```
Posição final -1? Sim, o Redis consegue contar de trás para frente, então qualquer um dos parâmetros de posições é possível passar um número negativo, sendo o -1 o último elemento, -2 o penúltimo e assim por diante.

Inserimos uma lista, só que podemos ter registros repetidos, mas se precisar que não repita os valores, é possível utilizar um conjunto, o SET.
```
> SADD conjunto 1 2 3 2 1
(integer) 3
```
Podemos ver que o retorno foi o número 3, pois como mencionamos anteriormente, só será adicionado no conjunto, caso o valor não exista. E para comprovar isso, podemos consultar.
```
> SMEMBERS conjunto
1) "1"
2) "2"
3) "3"
```

No exemplo de uso do Redis como cache de sessão de um servidor web mencionada no post, podemos utilizar a estrutura de hashes, onde existe uma chave, vinculada a vários atributos e seus valores.
```
> HMSET sessao:usuario:1 usuario lucas nome "Lucas Furlaneto" perfil administrador
OK
```
Para recuperar o valor de um dos atributos do hash, é só utilizar a palavra reservada HMGET e passar como parâmetro a chave e o atributo.
```
> HMGET sessao:usuario:1 perfil
1) "administrador"
```

Como estamos falando de sessão de usuário, é comum que após um certo período de tempo, o sistema expire a sessão e realize o logout do usuário. Para isso, o Redis já fornece o recurso através do comando EXPIRE, informando os parâmetros da chave e o tempo em segundos que deseja que essa chave expire.
```
> EXPIRE sessao:usuario:1 60
(integer) 1
```
Caso queira consultar o tempo ainda que resta para a chave expirar, basta utilizar a instrução TTL.
```
> TTL sessao:usuario:1
(integer) 44
```
Após o período informado para expirar, no caso 60 segundos, ao tentar recuperar essa chave, já não será mais possível.
```
> HMGET sessao:usuario:1 perfil
1) (nil)
```

Essa é uma pequena introdução ao mundo de Redis e algumas funcionalidades presentes em suas estruturas. Espero que com isso consigam ter entendido o conceito e um pouco mais sobre Redis. Caso queiram saber mais sobre Redis, vou deixar alguns links que podem consultar.

**Links Úteis**
* [Site oficial](https://redis.io/)  
* [Documentação](https://redis.io/documentation)  
* [Playground](https://try.redis.io/)  