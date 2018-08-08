---
layout: post
title: JSON-B, trabalhando com JSON através das especificações Java
date: '2018-01-08 10:02:00'
tags:
- java
- json
- jsonb
- json-b
- jsr353
- jsr367
- yasson
---

Com a evolução dos sistemas, cada vez mais precisamos nos comunicar com outros aplicações, sejam elas aplicações externas, como uma consulta na API do Facebook, Github ou até mesmo em aplicações internas, como *microservices*. 
Hoje em dia, a maioria dessas mensagens em sistemas modernos, utilizam como formato de comunicação o JSON, mas sempre sentimos falta de uma API/biblioteca do próprio Java para realizar os *parses* de JSON para objeto ou vice versa e também os *binds*. Para resolver isso, acabamos utilizando *frameworks* como o [Jackson](https://github.com/FasterXML/jackson) e o [Gson](https://github.com/google/gson), que por sinal são excelentes.
Para resolver este problema, desde o Java EE7 já começaram algumas especificações, como a [JSR 353](https://www.jcp.org/en/jsr/detail?id=353) para realizar o processamento do JSON, realizando *parses*, geração, transformação e etc. Para completar, após alguns anos surgiu uma nova especificação, a [JSR 367](https://jcp.org/en/jsr/detail?id=367), conhecida também como JSON-B, trazendo melhorias para trabalhar com JSON.
Sendo assim, agora existem mais possibilidades e opções de bibliotecas para trabalhar com o formato JSON, além do Java oferecer tal funcionalidade, que para XML já existe faz algum tempo, o JAXB. Para a demonstração do JSON-B, será utilizado a implementação de referência para a API, o [Eclipse Yasson](https://github.com/eclipse/yasson), mas não é a única implementação, existe o [Apache Johnzon](https://johnzon.apache.org/) também.

Para começar, o projeto deve ter as seguintes dependências:
````xml
<dependencies>
    <dependency>
        <groupId>org.eclipse</groupId>
        <artifactId>yasson</artifactId>
        <version>1.0.1</version>
    </dependency>
    <dependency>
        <groupId>org.glassfish</groupId>
        <artifactId>javax.json</artifactId>
        <version>1.1.2</version>
    </dependency>
</dependencies>
````

Como classe de exemplo, será utilizada a seguinte:
````java
public class User {

    private Integer id;

    @JsonbTransient
    private String name;

    private String username;

    @JsonbProperty("passwd")
    private String password;

    private String email;

    @JsonbDateFormat("dd/MM/yyyy")
    private LocalDate createdAt;

    //getters and setters
}
````

Aqui já é possível ver algumas anotações:
- *@JsonbTransient:* Para ignorar o campo na serialização e desserialização.
- *@JsonbProperty:* Definir qual será o nome do campo no Json, caso seja diferente do que está na classe.
- *@JsonbDateFormat:* Trocar o formato da data.

Essas são apenas algumas anotações, existindo outros para cada funcionalidade específica.

Após isso, vamos utilizar o seguinte usuário como exemplo:
````java
User user = User.builder()
                .id(1)
                .name("Lucas")
                .username("furlaneto")
                .password("123")
                .email("lucas@techiohub.com")
                .createdAt(LocalDate.of(2018, 1, 1))
                .build();
````

Com o nosso objeto já pronto, agora é só instanciar o nosso **Jsonb** para começarmos a manipular.
````java
Jsonb jsonb = JsonbBuilder.create();
````

Para transformar o objeto para JSON, simplesmente execute o seguinte método:
````java
String jsonUser = jsonb.toJson(user);
````

Ao visualizar a *String* **jsonUser** terá o seguinte conteúdo:
````json
{
    "createdAt": "01/01/2018",
    "email": "lucas@techiohub.com",
    "id": 1,
    "passwd": "123",
    "username": "furlaneto"
}
````

Para realizar a conversão contrária, basta utilizar o método *fromJson* e vamos ter o nosso **User** novamente na forma de objeto
````
User [id=1,name="null",username="furlaneto",password="123",email="lucas@techiohub.com",createdAt=2018-01-01]
````

Esta foi apenas uma pequena demonstração das especificações, mas existem outras funcionalidades que também são encontradas nos frameworks citadas no início do post. Caso queira conhecer mais sobre o JSON-B pode accessar os links úteis no fim do post e também conferir o projeto por completo utilizado como exemplo.

**Links úteis**
- [Site JSON-B](http://json-b.net/)  
- [Exemplo no Github](https://github.com/furlaneto/jsonb-example)  