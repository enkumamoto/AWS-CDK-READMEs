# O que é AWS CDK?

O AWS CDK (Cloud Development Kit) é uma ferramenta que permite definir sua infraestrutura na AWS usando linguagens de programação familiares como Python, JavaScript, TypeScript, Java, e C#. Em vez de escrever longos arquivos de configuração JSON ou YAML, você pode usar o CDK para escrever código que define e provisiona recursos da AWS de maneira mais intuitiva e modular.

Aqui estão alguns pontos-chave sobre o AWS CDK:

1. **Codificação em Linguagens Familiares**: Você pode usar linguagens de programação que já conhece, o que facilita a integração da infraestrutura como código (IaC) com o desenvolvimento de aplicações.

2. **Componentes Reutilizáveis**: O CDK permite criar componentes reutilizáveis chamados "Constructs". Esses Constructs podem ser usados para definir padrões de infraestrutura que você pode reutilizar em diferentes partes do seu projeto ou em diferentes projetos.

3. **Stacks e App**: No CDK, você organiza seus recursos em "Stacks". Uma Stack é um contêiner que agrupa recursos relacionados. Um aplicativo CDK (App) pode conter várias Stacks.

4. **Infraestrutura como Código (IaC)**: O CDK gera CloudFormation templates a partir do seu código. O CloudFormation é um serviço que permite gerenciar seus recursos da AWS através de templates.

5. **Bibliotecas Ricas**: O CDK vem com um conjunto de bibliotecas que facilitam a criação de recursos comuns, como APIs, bancos de dados, funções Lambda, redes, e muito mais.
---

# Conceitos de Contruções e Stacks

No AWS CDK (Cloud Development Kit), os conceitos de **construções** (constructs) e **stacks** são fundamentais para a organização e gerenciamento de recursos da infraestrutura como código.

### Stacks

**Stacks** são as unidades de implantação no AWS CDK. Elas representam a coleção de construções que formam a sua aplicação ou uma parte dela. Cada stack é convertida em um modelo do CloudFormation quando você implanta a aplicação CDK.

- **Stack** é uma coleção de recursos que você quer implantar juntos. 
- Cada stack é implementada como uma classe que herda de `cdk.Stack`.
- Você pode definir múltiplas stacks dentro de uma aplicação CDK para separar diferentes partes da sua infraestrutura.

### Construções (Constructs)

**Construções** são os blocos de construção básicos do AWS CDK. Elas representam componentes ou padrões reutilizáveis da infraestrutura. Podem ser desde recursos individuais da AWS (como uma instância EC2 ou um bucket S3) até conjuntos mais complexos de recursos que trabalham juntos para cumprir um propósito específico (como uma aplicação web completa).

Existem três níveis de construções no AWS CDK:

1. **Nível 1 (L1) - Constructs de Nível Baixo**:
   - Representam diretamente os recursos da AWS.
   - São gerados a partir dos arquivos CloudFormation.
   - Exemplo: `s3.CfnBucket` representa um bucket S3 no nível CloudFormation.
    ```python
    from aws_cdk import (
    Stack,  # Importa a classe Stack, que é a base para todos os stacks CDK
    aws_s3 as s3  # Importa o módulo aws_s3 e o renomeia como s3
    )
    from constructs import Construct  # Importa a classe Construct, que é a base para todos os constructs CDK

    class CdkS3ExampleStack(Stack):  # Define uma nova classe que herda de Stack

    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)  # Chama o construtor da classe base

        # Cria um bucket S3 usando o construct de nível 1 (L1)
        bucket = s3.CfnBucket(self, "MyBucket",
            bucket_name="my-cdk-s3-bucket"  # Define o nome do bucket (opcional)
        )
        # CfnBucket: Construct de Nível 1 que representa um recurso AWS CloudFormation para um bucket S3
        # self: Refere-se ao stack atual
        # "MyBucket": Identificador lógico do recurso no template CloudFormation
        # bucket_name: Parâmetro opcional para definir o nome do bucket S3
    ```

2. **Nível 2 (L2) - Constructs de Nível Médio**:
   - São abstrações opinativas dos recursos de nível 1, oferecendo interfaces mais amigáveis e funcionalidades integradas.
   - Exemplo: `s3.Bucket` é uma construção L2 que facilita a criação e configuração de um bucket S3.
    ```python
    from aws_cdk import (
        Stack,
        aws_s3 as s3  # Importa o módulo aws_s3 e o renomeia como s3
    )
    from constructs import Construct  # Importa a classe Construct, que é a base para todos os constructs CDK

    class CdkS3L2ExampleStack(Stack):

        def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
            super().__init__(scope, construct_id, **kwargs)  # Chama o construtor da classe base

            # Cria um bucket S3 usando o construct de nível 2 (L2)
            bucket = s3.Bucket(self, "MyL2Bucket",
                # Define o nome do bucket (opcional)
                bucket_name="my-l2-cdk-s3-bucket",
                # Define a política de remoção para deletar o bucket ao destruir o stack
                removal_policy=s3.RemovalPolicy.DESTROY,
                # Define o bloqueio de versão para o bucket
                versioned=True
            )
            # Explicação:
            # - Bucket: É o construct de nível 2 para o recurso S3 Bucket.
            # - self: Refere-se ao escopo do stack atual.
            # - "MyL2Bucket": O identificador lógico do bucket.
            # - bucket_name: O nome do bucket S3 (opcional).
            # - removal_policy: Define a política de remoção do bucket (DESTROY = destruir o bucket ao remover o stack).
            # - versioned: Habilita o versionamento no bucket S3.
    ```

3. **Nível 3 (L3) - Constructs de Nível Alto**:
   - São padrões completos que combinam múltiplos recursos para resolver casos de uso específicos.
   - Exemplo: `aws_solutions_constructs.aws_s3_lambda` que configura um bucket S3 junto com uma função Lambda que responde a eventos desse bucket.
    ```python
    from aws_cdk import (
        Stack,
        aws_s3 as s3,
        aws_s3_deployment as s3deploy  # Importa o módulo aws_s3_deployment para realizar o deploy de conteúdo
    )
    from constructs import Construct
    
    class CdkS3L3ExampleStack(Stack):

        def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
            super().__init__(scope, construct_id, **kwargs)

            # Cria um bucket S3 de Nível 2 (L2)
            bucket = s3.Bucket(self, "MyL3Bucket",
                bucket_name="my-l3-cdk-s3-bucket",
                removal_policy=s3.RemovalPolicy.DESTROY,
                versioned=True
            )

            # Realiza o deploy de conteúdo estático para o bucket S3
            deployment = s3deploy.BucketDeployment(self, "DeployStaticContent",
                sources=[s3deploy.Source.asset("./static-content")],  # Define o diretório de origem do conteúdo estático
                destination_bucket=bucket,
                destination_key_prefix="web/static"  # Prefixo de destino no bucket S3
            )
            # Explicação:
            # - Bucket: Construct de Nível 2 que cria um bucket S3.
            # - BucketDeployment: Construct de Nível 3 que realiza o deploy de conteúdo estático para o bucket S3.
            # - sources: Define a origem do conteúdo a ser enviado para o bucket S3.
            # - destination_bucket: Define o bucket de destino.
            # - destination_key_prefix: Define o prefixo de destino para o conteúdo dentro do bucket S3.
    ```

### Resumo

- **Construções** são blocos de construção reutilizáveis que representam componentes da infraestrutura.
- **Stacks** são coleções de construções que formam uma unidade de implantação.

Esses conceitos permitem a você estruturar, organizar e gerenciar a infraestrutura de forma eficiente e modular no AWS CDK.
