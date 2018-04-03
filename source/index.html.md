---
title: API Zapay

language_tabs:
  - shell
---

# Introdução

Bem-vindo à API da Zapay Pagamentos. Através de nossa API, são disponibilizados endpoints para que seu sistema se comunique diretamente com o usuário do aplicativo.

# Autenticação

A Zapay Pagamentos utiliza o sistema de autenticação OAuth2 para validar suas requisições. Isto quer dizer que é esperada uma header contendo um Token de acesso em todas as chamadas feitas à API Zapay. O formato da header é o seguinte:

`Authorization: Bearer <TOKEN>`

<aside class="notice">
Substitua <code>TOKEN</code> pelo seu próprio Token de acesso.
</aside>

Para adquirir seu Token de acesso para o ambiente de produção, entre em contato conosco atráves do e-mail `contato@usezapay.com.br`.

## Homologação

O ambiente de homologação pode ser usado para validar sua integração com a API Zapay.

O acesso é feito através do endpoint: `hmg.api.usezapay.com.br`.

Para a autenticação, deve-se utilizar o Token de acesso abaixo:

<aside class="notice">
G2XHRBvcVzmUcLKkQEyzubmwybFyTh
</aside>

# Unidades

Unidades são os estabelecimentos que fazem vendas: é necessário possuir uma unidade cadastrada para fazer uma associação com os produtos vendidos.

```shell
    curl -X POST \
    -H "Authorization: Bearer <Token>" \
    -H "Content-Type: application/json" \
    -d '{
        "name":"Stark Industries LTDA",
        "fantasy_name":"Stark Industries",
        "cnpj":"00000000000000",
        "contact_info": {
            "phone": "00000000000",
            "email": "contato@starkindustries.com"
        }
    }' \
http://hmg.api.usezapay.com.br/unit/register/
```

> Exemplo de resposta:

```json
    {"status": "success"}
```

Este endpoint registra uma unidade.

### Requisição HTTP

`POST http://hmg.api.usezapay.com.br/unit/register/`

### Parâmetros

Parâmetro | Tipo | Obrigatório | Descrição
--------- | ---- | ----------- | ------ | -----------
name | String | Sim | A Razão Social da unidade.
fantasy_name | String | Sim | Nome Fantasia da unidade.
cnpj | String | Sim | O CNPJ da unidade. Deve conter apenas números e ter 14 dígitos.
 contact_info[phone] | String | Sim | O telefone de contato da unidade. Deve ter apenas números e 10 ou 11 dígitos (sendo os dois primeiros o DDD).
contact_info[email] | String | Sim | O e-mail de contato da unidade. Deve ter um formato de e-mail válido.

# Merchandise

Merchandise são os produtos ou serviços a serem comprados ou adquiridos pelos usuários finais.

```shell
  curl -X POST \
    -H "Authorization: Bearer <Token>" \
    -H "Content-type: application/json" \
    -d '{
        "merchandise": [
            {
                "name": "PROPULSOR DE ÍONS",
                "code": "192301",
                "value": 20000
            }, {
                "name": "REATOR ARC",
                "code": "192302",
                "value": 50000
            }
        ],
        "user_identifier": {
            "value": "00000000000",
            "type": "cpf"
        },
        "unit_identifier": {
            "type": "cnpj",
            "value": "00000000000000"
        }
    }' \
    http://hmg.api.usezapay.com.br/merchandise/send/
```

> Exemplo de resposta:

```json
    {"status": "success"}
```

Este endpoint envia produtos ou serviços a serem pagos pelo usuário final.

<aside class="notice">
O valor total da compra é calculado automaticamente com base nos valores dos produtos/serviçoes e suas quantidades.
</aside>

### Requisição HTTP

`POST http://hmg.api.usezapay.com.br/merchandise/send/`

### Parâmetros

Parâmetro | Tipo | Obrigatório | Descrição
--------- | ---- | ----------- | ------ | -----------
merchandise | Array | Sim | A lista de objetos do tipo Merchandise (veja abaixo). Deve conter ao menos um objeto.
user_identifier | Objeto | Sim | Objeto contendo informações sobre o usuário final.
user_identifier[type] | String | Sim | O tipo de parâmetro identificador do usuário. Atualmente, somente cpf é aceito.
user_identifier[value] | String | Sim | O valor do parâmetro identificador do usuário. Um CPF, por exemplo.
unit_identifier | Objeto | Sim | Objeto contendo informações sobre a unidade responsável pela venda.
unit_identifier[type] | String | Sim | O tipo de parâmetro identificador da unidade. Atualmente, somente cnpj é aceito.
unit_identifier[value] | String | Sim | O valor do parâmetro identificador da unidade. Um CNPJ, por exemplo.


## Objeto Merchandise

Trata-se de uma representação do produto ou serviço a ser adquirido. 

### Parâmetros

Parâmetro | Tipo | Obrigatório | Descrição
--------- | ---- | ----------- | ------ | -----------
name | String | Sim | O nome do produto ou serviço
value | Inteiro | Sim | O valor do produto ou serviço. Deve ser um inteiro e deve ser enviado em centavos (por exemplo, R$ 12,50 = 1250)
code | String | Sim | Um código único identificador do produto. Deve ser o código de barras, quando existir.
amount | Inteiro | Não | A quantidade dete tipo de produto. Se não for informada, assume 1.

# Confirmação

É possível cadastrar um endpoint para o qual serão enviadas confirmações de compras em sua unidade.
O endpoint deve ser capaz de receber uma requisição POST da API da Zapay.

## Cadastrar endpoint

O endpoint cadastrado pode ser sobreescrito a qualquer momento.

```shell
    curl -X GET \
    -H "Authorization: Bearer <Token>" \
    -H "Content-type: application/json" \
    "http://hmg.api.usezapay.com.br/unit/confirmation-endpoint/?type=cnpj&value=00000000000000"
```

> Exemplo de resposta:

```json
    {"status": "success"}
```

### Requisição HTTP

`GET http://hmg.api.usezapay.com.br/unit/confirmation-endpoint/`

### Parâmetros

Os parâmetros deste método devem ser enviados na URL.

Parâmetro | Tipo | Obrigatório | Descrição
--------- | ---- | ----------- | ------ | -----------
type  | String | Sim | O tipo de parâmetro identificador da unidade. Atualmente, somente cnpj é aceito.
value | String | Sim | O valor do parâmetro identificador da unidade. Um CNPJ, por exemplo.

## Consultar endpoint

Em caso de dúvidas, o endpoint de confirmação cadastrado para a unidade pode ser consultado.

```shell
    curl -X POST \
    -H "Authorization: Bearer <Token>" \
    -H "Content-type: application/json" \
    -d '{
        "endpoint": "https://your.magical.api.com/confirmation-receiver/",
        "unit_identifier": {
            "type": "cnpj",
            "value": "00000000000000"
        }
    }' \
    http://hmg.api.usezapay.com.br/unit/confirmation-endpoint/
```

> Exemplo de resposta:

```json
  {"endpoint": "https://your.magical.api.com/confirmation-receiver/"}
```

### Requisição HTTP

`GET http://hmg.api.usezapay.com.br/unit/confirmation-endpoint/`

### Parâmetros

Parâmetro | Tipo | Obrigatório | Descrição
--------- | ---- | ----------- | ------ | -----------
endpoint | String | Sim | O endpoint a ser cadastrado.
unit_identifier | Objeto | Sim | Objeto contendo informações sobre a unidade a ser consultada.
unit_identifier[type] | String | Sim | O tipo de parâmetro identificador da unidade. Atualmente, somente cnpj é aceito.
unit_identifier[value] | String | Sim | O valor do parâmetro identificador da unidade. Um CNPJ, por exemplo.


## Objeto Confirmation
> Exemplo de confirmação de compra:

```json
    {
        "status": "success",
        "user_info": {
            "type": "cpf",
            "value": "00000000000"
        },
        "merchandise": [
            {
                "name": "PROPULSOR DE ÍONS",
                "code": "192301",
                "value": 20000
            }, {
                "name": "REATOR ARC",
                "code": "192302",
                "value": 50000
            }
        ],
        "total": 70000
    }
```
A confirmação enviada para o endpoint registrado possui o formato visto ao lado.