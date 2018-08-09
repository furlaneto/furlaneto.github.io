---
layout: post
title: Testes mutantes, testando nossos testes
date: '2018-04-19 22:40:39'
tags:
- java
- teste
- mutacao
---

Alguma vez você já parou para pensar se o seu teste realmente está cobrindo boa parte das possibilidades do seu código? Afinal, quem garante seus testes? Criando um outro teste para testar o teste?

Para isso, existem os testes mutantes, mas ai você está pensando, "Tipo X-Men?". É, pode ser.

Quando estamos desenvolvendo, criamos vários métodos onde possuem funções de *if*, *while*, *for*, soma, subtração, multiplicação e por ai em diante. Após desenvolver nosso código, ou antes, vamos lá e escrevemos nossos testes unitários para garantir que realmente cada cenário escrito, irá resultar na saída que colocamos.

Sendo assim, vamos criar uma classe simples que irá retornar *true* se o número for maior igual que 25 e *false* caso contrário.

````java
public class Condition {

    public boolean isOk (int num) {
        if (num >= 25) {
            return true;
        } else {
            return false;
        }
    }
}
````

Após isso, escrevemos dois testes. Um com número maior ou igual a 25 e outro para um número menor.

````java
public class ConditionTest {

    private Condition condition;

    @Before
    public void setUp() {
        this.condition = new Condition();
    }

    @Test
    public void shouldReturnTrueWhenNumIsGreaterOrEquals() {
        assertTrue(condition.isOk(200));
    }

    @Test
    public void shouldReturnFalseWhenNumIsLesser() {
        assertFalse(condition.isOk(10));
    }
}
````

Se rodar este teste, irá perceber que a cobertura de código da classe que está sendo testada será de 100%, mas estando 100% coberto o código, todos os mutantes morreram?

Afinal de contas, o que são esses mutantes? Os mutantes realizam algumas alterações no nosso código e verifica se existe alguma mutação que ocorra e o nosso teste não cubra. Por exemplo, podemos ter uma condição onde validamos se num >= 25, nesse caso, o mutante altera para num > 25, vendo se o nossos teste contemplam.

Em Java, existe a lib [PITest](http://pitest.org/) que é possível utilizar através de *plugins* para Eclipse e IntelliJ, além de integrar nas ferramentas de *build* [maven](http://pitest.org/quickstart/maven/) e [gradle](https://gradle-pitest-plugin.solidsoft.info/).

Após executar a lib, vamos ver o seguinte resultado

![mutation_failed](/content/images/2018/04/mutation_failed.png)

Podemos perceber que um mutante sobreviveu! Por que? Porque não validamos o **=** da condição. Assim, se adicionarmos um teste para validar as duas condições, os mutantes não conseguem sobreviver.

![mutarion_success](/content/images/2018/04/mutarion_success.png)

Uma observação importante é que por ele poder gerar diversos casos para seus testes, dependendo de quantas intruções e testes existem, irá demorar mais que os testes unitários em si.

Essa foi apenas uma introdução ao assunto, mas caso queira saber mais sobre o PITest e testes mutantes, é só acessar os links úteis.

# **Links Úteis**
[- Site PITest](http://pitest.org/)  
[- Conceitos Básicos PITest](http://pitest.org/quickstart/basic_concepts/)  
[- Tipos de mutações](http://pitest.org/quickstart/mutators/)  