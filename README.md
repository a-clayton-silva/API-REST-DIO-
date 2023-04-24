# Roteiro para o desenvolvimento da atividade prática 

## Serviços AWS utilizados

- Amazon Cognito
- Amazon DynamoDB
- Amazon API Gateway
- AWS Lambda

## Etapas do desenvolvimento

### Criando uma API REST no Amazon API Gateway

- API Gateway Dashboard --> Create API --> REST API --> Build:
	Protocol - REST
	Create new API
	API name [app_api]
	Endpoint Type - Regional
	Create API
- Resources --> Actions --> Create Resource --> Resource Name [Items] --> Create Resource

### No Amazon DynamoDB

- DynamoDB Dashboard --> Tables --> Create table:
	Table name [Items]
	Partition key [id]
	Create table

### No AWS Lambda

#### Função para inserir item

- Lambda Dashboard --> Create function:
	Author from scratch
	Name [put_item_function]
	Create function
- Inserir código da função ```put_item_function.js disponível no git na pasta /src``` --> Deploy
- Configuration --> Permissions --> Execution role --> Add Permissions --> Create inline policy
- Service - DynamoDB --> Access level - Write --> PutItem --> Resources --> Add arn --> Selecionar o arn da tabela criada no DynamoDB --> Add --> Review policy --> Name [lambda_dynamodb_putItem_policy] --> Create policy

### Integrando o API Gateway com o Lambda backend

- API Gateway Dashboard --> Selecionar a API criada --> Resources --> Selecionar o resource criado --> Action --> Create method - POST:
	Integration type - Lambda function
	Use Lambda Proxy Integration - Lambda function
	Selecionar a Lambda Funcion 
	Save
- Actions --> Deploy API --> Deployment Stage - New Stage --> Stage name [dev] --> Deploy

### No POSTMAN inclusao de item no dynamodb

- Create collecion --> Add Request --> Method POST --> Copiar o endpoint gerado no API Gateway - API Stages, dev, items, post, invoke url
- Body --> Raw --> JSON --> Adicionar o seguinte body
```
{
  "id": "003",
  "price": 600
}
```
- Send

### No Amazon Cognito

- Cognito Dashboard --> Manage User Pools --> Create a User Pool 
- Authentication providers - Cognito user pool --> Email --> Next
- Configurações Opcionais --> Next
- Required attributes - name --> Next
- Send email with Cognito --> Next
- Pool name [TestPool] --> Use the Cognito Hosted UI --> colocar um nome unico - Cognito domain --> App client name [TestClient] --> Allowed callback URLs [https://oauth.pstmn.io/v1/callback] --> Advanced app client settings - OAuth 2.0 grant types--> Implicit grant --> OpenID Connect scopes - profile --> Next
- Create Pool
- Users --> create user --> Send an email invitation --> Email address --> Password --> Create user
 
### Criando um autorizador do Amazon Cognito para uma API REST no Amazon API Gateway

- API Gateway Dashboard --> Selecionar a API criada --> Authorizers --> Create New Authorizer
- Name [CognitoAuth] --> Type - Cognito --> Cognito User Pool [pool criada anteriormente] --> Token Source [Authorization]
- Resources --> selecionar o resource criado --> selecionar o método POST criado --> Method Request --> Authorization - Selecionar o autorizador criado --> Actions --> Deploy api --> selecionar o deployment stage --> deploy

### No POSTMAN

- Add request --> Authorization
- Type - OAuth 2.0
- Token Name [tokenApp]
- Grant Type [Authorization code with pkce]
- Callback URL [https://oauth.pstmn.io/v1/callback]
- Auth URL [https://hotmaildiocog.auth.us-east-1.amazoncognito.com/oauth2/authorize]
- Access Token URL [https://hotmaildiocog.auth.us-east-1.amazoncognito.com/oauth2/token]
- Client ID - obter o Client ID do Cognito em App clients
- Scope [email - openid]
- Client Authentication [Send client credentials in body]
- Get New Acces Token
- Copiar o idtoken gerado
- No request anterior para inclusao de item no dynamodb
- Authorization --> Type - Bearer Token --> Inserir o token copiado
- Send
