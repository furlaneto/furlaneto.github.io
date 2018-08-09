---
layout: post
title: Facilitando os testes de integração em Java com Testcontainers
date: '2018-08-09 13:32:21'
tags:
- java
- teste
- testcontainers
- junit
- docker
- container
---

Facilitando os testes de integração em Java com Testcontainers

Nos dias atuais, cada vez mais nós integramos com diversas ferramentas, podendo ser elas um banco de dados (Ex: MySQL, Redis, Mongo), um servidor web/proxy (Ex: nginx, Apache) ou até mesmo algum outro serviço que é desenvolvido pelo próprio time ou empresa. Muitas vezes apenas queremos subir alguma dessas dependências, executar os nossos testes e descer as mesmas para que não fique consumindo recursos sem necessidade.

Com o surgimento do Docker conseguimos tornar mais fácil nossa vida, empacotando cada dependência em um container, iniciando ele e depois parando, mas ainda assim é necessário ter um gerenciamento manual do ciclo de vida desses containers. Para que seja possível realizar a comunicação com os containers, ainda é necessário recuperar o IP ou a porta aberta e passar para os nossos testes para que seja possível executar.

Para resolver todos esses nosso problemas, podemos utilizar o Testcontainer, uma incrível biblioteca Java que controla todo esse ciclo de vida dos containers dockers e que possui uma integração com JUnit, facilitando ainda mais o nosso trabalho. O Testcontainer possui dois tipos de containers que você pode trabalhar com ele, o container genérico e o container específico que vamos ver em seguida:

Antes de qualquer coisa precisamos importar as dependências em nosso projeto.

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.8.3</version>
</dependency>
```

## Container Genérico

Como o próprio nome já diz, é um container genérico, sendo mais flexível e que aceita qualquer imagem que você especifique para ele, como um serviço desenvolvido interno. Como exemplo, vamos utilizar o da própria documentação.

```java
@ClassRule
public static GenericContainer alpine =
    new GenericContainer("alpine:3.2")
            .withExposedPorts(80)
               .withEnv("MAGIC_NUMBER", "42")
               .withCommand("/bin/sh", "-c", 
               "while true; do echo \"$MAGIC_NUMBER\" | nc -l -p 80; done");
```

Estamos criando uma **@ClassRule** do próprio JUnit que irá iniciar o container antes dos testes e destrui-lo após todos os testes, mas também é possível utilizar a anotação **@Rule** que irá criar uma nova instância do container antes da execução de cada teste. Na construção de um *GenericContainer* nós passamos qual será a imagem base para a criação do container e após a construção é possível informar algumas configurações como portas que serão expostas pelo container, variáveis de ambiente, comando inicial que será executado e muitos outros. Lembrando que são as mesmas configurações de quando criamos um container através do docker.

Após o container ser inicializado é possível recuperar algumas informações, como o IP e porta mapeada, que seriam as informações neccessárias caso o nosso container fosse um banco de dados ou um servidor web. Vamos utilizar um container genérico de Redis como exemplo.

```java
@ClassRule
public static GenericContainer redis =
    new GenericContainer("redis:4")
               .withExposedPorts(6379);

@Test
public void getConnectionInfoFromRedis() {
    String redisUrl = redis.getContainerIpAddress() + ":" + redis.getMappedPort(6379);
}

```

## Container Específico

Os containers específicos herdam de *GenericContainer*, com uma única diferença que ao utiliza-los você está dizendo que irá trabalhar com um container específico e irá ganhar de brinde alguns métodos a mais para recuperar informações como o usuário, senha e URL de conexão do banco de dados, mas que irá variar de acordo com o container que você está trabalhando, pois cada um terá sua peculiaridade.

```java
@Rule
public MySQLContainer mysql = new MySQLContainer();

@Test
public void getMySQLInfos() {
    mysql.getJdbcUrl();
    mysql.getUsername();
    mysql.getPassword();
}
```

Por ser algo específico, é necessário importar as dependências de cada um que for utilizar.

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>mysql</artifactId>
    <version>1.8.3</version>
</dependency>
```

Essa é uma pequena introdução do Testcontainer que possui muitas outras funcionalidades além dessas apresentadas, mas já podemos perceber que isso nos ajuda de mais no dia a dia para a execução de testes de integração, evitando até mesmo aquela velha história "Na minha máquina funciona".

## Links úteis  
[- Documentação Testcontainers](https://www.testcontainers.org/)