---
layout: post
title: Trabalhando com Redis no Java
date: '2018-02-07 16:25:00'
tags:
- java
- nosql
- jnosql
- redis
---

Redis é uma estrutura de armazenamento de dados open source, baseado no conceito de chave e valor. Seus dados são salvos em memória, possuindo a opção de informar o tempo de vida da chave, além de ter diversos tipos de estruturas para armazenamento. Para saber mais e entrar a fundo no conceito e utilização do Redis, você pode conferir [neste artigo](https://techiohub.com/introducao-ao-redis-o-nosql-mais-famoso/).

Para maior facilidade e comodidade na demonstração da utilização de Redis com Java, a instalação pode ser realizada via docker, através do seguinte comando:

````
docker run --name redis -p 6379:6379 -d redis
````

# **Integração com Java**

Para a demonstração de Redis com Java, será utilizado o **JNoSQL**, um *framework* que abstrai toda a comunicação com os diferentes tipos de bancos NoSQL, da mesma forma que todos estão acostumados com o JPA nos bancos relacionais. Caso você queria saber mais a respeito do JNoSQL, pode conferir através [deste post](https://techiohub.com/eclipse-jnosql-uma-solucao-em-java-para-bancos-nao-relacionais/).
Para este exemplo, você precisará utilizar Java 8, ou superior e CDI. Além desses requisitos, assim como em todo projeto, é necessário declarar as dependências a serem utilizadas.

````xml
    <properties>
        <maven.compile.targetLevel>1.8</maven.compile.targetLevel>
        <maven.compile.sourceLevel>1.8</maven.compile.sourceLevel>
        <maven.compile.version>3.5.1</maven.compile.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <json.b.version>1.0.1</json.b.version>
        <javax.json.version>1.1</javax.json.version>
        <javax.json.bind>1.0</javax.json.bind>
        <jnosql.version>0.0.4-SNAPSHOT</jnosql.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.jboss.weld.se</groupId>
            <artifactId>weld-se-shaded</artifactId>
            <version>3.0.1.Final</version>
        </dependency>
        <dependency>
            <groupId>javax.json</groupId>
            <artifactId>javax.json-api</artifactId>
            <version>${javax.json.version}</version>
        </dependency>
        <dependency>
            <groupId>javax.json.bind</groupId>
            <artifactId>javax.json.bind-api</artifactId>
            <version>${javax.json.bind}</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse</groupId>
            <artifactId>yasson</artifactId>
            <version>${json.b.version}</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish</groupId>
            <artifactId>javax.json</artifactId>
            <version>${javax.json.version}</version>
        </dependency>

        <dependency>
            <groupId>org.jnosql.artemis</groupId>
            <artifactId>artemis-key-value</artifactId>
            <version>${jnosql.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jnosql.diana</groupId>
            <artifactId>redis-driver</artifactId>
            <version>${jnosql.version}</version>
        </dependency>
    </dependencies>
````

Nas dependências é possível ver que é utilizado o [JSON-B](http://json-b.net/), a [especificação](https://jcp.org/en/jsr/detail?id=367) Java para padronização de serialização e desserialização de JSON. Isso é devido o funcionamento do JNoSQL, pois no Redis todos os valores são tratados como *String*, dessa forma, se for informado um valor que seja diferente de uma String, ele irá transformar para JSON.

# **Utilização**

Antes de começar é preciso configurar os Producers do nosso bucket, list, set, etc. As configurações podem ser conferidas [nessa classe](https://github.com/furlaneto/jnosql-redis-example/blob/master/src/main/java/com/example/jnosql/redis/RedisProducer.java).

Como classe modelo, será utilizada a seguinte:

````java
@Entity
public class God implements Serializable {

    @Id
    private String id;
    private String power;
    private Set<String> duties;
    
    @JsonbCreator
    public God(@JsonbProperty("id") String id,
        @JsonbProperty("power")String power,
        @JsonbProperty("duties")Set<String> duties) {
        this.id = requireNonNull(id, "id is required");
        this.power = requireNonNull(power, "power is required");
        this.duties = requireNonNull(duty, "duties is required");
    }
}
````

Nessa classe é possível visualizar a integração entre **JNoSQL** (@Entity, @Id) e **JSON-B** (@JsonbCreator, @JsonbProperty).

Para realizar a manipulação é possível através de duas formas, por *Template* ou *Repository*.

*Template*
````java
public class Main {
    public static void main (String[] args) {
        try (SeContainer container = SeContainerInitializer.newInstance().initialize()) {
            Set<String> duties = new HashSet<>();
            duties.add("music");
            duties.add("poetry");
            duties.add("medicine");
            God apollo = God.builder()
                    .id("Apollo")
                    .power("Sun")
                    .duties(duties)
                    .build();
            KeyValueTemplate keyValueTemplate = container.select(KeyValueTemplate.class).get();
            God godSaved = keyValueTemplate.put(apollo);
            System.out.println(godSaved);
            Optional<God> godFound = keyValueTemplate.get("Apollo", God.class);
            System.out.println(godFound);
        }
    }
}
````

*Repository*
````java
public interface GodRepository extends Repository<God, String> {
}
````

Utilizando o *Repository*
````java
public class Main2 {
    public static void main (String[] args) {
        try (SeContainer container = SeContainerInitializer.newInstance().initialize()) {
            Set<String> duties = new HashSet<>();
            duties.add("moon");
            duties.add("hunt");
            God artemis = God.builder()
                    .id("artemis")
                    .power("archery")
                    .duties(duties)
                    .build();
            GodRepository repository = container.select(GodRepository.class, DatabaseQualifier.ofKeyValue()).get();
            God godSaved = repository.save(artemis);
            System.out.println(godSaved);
            Optional<God> godFound = repository.findById("artemis");
            System.out.println(godFound);
        }
    }
}
````

# **Estruturas Padrões**

Para facilitar a vida dos desenvolvedores Java, as estruturas padrões forão abstraídas através do [Jedis](https://github.com/xetorthio/jedis), assim, ao trabalhar com uma lista, conjunto do Redis ou outra estrutura, você irá manipular de forma transparente através das classes Java, como é o caso do *List*, *Set*, *Queue* e *Map*.

*List*
````java
public class Main3 {
    public static void main (String[] args) {
        try (SeContainer container = SeContainerInitializer.newInstance().initialize()) {
            List<String> gods = container.select(new TypeLiteral<List<String>>() {}).get();
            gods.clear();
            System.out.println("Adds new gods");
            gods.add("Artemis");
            gods.add("Diana");
            gods.add("Diana");
            gods.add("Poseidon");
            gods.forEach(System.out::println);
            System.out.println("Removing an element");
            gods.remove("Diana");
            gods.forEach(System.out::println);
        }
    }
}
````

*Set*
````java
public class Main4 {
    public static void main (String[] args) {
        try (SeContainer container = SeContainerInitializer.newInstance().initialize()) {
            Set<String> gods = container.select(new TypeLiteral<Set<String>>() {}).get();
            gods.clear();
            System.out.println("Adds new gods");
            gods.add("Artemis");
            gods.add("Diana");
            gods.add("Diana");
            gods.add("Poseidon");
            gods.forEach(System.out::println);
            System.out.println("Removing an element");
            gods.remove("Diana");
            gods.forEach(System.out::println);
        }
    }
}
````

*Queue*
````java
public class Main5 {
    public static void main (String[] args) {
        try (SeContainer container = SeContainerInitializer.newInstance().initialize()) {
            Queue<String> gods = container.select(new TypeLiteral<Queue<String>>() {}).get();
            gods.clear();
            System.out.println("Adds new gods");
            gods.add("Artemis");
            gods.add("Diana");
            gods.add("Poseidon");
            gods.forEach(System.out::println);
            System.out.println("Pool element: " + gods.poll());
            System.out.println("Pool element: " + gods.poll());
            System.out.println("Pool element: " + gods.poll());
            System.out.println("Pool element: " + gods.poll());
            System.out.println("Pool element: " + gods.poll());
            gods.forEach(System.out::println);
        }
    }
}
````

*Map*
````java
public class Main6 {
    public static void main(String[] args) {
        try (SeContainer container = SeContainerInitializer.newInstance().initialize()) {
            Map<String, String> gods = container.select(new TypeLiteral<Map<String, String>>() {
            }).get();
            gods.clear();
            gods.put("hunt", "Artemis");
            gods.put("Sun", "Apollo");
            gods.put("Sea", "Poseidon");
            System.out.println("hunt: " + gods.get("hunt"));
            System.out.println("Sun: " + gods.get("Sun"));
            System.out.println("Sea: " + gods.get("Sea"));
            System.out.println("the keys: " + gods.keySet());
        }
    }
}
````

Além dessas estruturas similares, o Redis possui outras que não é possível demonstrar através dos padrões Java, que é o caso do **[SortedSet](https://redis.io/topics/data-types#sorted-sets)** e o **[Counter](https://redis.io/commands/incr)**, mas que o framework também abstrai e fornece classes para trabalhar com estas funcionalidades.

SortedSet
````java
public class Main7 {
    public static void main (String[] args) {
        try (SeContainer container = SeContainerInitializer.newInstance().initialize()) {
            SortedSet gods = container.select(SortedSet.class).get();
            gods.add("Artemis", 10);
            gods.add("Hera", 5);
            gods.add("Zeus", 2);
            gods.add("Apollo", 1);
            System.out.println("The ranking:");
            gods.getRanking().forEach(System.out::println);
            System.out.println("The reverse ranking:");
            gods.getRevRanking().forEach(System.out::println);
            System.out.println("The worse goods: " + gods.range(0, 1));
            System.out.println("The best ones: " + gods.revRange(0, 1));
        }
    }
}
````

Counter
````java
public class Main8 {
    public static void main (String[] args) {
        try (SeContainer container = SeContainerInitializer.newInstance().initialize()) {
            Counter gods = container.select(Counter.class).get();
            System.out.println("increment: " + gods.increment());
            System.out.println("increment: " + gods.increment(2));
            System.out.println("decrement: " + gods.decrement(2));
        }
    }
}
````

Como o post é focado mais na integração Java com Redis, para aqueles que queiram saber mais especificamente sobre **JNoSQL** ou **Redis**, pode acessar os Links Úteis abaixo, além também de ter o repositório no GitHub com todos os exemplos utilizados aqui.

# **Links Úteis**
- [Código de Exemplo](https://github.com/furlaneto/jnosql-redis-example)  
- [Site JNoSQL](http://www.jnosql.org/)  
- [Post JNoSQL](https://techiohub.com/eclipse-jnosql-uma-solucao-em-java-para-bancos-nao-relacionais/)  
- [Post Redis](https://techiohub.com/introducao-ao-redis-o-nosql-mais-famoso/)  