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
  from aws_cdk import core
  from my_cdk_app.my_cdk_app_stack import MyCdkAppStack

  app = core.App()
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
  from aws_cdk import core
  from aws_cdk import aws_s3 as s3

  class MyCdkAppStack(core.Stack):

      def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
          super().__init__(scope, id, **kwargs)

          # Definição de um bucket S3
          s3.Bucket(self, 
                    "MyFirstBucket",
                    versioned=True,
                    removal_policy=core.RemovalPolicy.DESTROY)
  ```

#### Diretório `tests/`

- **`__init__.py`**: Arquivo que torna este diretório um pacote Python. Geralmente está vazio.

- **`unit/`**: Subdiretório para testes unitários.

  - **`__init__.py`**: Arquivo que torna este diretório um pacote Python. Geralmente está vazio.
  
  - **`test_my_cdk_app_stack.py`**: Arquivo onde você pode escrever testes unitários para a sua stack usando frameworks como `unittest` ou `pytest`. Exemplo de conteúdo:
    ```python
    import unittest
    from aws_cdk import core
    from my_cdk_app.my_cdk_app_stack import MyCdkAppStack

    class TestMyCdkAppStack(unittest.TestCase):

        def test_s3_bucket_created(self):
            app = core.App()
            stack = MyCdkAppStack(app, "test-stack")
            template = app.synth().get_stack_by_name("test-stack").template

            # Verificar se um bucket S3 foi criado
            self.assertIn("AWS::S3::Bucket", template["Resources"])
    ```

Esta estrutura proporciona uma base organizada para desenvolver e gerenciar a infraestrutura com AWS CDK em Python, facilitando a manutenção, a escalabilidade e a colaboração no projeto.