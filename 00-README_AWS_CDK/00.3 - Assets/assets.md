# Conceito de Assets

No AWS CDK (Cloud Development Kit), **assets** são recursos que permitem incluir arquivos externos, como arquivos estáticos, imagens de contêiner, código-fonte de aplicações Lambda, entre outros, diretamente nas suas stacks. Eles são utilizados para empacotar e implantar esses arquivos juntamente com sua infraestrutura definida pelo CDK.

### Tipos de Assets

Existem dois tipos principais de assets no AWS CDK:

1. **Assets de Arquivo**: Permitem incluir arquivos ou diretórios do sistema de arquivos local no seu aplicativo CDK. Esses arquivos são carregados para o Amazon S3 e referenciados nas suas stacks.

2. **Assets de Imagem de Contêiner**: Permitem construir e carregar imagens de contêiner Docker para o Amazon Elastic Container Registry (ECR), facilitando a implantação de aplicações baseadas em contêiner.

### Uso de Assets de Arquivo

Um uso comum de assets de arquivo é empacotar o código-fonte para funções AWS Lambda. Aqui está um exemplo de como utilizar assets para uma função Lambda:

#### Exemplo de Uso de Assets para uma Função Lambda

1. **Estrutura de Diretórios**:
    ```
    my-cdk-app/
    ├── app.py
    ├── cdk.json
    ├── lambda/
    │   └── handler.py
    ├── my_cdk_app/
    │   ├── __init__.py
    │   └── my_cdk_app_stack.py
    └── requirements.txt
    ```

2. **Arquivo `lambda/handler.py`**:
    ```python
    import json
    import os
    import logging
    import boto3 # AWS Python SDK

    # Configure logging
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)

    # Initialize the DynamoDB client
    dynamodb_client = boto3.client('dynamodb')

    def lambda_handler(event, context):
        '''
        Returns all products from the DynamoDB table provided.

        Environment variables:
            - TABLE_NAME: The name of the DynamoDB table scanned.
        '''

        logger.info(f"Received event: {json.dumps(event, indent=2)}")

        # Scan the DynamoDB table to get all products
        products = dynamodb_client.scan(
            TableName=os.environ['TABLE_NAME']
        )

        return {
            "statusCode": 200,
            "body": json.dumps(products['Items'])
        }
    ```

3. **Definindo a Stack com Assets no `my_cdk_app/my_cdk_app_stack.py`**:
    ```python
    from aws_cdk import (
    RemovalPolicy,
    CfnOutput,
    Duration,
    Stack,
    aws_dynamodb as dynamodb,
    aws_lambda as lambda_,
    aws_cloudwatch as cloudwatch)

    from constructs import Construct


    class ServerlessAppStack(Stack):

        def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
            super().__init__(scope, construct_id, **kwargs)

            products_table = dynamodb.Table (self, "ProductsTable",
                                            partition_key = dynamodb.Attribute(
                                                name = "id",
                                                type = dynamodb.AttributeType.STRING
                                            ),
                                            billing_mode = dynamodb.BillingMode.PAY_PER_REQUEST,
                                            removal_policy = RemovalPolicy.DESTROY)
            
            # Definição da função Lambda utilizando assets
            product_list_function = lambda_.Function(self, "ProductListFunction",
                                                    code = lambda_.Code.from_asset("lambda_src"),
                                                    handler = "product_list_function.lambda_handler",
                                                    runtime = lambda_.Runtime.PYTHON_3_10,
                                                    environment = {
                                                        "TABLE_NAME": products_table.table_name
                                                    })
            # Concedendo permissões para função Lambda para ler dados das tabelas do DynamoDB
            products_table.grant_read_data(product_list_function.role)

            # Adicionando uma URL Lambda pa a Função Lambda para executa via internet
            product_list_url = product_list_function.add_function_url(
                auth_type = lambda_.FunctionUrlAuthType.NONE
                )
            
            # Adicionando um output stack para acessar facilmente a função URL
            CfnOutput(self, "ProductUrl",
                    value = product_list_url.url)
            
            # Configuração de alarme para métricas de erros para Lambda Function
            errors_metric = product_list_function.metric_errors(
                label = "ProductListFunction Error",
                period = Duration.minutes(5),
                statistic = cloudwatch.Stats.SUM
            )

            errors_metric.create_alarm(self, "ProductListErrorAlarm",
                                    evaluation_periods = 1,
                                    threshold = 1,
                                    comparison_operator = cloudwatch.ComparisonOperator.GREATER_THAN_OR_EQUAL_TO_THRESHOLD,
                                    treat_missing_data = cloudwatch.TreatMissingData.IGNORE)
    ```

Aqui temos uma função lambda que retorna todos os produtos de uma tabela DynamoDB.

```python
import json
import os
import logging
import boto3 # AWS Python SDK

# Configure logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# Initialize the DynamoDB client
dynamodb_client = boto3.client('dynamodb')

def lambda_handler(event, context):
    '''
    Retorna todos os produtos da tabela DynamoDB fornecida.

    Variáveis ​​ambientais:
    - TABLE_NAME: o nome da tabela do DynamoDB verificada.
    '''

    logger.info(f"Received event: {json.dumps(event, indent=2)}")

    # Scan the DynamoDB table to get all products
    products = dynamodb_client.scan(
        TableName=os.environ['TABLE_NAME']
    )

    return {
        "statusCode": 200,
        "body": json.dumps(products['Items'])
    }

```

Aqui está uma explicação linha por linha:

1. `import json`, `import os`, `import logging`, `import boto3`: Importa os módulos necessários para lidar com JSON, variáveis de ambiente, logging e o SDK da AWS para Python (boto3).

2. `logger = logging.getLogger()` e `logger.setLevel(logging.INFO)`: Configura o logger para o nível de log INFO.

3. `dynamodb_client = boto3.client('dynamodb')`: Inicializa o cliente DynamoDB usando o boto3.

4. `def lambda_handler(event, context):`: Define a função lambda_handler, que é o ponto de entrada da função lambda. Ela recebe dois argumentos, `event` e `context`, que contêm informações sobre o evento que disparou a função e o contexto de execução da função.

5. `logger.info(f"Received event: {json.dumps(event, indent=2)}")`: Registra informações sobre o evento recebido pela função lambda.

6. `products = dynamodb_client.scan(TableName=os.environ['TABLE_NAME'])`: Executa uma operação de scan na tabela DynamoDB especificada pela variável de ambiente `TABLE_NAME` para obter todos os produtos.

7. `return {"statusCode": 200, "body": json.dumps(products['Items'])}`: Retorna uma resposta HTTP com status 200 e o corpo contendo os itens da tabela DynamoDB no formato JSON.


### Funcionamento Interno dos Assets

Quando você usa assets no AWS CDK:

1. **Empacotamento**: Os arquivos especificados são empacotados no seu diretório local.
2. **Carregamento**: Os assets são carregados para um bucket S3 (para arquivos) gerenciado pelo CDK.
3. **Referenciamento**: O CDK gera automaticamente as permissões necessárias e os metadados para que os recursos da AWS possam acessar esses assets.

### Resumo

- **Assets de Arquivo**: Empacotam e carregam arquivos ou diretórios locais para o S3.
- **Utilização**: Facilitam a inclusão de arquivos externos, como código-fonte para Lambda ou imagens Docker, nas suas stacks CDK.
- **Automatização**: O CDK cuida de empacotar, carregar e gerenciar permissões para os assets, simplificando o processo de implantação.

Essa funcionalidade é poderosa para gerenciar e implantar componentes complexos da infraestrutura com facilidade e eficiência no AWS CDK.