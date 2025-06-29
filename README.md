
## **ELASTICSEARCH**

O Elasticsearch é um mecanismo de busca e análise distribuído desenvolvido com base no Apache Lucene. Ele foi projetado para busca de texto completo de alto desempenho, filtragem e análise de  dados em grandes volumes de dados estruturados e  não estruturados.

## **Principais Recursos do Elasticsearch**

• Busca e análises quase em tempo real.
• Arquitetura distribuída – escala horizontalmente.
• Busca de texto completo com pontuação de
relevância, stemming, sinônimos, etc.
• Consulta flexível usando JSON DSL.
• Armazena dados estruturados e não estruturados
(baseados em JSON).
• Suporta agregações (como SQL GROUP BY +
métricas).
• Alta disponibilidade via replicação.

## **GRAPHQL**

O GraphQL é uma linguagem de consulta para APIs e também um ambiente de execução para essas consultas. Ele foi desenvolvido pelo Facebook e permite que o cliente (frontend ou consumidor da API) peça exatamente os dados que precisa, em uma única requisição — nem mais, nem menos.

## **Principais Recursos do GraphQL**

• Pontos importantes do GraphQL
Um único endpoint (Em vez de múltiplas rotas
como no REST)
• Consultas personalizadas
• Esquema fortemente tipado ( modelo de dados
bem definido)
• Introspecção embutida (facilita a geração de
documentação)

## **União de GraphQL e Elasticsearch**

Unir GraphQL (linguagem de consulta para APIs) com Elasticsearch (motor de busca e análise de dados) oferece uma camada de acesso inteligente + motor
de busca veloz, formando uma arquitetura robusta para sistemas que exigem performance, precisão e flexibilidade. 

## **Exemplo de implementação - PHP**

Da mesma forma como ocorre com outras stacks /
linguagens, é necessário instalar as bibliotecas:


```shell
composer require elasticsearch/elasticsearch
composer require guzzlehttp/guzzle
```

## **Conectar ao ElasticSearch**

Use um hash da query GraphQL com variáveis para identificar os requests únicos.

```php
use Elasticsearch\ClientBuilder;

$client = ClientBuilder::create()->build();
```


## **Gerar uma Chave de Pesquisa**

Criar uma chave de pesquisa com um hash GraphQL e variáveis apra identificar os requests únicos.

```php
function generateCacheKey(string $query, array $variables = []): string {
    return md5($query . json_encode($variables));
}
```


## **Buscar com a API GraphQL** 

Seja em uma dupla de dados ou em um encapsulamento mais complexo, a busca de
informações pela API ocorre no seguinte fluxo

```php
use GuzzleHttp\Client;

$gqlClient = new Client();

$response = $gqlClient->post('https://your-graphql-api.com/graphql', [
    'json' => [
        'query' => $query,
        'variables' => $variables
    ]
]);
```


## **Aplicar Caching na resposta recebida**

Encapsular os dados recebidos no caching é a chave para a coexistência entre os 2 sistemas

```php
$cacheKey = generateCacheKey($query, $variables);

$response = $client->get([
    'index' => 'graphql_cache',
    'id'    => $cacheKey
]);

if (isset($response['_source']['data'])) {
    return $response['_source']['data']; // Return cached response
}
```


## **Opcional: Configurar a expiração / TTL**

O Elasticsearch não tem mais o TTL nativamente, logo, gerencie o Index do ciclo de vida, ou delete manualmente registros no cache antigo:

```php
    Use Index Lifecycle Management, or

    Manually delete old cache entries:

$client->delete([
    'index' => 'graphql_cache',
    'id'    => $oldCacheKey
]);
```


## **Tratar todo o conjunto em Função**

O uso de uma função auxiliar para organizar as diferentes partes do código e estabelecendo um gatilho para o acionamento deste contexto

```php
function graphqlWithCache($query, $variables = []) {
    $client = ClientBuilder::create()->build();
    $gqlClient = new Client();

    $cacheKey = generateCacheKey($query, $variables);

    // Check cache
    try {
        $cached = $client->get(['index' => 'graphql_cache', 'id' => $cacheKey]);
        return $cached['_source']['data'];
    } catch (\Exception $e) {
        // Cache miss or error
    }

    // Fetch GraphQL
    $response = $gqlClient->post('https://your-graphql-api.com/graphql', [
        'json' => ['query' => $query, 'variables' => $variables]
    ]);
    $data = json_decode($response->getBody(), true)['data'] ?? null;

    // Store in cache
    $client->index([
        'index' => 'graphql_cache',
        'id'    => $cacheKey,
        'body'  => ['data' => $data, 'cached_at' => date('c')]
    ]);

    return $data;
}
```

## **Exemplo de uso**

Como chamar a função da maneira correta passando os parâmetros necessários a fim de acionar o gatilho para o caching chamando o *graphqlWithCache*

```php
$query = <<<GQL
query GetUser(\$id: ID!) {
  user(id: \$id) {
    name
    email
  }
}
GQL;

$data = graphqlWithCache($query, ['id' => 123]);
```

Este foi apenas um artigo breve, direto ao ponto, sobre como orquestrar as consultas do graphQL com o caching do ElastichSearch


Mais detalhes, valide a versão em [PDF](https://github.com/educooper/graphqlcomelastichsearch/blob/main/ebook.pdf)
