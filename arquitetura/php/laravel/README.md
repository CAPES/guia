# Arquitetura PHP

## Introdução

Este documento tem por objetivo exemplificar os conceitos para implementação dos padrões arquiteturais da CAPES em PHP, servindo como base para as equipes de desenvolvimento.

## Stack de tecnologias

As seguintes ferramentas, frameworks e bibliotecas formam a stack da arquitetura PHP:

- [Lumen](https://lumen.laravel.com/) (v8.2): micro-framework baseado no [Laravel](https://laravel.com/) com foco em construção de APIs performáticas sem abrir mão das facilidades de desenvolvimento que o Laravel oferece.
- [Doctrine ORM](https://www.doctrine-project.org/projects/orm.html) (v2.8): camada de abstração para o banco de dados. É preferível em favor do [Eloquent](https://laravel.com/docs/8.x/eloquent) ([ORM](https://pt.wikipedia.org/wiki/Mapeamento_objeto-relacional) padrão do Laravel, utiliza o padrão de _design_ [Active Record](https://www.martinfowler.com/eaaCatalog/activeRecord.html)) por utilizar o padrão de design [Data Mapper](https://martinfowler.com/eaaCatalog/dataMapper.html), possibilitando uma separação entre a lógica de negócio/domínio da lógica de persistência de dados, além de ser mais performático ao lidar com uma quantidade maior de dados.
- [Laravel Doctrine](http://www.laraveldoctrine.org/) (v2.0): integração entre o Doctrine ORM e o Laravel/Lumen.
- [Elasticsearch](https://www.elastic.co/) (v7.10): utilizado como banco de dados NoSQL para implementação do padrão [CQRS](https://martinfowler.com/bliki/CQRS.html).
- [RabbitMQ](https://www.rabbitmq.com/) (v3.8): _message-broker_ de código aberto para uso de filas na aplicação.

## Instalação

### Ambiente
Para rodar a aplicação é necessário ter o *Docker* e *Docker Compose* instalados. Para instalar, basta seguir as instruções de instalação para o [Docker](https://docs.docker.com/get-docker/)  e [Docker Compose](https://docs.docker.com/compose/install/).

Após a instalação, execute os seguintes comandos para configurar o ambiente da aplicação:

> *Obs.: Caso o usuário do sistema operacional tenha um ID diferente de
> **1000**, altere a propriedade **UID** no arquivo **.env** para o valor correto. Isso é necessário para configurar corretamente as permissões dos arquivos da aplicação no sistema de arquivos.*

```sh
git clone https://git.capes.gov.br/cgs/narq/pocs/poc-microsservico/php/task-list.git
cd task-list
cp .env.example .env
docker-compose up -d
```

Após o download e construção das imagens o ambiente estará configurado.

### Banco de Dados
Para popular as tabelas e dados no banco de dados da aplicação, basta executar o seguinte comando:

```sh
docker exec task-list_system php artisan migrate --seed
```

## Padrões Arquiteturais e Boas Práticas

### Infraestrutura

As aplicações devem ser construídas utilizando a abordagem arquitetural de microsserviços. Essa abordagem proporciona diversos benefícios, como escalabilidade, resiliência, implantações e atualizações mais ágeis, facilidade de desenvolvimento e manutenção, entre outros.

Conteinerização através do *Docker* permite a execução da aplicação da maneira fidedigna independente de sistema operacional e hardware. Por fim, orquestração através do *Docker Compose* (em ambiente local) e *Openshift* (em ambientes DHT e produção) facilita o gerenciamento dos serviços.

### Controllers

A camada de controle é a camada que lida com lógica referente a uma determinada requisição. Recebe uma requisição e deve devolver uma resposta adequada ao cliente. **NÃO DEVE** conter regras de negócio, para isso utilizamos os [Services](#services).

#### Single Action Controllers

É aconselhável a utilização de [Single Action Controllers](https://laravel.com/docs/8.x/controllers#single-action-controllers). Esta abordagem traz alguns benefícios:

- Adequação ao [Princípio da Responsabilidade Única](https://en.wikipedia.org/wiki/Single-responsibility_principle);
- Estimula a [composição sobre herança](https://en.wikipedia.org/wiki/Composition_over_inheritance);
- Diminui o acoplamento ao diminuir a quantidade de dependências no construtor;

#### Requests

A validação e autorização de requisições deve ser feita utilizando [Form Requests](https://laravel.com/docs/8.x/validation#form-request-validation) em conjunto com [Policies](https://laravel.com/docs/authorization#creating-policies). Isso possibilita a separação da lógica de validação e autorização dos *Controllers* e facilita a reutilização de regras, especialmente as mais complexas.

Como o *Lumen* não implementa *Form Requests* nativamente, os *Form Requests* da aplicação devem extender a classe FormRequest do componente [arquitetura-capes/infra-controller](TODO:ADICIONAR_LINK).

#### Responses

Para a serialização de respostas é utilizado como base [API Resources](https://laravel.com/docs/8.x/eloquent-resources).

Apesar de, no Laravel, a utilização ser em conjunto com o *Eloquent* (que não é utilizado nesta arquitetura), é possível a sua utilização com qualquer objeto.

Uma única adição foi feita em relação aos *Resources* padrões do *Laravel*: a possibilidade de especificação de quais campos da resources devem ser incluídos/excluídos da serialização.

Em uma aplicação de médio/grande porte, há um momento em que apenas uma parte de uma Entidade precisa ser serializada para o cliente em uma determinada requisição. Para que isso seja possível, é necessário extender a classe base `Capes\Http\Resources\AbstractResource` e declarar o método `resourceFields()`, que deve retornar um array associativo com todos os campos serializáveis do _Resource_, onde as chaves são os nomes dos campos e os valores são *callbacks* que retornam o seu valor:

```php  
<?php  
  
namespace App\Http\Resources;  
  
use App\Entities\Colecao;  
use Capes\Http\Resources\AbstractResource;  
  
class ColecaoResource extends AbstractResource  
{    
    /** @var Colecao */    
    public $resource;  
  
    public function __construct(Colecao $resource)    
    {  
        parent::__construct($resource);  
    }  
  
    public function resourceFields(): array    
    {  
        $c = $this->resource;    
        return [  
           'id'        => fn() => $c->getId(),  
           'descricao' => fn() => $c->getDescricao()  
       ];  
    }  
}  
```  

Posteriormente, no controller, indicamos quais campos devem ser incluídos/excluídos da serialização através dos métodos `Resource::including()` e `Resource::excluding()`:

```php  
<?php  
  
namespace App\Http\Controllers\Colecoes;  
  
use App\Http\Controllers\Controller;  
use App\Http\Resources\ColecaoResource;  
use Capes\Http\Resources\ResourceCollection;  
use App\Repositories\Colecoes\ColecoesRepositoryInterface;  
use App\Services\ColecoesService;  
  
class IndexController extends Controller  
{  
    public ColecoesService $service;    
  
    public function __construct(ColecoesService $service)    
    {  
        $this->service = $service;  
    }  
  
    public function __invoke(ColecoesRepositoryInterface $repository): ResourceCollection    
    {  
        $colecoes = $repository->findAll();  
  
        $resource = ColecaoResource::collection($colecoes)->including([  
            ColecaoResource::class => ['id']  
        ]);  
  
        return $resource;  
    }  
}  
```

A resposta retornada seria:

```json
{
  "data": [
    {
      "id": 1
    }
  ]
}
```

### Services

*Services* são objetos que possuem uma responsabilidade bem definida. Neles são definidas regras de negócio da aplicação e comunicação com serviços externos.

### Repositories

_Repositories_ são classes responsáveis por concentrar lógicas referentes a persistência de dados. Todo repositório deve ter uma interface correspondente, que será utilizada na tipagem para injeção de dependência. Isso facilita a implementação de testes unitários.

### Elasticsearch

Para instruções sobre a configuração e utilização do Elasticsearch, consultar a [documentação oficial](https://github.com/cviebrock/laravel-elasticsearch) da bibilioteca laravel-elasticsearch.