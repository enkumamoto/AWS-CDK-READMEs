# Estrutura de diretórios

Quando você executa o comando `cdk init app --language python` para iniciar um novo projeto AWS CDK em Python, o AWS CDK cria uma estrutura de diretórios padrão que serve como ponto de partida para o desenvolvimento da infraestrutura como código. Aqui está uma descrição detalhada da estrutura de diretórios e dos arquivos gerados:

```
my-cdk-app/
├── README.md
├── app.py
├── cdk.json
├── requirements.txt
├── source.bat
├── source.ps1
├── my_cdk_app/
│   ├── __init__.py
│   ├── my_cdk_app_stack.py
├── tests/
│   ├── __init__.py
│   └── unit/
│       ├── __init__.py
│       └── test_my_cdk_app_stack.py
├── .gitignore
└── .venv/
```

### Detalhamento dos Arquivos e Diretórios

#### Arquivos na Raiz

- **`README.md`**: Arquivo de documentação que descreve o seu projeto e fornece informações básicas sobre como configurá-lo e usá-lo.

- **`app.py`**: Arquivo principal do aplicativo CDK. É aqui que a aplicação CDK é instanciada e as stacks são definidas e adicionadas ao aplicativo. Exemplo de conteúdo:
  ```python
  # app.py
  import aws_cdk as cdk
  from my_cdk_app.my_cdk_app_stack import MyCdkAppStack

  app = cdk.App()
  MyCdkAppStack(app, "my-cdk-app")

  app.synth()
  ```

- **`cdk.json`**: Arquivo de configuração do CDK, que define como o CDK deve ser executado. Exemplo de conteúdo:
  ```json
  {
    "app": "python3 app.py"
  }
  ```

- **`requirements.txt`**: Arquivo que lista as dependências Python necessárias para o projeto. Inclui o `aws-cdk.core` e outras bibliotecas CDK que você venha a utilizar. Exemplo de conteúdo:
  ```plaintext
  aws-cdk.core
  ```

- **`source.bat` e `source.ps1`**: Scripts para ativar o ambiente virtual do Python no Windows (Batch e PowerShell, respectivamente).

- **`.gitignore`**: Arquivo de configuração que especifica quais arquivos e diretórios devem ser ignorados pelo Git. Inclui diretórios como `.venv` e arquivos gerados pelo CDK.

- **`.venv/`**: Diretório onde o ambiente virtual do Python será configurado (este diretório pode não existir imediatamente após a inicialização, mas será criado quando você configurar o ambiente virtual).

#### Diretório `my_cdk_app/`

- **`__init__.py`**: Arquivo que torna este diretório um pacote Python. Geralmente está vazio.

- **`my_cdk_app_stack.py`**: Arquivo onde você define a sua stack principal. Este arquivo contém a definição dos recursos AWS que compõem sua aplicação. Exemplo de conteúdo:
  ```python
  from aws_cdk import (
      Stack,
      aws_s3 as s3,
      Duration,
      CfnOutput,
  )
  
  from constructs import Construct
  
  class MyCdkAppStack(Stack):
  
      def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
          super().__init__(scope, construct_id, **kwargs)
  
          bucket = s3.Bucket(self, "TestBucket",
                    lifecycle_rules = [
                      s3.LifecycleRule(
                          expiration = Duration.days(3)
                      )
                    ]
                  )
          CfnOutput(self, "BucketName", 
                    value = bucket.bucket_name)
  ```

#### Diretório `tests/`

- **`__init__.py`**: Arquivo que torna este diretório um pacote Python. Geralmente está vazio.

- **`unit/`**: Subdiretório para testes unitários.

  - **`__init__.py`**: Arquivo que torna este diretório um pacote Python. Geralmente está vazio.
  
  - **`test_my_cdk_app_stack.py`**: Arquivo onde você pode escrever testes unitários para a sua stack usando frameworks como `unittest` ou `pytest`. Exemplo de conteúdo:
    ```python
    import aws_cdk as cdk
    import pytest
    from aws_cdk.assertions import Template
    from py_starter_stack import PyStarterStack

    def test_s3_bucket_created():
        app = cdk.App()
        stack = MyCdkAppStack(app, "MyTestStack")
        
        # Synthesizar a stack
        template = Template.from_stack(stack)
        
        # Verificar se um bucket S3 é criado
        template.has_resource_properties("AWS::S3::Bucket", {
            "LifecycleConfiguration": {
                "Rules": [
                    {
                        "Status": "Enabled",
                        "ExpirationInDays": 3
                    }
                ]
            }
        })
        
        # Verificar se a saída do nome do bucket foi criada
        template.has_output("BucketName", {})
    ```