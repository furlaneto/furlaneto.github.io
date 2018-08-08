---
layout: post
title: Melhorando a performance de requisições HTTP no RestTemplate utilizando um
  pool de conexões
date: '2018-01-31 20:49:11'
tags:
- java
- spring
- http
- pool
---

Assim como nos bancos de dados, o processo de abrir uma conexão pode ser muito custoso, pois envolver resolução de DNS, abertura de *socket*, *handshake*, caso seja uma conexão HTTPS, etc. Deste forma, se manter uma quantidade de conexões abertas e reutilizá-las, sem ter que passar por todo esse processo de abertura de conexão, resultará em um ganho de performance da sua aplicação.

Esse processo de manter algumas conexões abertas é chamado de *pool*, e assim como nos bancos de dados, todos fornecem tal funcionalidade e não é diferente para as linguagens de programação, mas neste exemplo será abordado especificamente o RestTemplate do Spring.

Antes de declarar os @Bean de configuração, é bom saber qual a média de consultas serão realizadas, pois assim, é possível configurar com maior precisão o *pool*.

Para começar, é necessário declarar os @Bean relacionados a configurações dos *requests*, HttpClient, RestTemplate e o *connection manager* do *pool*.

````java
@Bean
public PoolingHttpClientConnectionManager poolingHttpClientConnectionManager() {
    PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
    connectionManager.setMaxTotal(1000);
    connectionManager.setDefaultMaxPerRoute(100);
    return connectionManager;
}

@Bean
public RequestConfig requestConfig() {
    RequestConfig requestConfig = RequestConfig.custom()
        .setConnectionRequestTimeout(1000)
        .setConnectTimeout(1000)
        .setSocketTimeout(1000)
        .build();
    return requestConfig;
}

@Bean
public CloseableHttpClient httpClient(PoolingHttpClientConnectionManager poolingHttpClientConnectionManager, RequestConfig requestConfig) {
    CloseableHttpClient httpClient = HttpClientBuilder
        .create()
        .setConnectionManager(poolingHttpClientConnectionManager)
        .setDefaultRequestConfig(requestConfig)
        .build();
    return httpClient;
}

@Bean
public RestTemplate restTemplate(HttpClient httpClient) {
    HttpComponentsClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory();
    requestFactory.setHttpClient(httpClient);
    return new RestTemplate(requestFactory);
}
````

Na criação do *connection manager* do *pool*, é possível ver dois atributos, o **maxTotal** e **defaultMaxPerRoute**. O primeiro trata da quantidade total de conexões que poderão ser abertas no *pool* e o segundo a quantidade máxima de conexões que poderão ser abertas por rota, que nada mais é os endereços que serão consultados, formados por **protocolo://host:porta**. Neste exemplo foi utilizado 100 conexões abertas por cada roda e no máximo 1000 conexoes abertas no total, sendo assim, podemos ter até 10 rotas com 100 conexões abertas cada que não iria ultrapassar o limite ou também mais rotas, só que cada um com menos conexões abertas, por isso é importante conhecer o consumo antes de realizar a configuração.

Além de configurar um *default* para a quantidade máxima de conexões por rota, também é possível configurar a quantidade máxima para uma rota em específico. Fazendo da seguinte forma:

````java
@Bean
public PoolingHttpClientConnectionManager poolingHttpClientConnectionManager() {
    PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
    connectionManager.setMaxTotal(1000);
    connectionManager.setDefaultMaxPerRoute(100);

    final HttpHost httpHost = new HttpHost("localhost", 8085, "http");
    connectionManager.setMaxPerRoute(new HttpRoute(httpHost), 200);

    return connectionManager;
}
````
Assim, terá um *default* de 100 conexões por rota e especificamente para a rota http://localhost:8085 terá 200.

Esta é uma configuração básica de utilização de um *pool* de conexões HTTP e para aqueles que queiram saber mais do assunto em questão, pode acessar os links no fim do post.

**Links Úteis**
- [Connection management](https://hc.apache.org/httpcomponents-client-4.2.x/tutorial/html/connmgmt.html)  
- [RestTemplate](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)  