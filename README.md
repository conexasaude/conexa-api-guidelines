# conexa-api-guidelines

## Índices
1. [Design voltado a recursos](#design-voltado-a-recursos)
2. [Subrecurso](#subrecurso)
3. [Nomenclatura](#nomenclatura)
4. [Operações sobre recursos](#operações-sobre-recursos)
5. [Códigos de retorno](#códigos-de-retorno)
6. [Regras para consumidores](#regras-para-consumidores)
7. [Headers de requisição padrões](#headers-de-requisição-padrões)
8. [Headers customizados padrões](#headers-customizados-padrões)
9. [Operações sobre recurso em mais detalhes](#operações-sobre-recurso-em-mais-detalhes)
10. [Busca por um recurso usando o Id do recurso](#busca-por-um-recurso-usando-o-Id-do-recurso)
11. [Busca por uma coleção](#busca-por-uma-coleção)
12. [Deleção de um recurso](#deleção-de-um-recurso)
13. [Atualização Total de um recurso](#atualização-Total-de-um-recurso)
14. [Atualização Parcial de uma recurso](#atualização-Parcial-de-uma-recurso)
15. [Criação de um recurso](#criação-de-um-recurso)
16. [Paginação](#paginação)
17. [Ordenação](#ordenação)
18. [Controlar quantidade de campos retornados na resposta](#controlar-quantidade-de-campos-retornados-na-resposta)
19. [Busca por range](#busca-por-range)


## Objetivo deste guia

Este é um guia geral para auxiliar desenvolvedores a criar APIs simples, consistentes e fáceis de usar.

Clientes tanto internos quanto externos conseguem integrar com as APIs muito mais fácil pois conseguem identificar a consistência nas operações.

Desenvolvedores conseguem desenvolver e testar as APIs mais rapidamente.

Todas as APIs tanto internas quanto externas devem seguir este guia para obtermos a desejada consistência entre as APIs

## Design voltado a recursos

As APIs devem ser voltadas para recursos. Um recurso pode ser entendido com uma entidade dentro do domínio ou uma capacidade do seu negócio como “Comunicação”, “Autenticação”, “Auditoria”, etc etc.

## Subrecurso

São recursos que só podem ser acessados a partir do recurso pai. Exemplo: Endereço de um paciente. Não é obrigatório que um recurso tenha um subrecurso ou mesmo que forneça acesso direto ao subrecurso. Isso talvez faça sentido se algum consumidor desejar acessar o subrecurso diretamente.

A estutura da url para acessar um subrecurso é `https://patient-v1.conexasaude.com.br/patient/123/address`

## Nomenclatura 

Os nomes dos recursos devem ser em inglês americano. Devem ser minúsculas e no singular. Nomes compostos devem ser separados por “_”. O recursos geralmente são nomeados usando um substantivo

As urls devem conter a versão do recurso
As urls devem conter o nome do recurso no singular. 
Urls não devem conter espaço. 
Urls não devem conter acentos, etc. 
Urls não devem conter verbos, exceto para descrever operações que não consigam ser descritas usando os verbos HTTP

Exemplo recurso patient: `https://patient-v1.conexasaude.com.br/patient`

Os nomes dos campos devem ser em inglês americano. Devem ser minúsculas e no singular. Nomes compostos devem ser separados por “_”. Não devem conter espaços e nem acentos.

Campos do tipo lista devem estar no plural. Não usar o tipo do campo no nome. Exemplo: não usar patientList, melhor usar patients

Evitar criar objetos com um campo só. Verifique a possibilidade de trabalhar com um campo simples.

Campos do tipo data devem seguir o padrão **dd/MM/yyyy HH:mm:ss**

Evitar usar os artigos em inglês “a”, “the”, “of”. Exemplo: não usar “countOfPatient”, usar “patientCount”

## Operações sobre recursos

As operações sobre os recursos são representados por verbos. Usando o padrão REST, esses verbos são os próprios verbos definidos pelo HTTP.

Os verbos HTTP também são conhecidos como métodos HTTP

Operações devem usar o método correto sempre que possível e a idempotência deve ser respeitada

Nem todo recurso vai implementar todos os métodos, mas todos os recursos devem se ater ao caso de uso de cada método.

| Método | Descrição                                                                                       | Idempotente |
|--------|-------------------------------------------------------------------------------------------------|-------------|
| GET    | Busca por um recurso ou uma coleção                                                             | Sim         |
| PUT    | Atualização total de um recurso                                                                 | Sim         |
| DELETE | Deleta um recurso                                                                               | Sim         |
| POST   | Cria um novo recurso ou envia um comando                                                        | Não         |
| HEAD   | Retorna metadados de uma requisição GET.  Recursos que suportam GET, podem suportar HEAD também | Sim         |
| PATCH  | Atualização parcial de um recurso                                                               | Não         |

Nem toda operação pode ser descrita usando os verbos HTTP. Nesses casos, podemos usar verbos na url que indiquem a ação que vai ser executada sobre o recurso. Geralmente essas ações são feitas usando um POST Exemplo:

`/payment/{id}/capture`

Algumas pessoas consideram que TODAS as operações podem ser descritas com os verbos HTTP. Essas operações como a do exemplo acima, no fim das contas atualizam determinados campos do recurso e poderia ser feito através de um PATCH

Uma API deve conter no máximo 15 operações. Mais que isso, provavelmente sua API está com muitas responsabilidades e deve ser quebrada em duas APIs

## Códigos de retorno

APIs devem sempre retornar o status code HTTP correto em cada situação. Alguns status codes e quando devem ser usados:

| Status Code                 | Descrição                                                                                                                                                                                                                                                                                                                                                                                          |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 200 (OK)                    | Esta requisição foi bem sucedida. O significado do sucesso varia de acordo com o método HTTP                                                                                                                                                                                                                                                                                                       |
| 201 (Created)               | A requisição foi bem sucedida e um novo recurso foi criado como resultado. Esta é uma tipica resposta enviada após uma requisição POST.                                                                                                                                                                                                                                                            |
| 202 (Accepted)              | A requisição foi recebida mas nenhuma ação foi tomada sobre ela ainda. Geralmente indica que um processamento assíncrono vai ser executado e o resultado do processamento será enviado de outra forma                                                                                                                                                                                              |
| 204 (No Content)            | Não há conteúdo na resposta desta requisição. Geralmente usado em requisições DELETE                                                                                                                                                                                                                                                                                                               |
| 301 (Moved Permanently)     | Esse código de resposta significa que a URI do recurso requerido mudou. Provavelmente, a nova URI será especificada na resposta.                                                                                                                                                                                                                                                                   |
| 302 (Found)                 | Esse código de resposta significa que a URI do recurso requerido foi mudada temporariamente.                                                                                                                                                                                                                                                                                                       |
| 400 (Bad Request)           | Essa resposta significa que o servidor não entendeu a requisição pois está com uma sintaxe inválida. Ex: Faltando um campo obrigatório                                                                                                                                                                                                                                                             |
| 401 (Unauthorized)          | Embora o padrão HTTP especifique "unauthorized", semanticamente, essa resposta significa "unauthenticated". Ou seja, o cliente deve se autenticar para obter a resposta solicitada.                                                                                                                                                                                                                |
| 403 (Forbidden)             | O cliente não tem direitos de acesso ao conteúdo portanto o servidor está rejeitando dar a resposta. Diferente do código 401, aqui a identidade do cliente é conhecida.                                                                                                                                                                                                                            |
| 404 (Not Found)             | O servidor não pode encontrar o recurso solicitado.                                                                                                                                                                                                                                                                                                                                                |
| 405 (Method not Allowed)    | O método de solicitação é conhecido pelo servidor, mas foi desativado e não pode ser usado. Os dois métodos obrigatórios, GET e HEAD, nunca devem ser desabilitados e não devem retornar este código de erro.                                                                                                                                                                                      |
| 422 (Unprocessable Entity)  | A requisição está bem formada mas inabilitada para ser seguida devido a erros semânticos. Exemplo: Tentativa de criar um agendamento para um médico inativado.                                                                                                                                                                                                                                     |
| 500 (Internal Server Error) | O servidor encontrou uma situação com a qual não sabe lidar.                                                                                                                                                                                                                                                                                                                                       |
| 503 (Service Unavailable)   | O servidor não está pronto para manipular a requisição. Causas comuns são um servidor em manutenção ou sobrecarregado. Note que junto a esta resposta, uma página amigável explicando o problema deveria ser enviada. Estas respostas devem ser usadas para condições temporárias e o cabeçalho HTTP **Retry-After**: deverá, se posível, conter o tempo estimado para recuperação do serviço. |
| 504 (Gateway Timeout)       | Esta resposta de erro é dada quando o servidor está atuando como um gateway e não obtém uma resposta a tempo. Quando utilizamos o Nginx ou Apache na frente das aplicações também é possível receber essa resposta.                                                                                                                                                                                |

Campos faltando em uma requisição POST devem retornar erro 400 e não 422 pois a requisição está sintaticamente incorreta. Os campos obrigatórios devem ser indicados na documentação da API.

Erros devido a regras de negócio NÃO devem retornar erro 500 e sim 422.

## Regras para consumidores 

Consumidores devem ignorar campos não conhecidos
Consumidores não devem esperar uma ordem específica dos campos na resposta

## Headers de requisição padrões

Consumidores devem incluir os header padrões **Content-Type**, **Accept-Charset** e **Accept** em todas as requisições.

## Headers customizados padrões

Os headers customizados devem começar com o prefix “x-”

Header x-applicationName

Esse header identifica quem é o consumidor. Esse nome deve ser único entre os consumidores

Header x-applicationKey

Esse header possui a chave de acesso de um consumidor para chamar outras APIs. Deve ser único entre os consumidores. Através desses headers de identificação podemos restringir o acesso de determinados clientes.

Header x-tid

Esse header é utilizado para fazer o tracking das transações entre os diferentes microsserviços. Todos os consumidores ao realizar requisições para outros microsserviços DEVEM incluir esse header. O valor desse header deve corresponder ao applicationName-UUID. Exemplo: patient-7457ce0d-2f09-44b2-9877-3d33d3175e5c

As APIs ao processarem a requisição devem verificar a existência do header x-tid. Se não existir, a API deve criar um novo valor. Exemplo: se a API patient chamar a API clinic sem o header x-tid, a API clinic deve criar um novo valor para o TID com o valor clinic-7457ce0d-2f09-44b2-9877-3d33d3175e5c

Todos as entradas de log do processamento da requisição devem exibir o valor do x-tid

## Operações sobre recurso em mais detalhes

Para os exemplos abaixo vamos usar uma API fictícia chamada patient. A v1 da API de patient roda em `https://patient-v1.conexasaude.com.br`. Esse é um exemplo de response da API para o recurso 123.

```json
{
   “id”: 123,
   “name”: “Diego Sena”,
  “age”: 12,
  “cpf”: “12345678910”,
  “email”: “diego@teste.com”,
  “cellphone”: “21123456789”,
  “address”: {
    “street”: “Rua xyz”,
    “number”: “12323”,
    “city”: “RIo de Janeiro”,
  }
}

```

## Busca por um recurso usando o Id do recurso

A busca pelo recurso usando o id deve ser feitos usando um GET na url `https://{nome do recurso}-v1.conexasaude.com.br/{nome do recurso}/{id do recurso}`.

No exemplo acima, `https://patient-v1.conexasaude.com.br/patient/123`.

O retorno deverá ser a entidade solicitada com status code 200

## Busca por uma coleção

A busca pela coleção de um determinado recurso deve ser feita através de um GET na url  https://{nome do recurso}-v1.conexasaude.com.br/{nome do recurso}. 

No exemplo acima, https://patient-v1.conexasaude.com.br/patient

Essa url deve retornar uma lista com todos os recursos encontrados dados os critérios da busca. Esse é um exemplo de retorno.

```json
{
    “data”: [
        {
            “id”: 123,
            “name”: “Diego Sena”,
            “age”: 12,
            “cpf”: “12345678910”,
            “email”: “diego@teste.com”,
            “cellphone”: “21123456789”,
            “address”: {
                “street”: “Rua xyz”,
                “number”: “12323”,
                “city”: “RIo de Janeiro”,
            }
        },
        {
            “id”: 456,
            “name”: “Alice Sena”,
            “age”: 21,
            “cpf”: “09876543212”,
            “email”: “alice@teste.com”,
            “cellphone”: “21123456789”,
            “address”: {
                “street”: “Rua xyz”,
                “number”: “12323”,
                “city”: “RIo de Janeiro”,
            }
        }
    ],
    “_meta”: {
        "pagination": {
            "limit": 10,
            "offset": 0,
            "count" 2
        }
    }
}

```

## Deleção de um recurso

A deleção de um recurso deve ser feita através de um DELETE na url `https://{nome do recurso}-v1.conexasaude.com.br/{nome do recurso}/{id do recurso}`

No exemplo acima, `https://patient-v1.conexasaude.com.br/patient/123`.

Atualização Total de um recurso

A atualização total de um recurso deve ser feita através de um PUT na url `https://{nome do recurso}-v1.conexasaude.com.br/{nome do recurso}`. Na documentação deve-se indicar que o que for passado no corpo da requisição será o novo estado do recurso.

No exemplo acima, `https://patient-v1.conexasaude.com.br/patient`

```json
{
    “id”: 123,
    “name”: “Diego Sena”,
    “age”: 12,
    “cpf”: “12345678910”,
    “email”: “diego@teste.com”,
    “cellphone”: “21123456789”,
    “address”: {
        “street”: “Rua xyz”,
        “number”: “12323”,
        “city”: “RIo de Janeiro”,
    }
}
```
## Atualização Parcial de uma recurso

A atualização parcial de um recurso deve ser feita através de um PATCH na url `https://{nome do recurso}-v1.conexasaude.com.br/{nome do recurso}`. Essa operação deve ser usada quando se deseja atualizar apenas alguns campos do recurso. Na documentação deve-se indicar que APENAS o que for enviado na requisição vai ser atualizado. Os outros campos permanecerão inalterados.

No exemplo acima, `https://patient-v1.conexasaude.com.br/patient`

```json
{
    “id”: 123,
    “name”: “Diego Sena”,
    “cpf”: “12345678910”,
    “email”: “diego@teste.com”,
    “cellphone”: “21123456789”
}

```

Muitos preferem não utilizar o PATCH para atualizações parciais. Alguns desenvolvedores preferem um POST em um subrecurso.

## Criação de um recurso

A criação de um recurso deve ser feita através de um POST na url `https://{nome do recurso}-v1.conexasaude.com.br/{nome do recurso}`

No exemplo acima, `https://patient-v1.coonexasaude.com.br/patient`

O corpo da requisição deve conter os campos necessários para a criação do recurso. Se algum campo obrigatório estiver faltando, a API deve retornar erro 400.

```json
{
    “id”: 4452,
    “name”: “Denize Sena”,
    “age”: 12,
    “cpf”: “12345678910”,
    “email”: “diego@teste.com”,
    “cellphone”: “21123456789”,
    “address”: {
        “street”: “Rua xyz”,
        “number”: “12323”,
        “city”: “RIo de Janeiro”,
    }
}

```

Se o recurso foi criado, deve-se retornar status code 201 com a url para o recurso recém criado

```json
{
    _link: {
        “id”: 2344,
        “href”: “https://patient-v1.conexasaude.com.br/patient/2344”
    }
}

```

## Paginação 

A paginação deve ser feita na busca por coleção. Os parâmetros page e size são usados para controlar a paginação. TODAS AS APIS DEVEM LIMITAR O TAMANHO MÁXIMO DA PÁGINA PARA EVITAR TRAZER UMA QUANTIDADE MUITO GRANDE DE DADOS.

Ex: `https://patient-v1.conexasaude.com.br/patient?page=1&size=100`

## Ordenação

A ordernação deve ser feita na busca por coleção usando o parâmetro sort. É possível ordenar por mais de um campo. O formato do parâmetro de ordenação é o seguinte: \<campo\>,\<direção\>. Exemplo: name,desc.
Ex: `https://patient-v1.conexasaude.com.br/patient?sort=name,desc`

## Controlar quantidade de campos retornados na resposta

Às vezes o retorno de uma API é grande e um consumidor está interessado em uma versão resumida do retorno. Neste caso, em hipótese alguma devemos criar um endpoint que retorna uma versão resumida do recurso. Para isso existe o parâmetro fields. Ele é uma lista de todos os campos que o consumidor quer na resposta. O identificador do recurso SEMPRE deve retornar mesmo que o consumidor da API não o inclua no parâmetro fields.

Exemplo: 

Retorno normal sem o parâmetro fields

`https://patient-v1.conexasaude.com.br/patient`

```json
{
    “id”: 4452,
    “name”: “Denize Sena”,
    “age”: 12,
    “cpf”: “12345678910”,
    “email”: “diego@teste.com”,
    “cellphone”: “21123456789”,
    “address”: {
        “street”: “Rua xyz”,
        “number”: “12323”,
        “city”: “RIo de Janeiro”,
    }
}

```

Retorno exibindo só o id, cpf e nome

`https://patient-v1.conexasaude.com.br/patient?fields=name,cpf`
```json
{
   “id”: 4452,
   “name”: “Denize Sena”,
  “cpf”: “12345678910”
}
```

Para desenvolvedores backend, deve-se evitar criar uma entidade para cada possível retorno. Exemplo: AtendimentoResumidoBackoffice, AtendimentoResumido, AtendimentoCompleto, etc etc

## Busca por range

Não há um padrão consolidado nesse caso porém em muitas empresas usa-se o prefixo “from” e “to” antes dos nomes dos campos.

Exemplo: Buscar todos os pacientes entre 10 e 20 anos.

`https://patient-v1.conexasaude.com.br/patient?fromAge=10&toAge=20`
