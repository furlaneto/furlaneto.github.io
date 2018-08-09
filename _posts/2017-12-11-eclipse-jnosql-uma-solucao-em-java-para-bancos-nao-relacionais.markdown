---
layout: post
title: Eclipse JNoSQL, uma solução em Java para bancos não relacionais
date: '2017-12-11 13:53:20'
tags:
- java
- nosql
- jnosql
---

<p align="justify">
Em Java, muitas vezes facilitamos nossas vidas com <a target="_blank" href="http://www.oracle.com/technetwork/java/javaee/tech/persistence-jsp-140049.html">JPA</a> devido sua simplicidade e facilidade na utilização dos bancos relacionais como: Oracle, MySQL, SQL Server e demais. Apenas com algumas configurações e anotações e já temos um CRUD em pleno funcionamento, além de nos dar a possibilidade de realizar consultas de uma forma mais orientada a objetos, que é o que a <a target="_blank" href="https://docs.oracle.com/javaee/7/tutorial/persistence-querylanguage.htm">JPQL</a> permite. Também ganhamos na facilidade de troca de banco, já que uma vez que o código esteja utilizando JPA, qualquer que seja o banco de dados suportado pela especificação, funcionará da mesma forma, sendo necessário refatorar o código apenas se utilizar coisas específicas do banco de dados.
</p>
<p align="justify">
Mas afinal, o que JPA tem a ver com o tema do tópico, Eclipse JNoSQL? O <i>framework</i> veio para facilitar as nossas vidas, mas no mundo dos bancos de dados não relacionais, o que podemos dizer que acaba sendo um pouco mais complexo, pois assim como os relacionais, alguns bancos podem oferecer funcionalidades encontradas apenas nele e não para por ai, pois existem diferentes tipos de bancos de dados, sendo eles:
</p>
<li>Chave e valor;</li>
<li>Documento;</li>
<li>Coluna;</li>
<li>Gráfo.</li>  
<p align="justify">
Imagine só, você ter que entender o funcionamento de todos os bancos de dados não relacionais, mesmo que seja apenas de um tipo, ter que configurar um driver de conexão para cada um, utilizar uma forma de fazer para cada, um jeito de consultar em um e outro jeito de consultar em outro, além de operações de <i>insert</i>, <i>update</i> e <i>delete</i>. Isso requer tempo, pois existe uma curva de aprendizado e muitas vezes queremos algo rápido e que possa abstrair boa parte das coisas, assim com o JPA faz e é ai que o JNoSQL entra. Com ele, podemos pular essa curva de aprendizado, pois além dele abstrair toda essa questão de configuração para cada banco, ele também tenta ser de uma forma que o desenvolvedor que já está acostumado com o mundo relacional e suas ferramentas, como o JPA, consiga utilizar com maior facilidade.
</p>
<p align="justify">
Para solucionar os problemas, o projeto é dividido em duas partes principais, sendo:
<li><strong>Diana:</strong> Camada de comunicação com os bancos NoSQL, funcionando da mesma forma que o JDBC faz para os bancos de dados relacionais. O projeto é divido em um módulo para cada tipo de banco.</li>
<li><strong>Artemis:</strong> Camada mais alto nível, onde acontece o mapeamento, integração com Bean Validation, como no JPA e que transforma no modelo em que o Diana possa entender. Assim como o Diana, também é composto por módulos, onde cada um representa um tipo de banco.</li>
</p>
<p align="justify">
Este só foi um post introdutório no assunto, para demonstrar o problema que o framework resolve e também para mostrar que existem solucões voltadas para o NoSQL. Para aqueles que desejam saber mais, vou continuar com uma série de posts demonstrando a utilização do mesmo em diferentes bancos de dados, sendo de documento, coluna, chave-valor e gráfo.
</p>
<p align="justify">
    <strong>Curiosidade:</strong> O JNoSQL foi criado por um brasileiro, <a target="_blank" href="https://github.com/otaviojava">Otávio Santana</a>, palestrante em diversos eventos como o JavaOne e também commiter da Apache e OpenJDK, além de ter ajudado diversos outros projetos como Apache Commons, Hibernete e etc.
</p>
<strong>Links Úteis:</strong>
<li><a target="_blank" href="http://www.jnosql.org/">Site JNoSQL</a></li>
<li><a target="_blank" href="https://www.gitbook.com/book/jnosql/jnosql-book/details">Livro JNoSQL</a></li>
<li><a target="_blank" href="https://github.com/eclipse/jnosql-artemis">Artemis GitHub</a></li>
<li><a target="_blank" href="https://github.com/eclipse/jnosql-diana">Diana GitHub</a></li>