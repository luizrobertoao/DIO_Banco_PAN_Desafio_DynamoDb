# Desafio Bootcamp Banco PAN - Boas práticas com Amazon DynamoDB.

## Descrição:
Criação de um banco de dados para uma aplicação de coleção de figurinhas especiais do álbum "Fifa World cup Qatar 2022".

### Serviço utilizado
- Amazon DynamoDB
- Amazon CLI para execução em linha de comando

### Execução do desafio:


- Criando uma tabela

```
aws dynamodb create-table \
    --table-name Sticker \
    --attribute-definitions \
        AttributeName=Player,AttributeType=S \
        AttributeName=Rarity,AttributeType=S \
    --key-schema \
        AttributeName=Player,KeyType=HASH \
        AttributeName=Rarity,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```

- Inserir um item

```
aws dynamodb put-item \
    --table-name Sticker \
    --item file://itemsticker.json \
```

- Inserir múltiplos itens

```
aws dynamodb batch-write-item \
    --request-items file://batchsticker.json
```

- Criar um index global secundário baseado em Certification

```
aws dynamodb update-table \
    --table-name Sticker \
    --attribute-definitions AttributeName=Certification,AttributeType=BOOL \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"Certification-index\",\"KeySchema\":[{\"AttributeName\":\"Certification\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no Player e Certification

```
aws dynamodb update-table \
    --table-name Sticker \
    --attribute-definitions\
        AttributeName=Player,AttributeType=S \
        AttributeName=Certification,AttributeType=BOOL \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"CertificationPlayer-index\",\"KeySchema\":[{\"AttributeName\":\"Player\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"Certification\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no Rarity e Type

```
aws dynamodb update-table \
    --table-name Sticker \
    --attribute-definitions\
        AttributeName=Rarity,AttributeType=S \
        AttributeName=Type,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"RarityType-index\",\"KeySchema\":[{\"AttributeName\":\"Rarity\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"Type\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Pesquisar item por Player

```
aws dynamodb query \
    --table-name Sticker \
    --key-condition-expression "Player = :player" \
    --expression-attribute-values  '{":player":{"S":"Neymar"}}'
```
- Pesquisar item por Player e Rarity

```
aws dynamodb query \
    --table-name Sticker \
    --key-condition-expression "Player = :player and Rarity = :rarity" \
    --expression-attribute-values file://conditions.json
```

- Pesquisa pelo index secundário baseado em Certification

```
aws dynamodb query \
    --table-name Sticker \
    --index-name Certification-index \
    --key-condition-expression "Certification = :statement" \
    --expression-attribute-values  '{":statement":{"BOOL":"true"}}'
```

- Pesquisa pelo index secundário baseado no Player e em Certification

```
aws dynamodb query \
    --table-name Sticker \
    --index-name CertificationPlayer-index \
    --key-condition-expression "Player = :v_player and Certification = :v_certification" \
    --expression-attribute-values  '{":v_player":{"S":"Neymar"},":v_certification":{"BOOL":"true"} }'
```

- Pesquisa pelo index secundário baseado em Rarity e Type

```
aws dynamodb query \
    --table-name Sticker \
    --index-name RarityType-index \
    --key-condition-expression "Rarity = :v_rarity and Type = :v_type" \
    --expression-attribute-values  '{":v_rarity":{"S":"Gold"},":v_type":{"S":"Legend"} }'
```