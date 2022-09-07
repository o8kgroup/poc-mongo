# MONGO POC

> POC COM PROPÓSITO DE TESTAR A OPERAÇÃO DE "AGREGATE" DO MONGO_DB

## PROPOSTA
> DADA UMA COLLECTION CHAMADA "PRODUCTS" CONTENDO OS DOCUMENTOS CONFORME ABAIXO:

~~~json
/* 1 */
{
    "_id" : ObjectId("6317fb10dae9a407ef063cb9"),
    "nome" : "Produto 1",
    "codigo" : 9057.0,
    "etiqueta" : "A1 - Etiqueta do produto código: 9057",
    "garantia" : {
        "ativa" : true,
        "vencimento" : ISODate("2023-09-07T01:59:44.903Z")
    }
}

/* 2 */
{
    "_id" : ObjectId("6317fb10dae9a407ef063cba"),
    "nome" : "Produto 1",
    "codigo" : 9057.0,
    "etiqueta" : "A2 - Etiqueta do produto código: 9057",
    "garantia" : {
        "ativa" : true,
        "vencimento" : ISODate("2023-09-07T01:59:44.903Z")
    }
}

/* 3 */
{
    "_id" : ObjectId("6317fb10dae9a407ef063cbb"),
    "nome" : "Produto 1",
    "codigo" : 9057.0,
    "etiqueta" : "A3 - Etiqueta do produto código: 9057",
    "garantia" : {
        "ativa" : true,
        "vencimento" : ISODate("2023-09-07T01:59:44.903Z")
    }
}

/* 4 */
{
    "_id" : ObjectId("6317fb10dae9a407ef063cbc"),
    "nome" : "Produto 2",
    "codigo" : 8057.0,
    "etiqueta" : "A1 - Etiqueta do produto código: 8057",
    "garantia" : {
        "ativa" : true,
        "vencimento" : ISODate("2023-09-07T01:59:44.903Z")
    }
}

/* 5 */
{
    "_id" : ObjectId("6317fb10dae9a407ef063cbd"),
    "nome" : "Produto 2",
    "codigo" : 8057.0,
    "etiqueta" : "A2 - Etiqueta do produto código: 8057",
    "garantia" : {
        "ativa" : true,
        "vencimento" : ISODate("2023-09-07T01:59:44.903Z")
    }
}
~~~

> ATRAVÉS DO USO DO "AGGREGATE" (E SEUS ESTÁGIOS) DEVE-SE AGRUPAR AS "ETIQUETAS" DE CADA PRODUTO EM UMA LISTA. PRODUZINDO A SAÍDA ABAIXO:
~~~json
/* 1 */
{
    "_id" : 9057.0,
    "nome" : "Produto 1",
    "codigo" : 9057.0,
    "garantia" : {
        "ativa" : true,
        "vencimento" : ISODate("2023-09-07T02:12:01.207Z")
    },
    "etiquetas" : [ 
        "A1 - Etiqueta do produto código: 9057", 
        "A2 - Etiqueta do produto código: 9057", 
        "A3 - Etiqueta do produto código: 9057"
    ]
}

/* 2 */
{
    "_id" : 8057.0,
    "nome" : "Produto 2",
    "codigo" : 8057.0,
    "garantia" : {
        "ativa" : true,
        "vencimento" : ISODate("2023-09-07T02:12:01.207Z")
    },
    "etiquetas" : [ 
        "A1 - Etiqueta do produto código: 8057", 
        "A2 - Etiqueta do produto código: 8057"
    ]
}
~~~
## PASSOS PARA REPRODUZIR A CRIAÇÃO DA COLLECTION

> 1. NO MONGO CRIAR A BASE DE DADOS
~~~
use poc_aggregate
~~~
> SE O BANCO DE DADOS NÃO EXISTIR, ENTÃO O CLUSTER DO MONGODB IRÁ CRIÁ-LO.

> CRIE A COLLECTION COM SEUS DOCUMENTOS 

~~~js
db.runCommand(
   {
      insert: "products",
      documents: [ 
        { 
            nome: "Produto 1", 
            codigo: 9057, 
            etiqueta: "A1 - Etiqueta do produto código: 9057",
            garantia: {
                "ativa": true,
                "vencimento": new Date(ISODate().getTime() + 1000 * 86400 * 365)
            }
        },
        { 
            nome: "Produto 1", 
            codigo: 9057, 
            etiqueta: "A2 - Etiqueta do produto código: 9057",
            garantia: {
                "ativa": true,
                "vencimento": new Date(ISODate().getTime() + 1000 * 86400 * 365)
            }
        },
        { 
            nome: "Produto 1", 
            codigo: 9057, 
            etiqueta: "A3 - Etiqueta do produto código: 9057",
            garantia: {
                "ativa": true,
                "vencimento": new Date(ISODate().getTime() + 1000 * 86400 * 365)
            }
        },
        { 
            nome: "Produto 2", 
            codigo: 8057, 
            etiqueta: "A1 - Etiqueta do produto código: 8057",
            garantia: {
                "ativa": true,
                "vencimento": new Date(ISODate().getTime() + 1000 * 86400 * 365)
            }
        },
        { 
            nome: "Produto 2", 
            codigo: 8057, 
            etiqueta: "A2 - Etiqueta do produto código: 8057",
            garantia: {
                "ativa": true,
                "vencimento": new Date(ISODate().getTime() + 1000 * 86400 * 365)
            }
        },
        
    ]
   }
)
~~~
> USE O AGGREGATE PARA CHEGAR NA SOLUÇÃO
~~~js
db.getCollection('products').aggregate([
    {
        $group: {
            _id: "$codigo",
            nome: {"$first": "$nome"},
            codigo: {"$first": "$codigo"},
            garantia: {"$first":"$garantia"},
            etiquetas: {
                $push: "$etiqueta"
            },
        }
    },
])
~~~