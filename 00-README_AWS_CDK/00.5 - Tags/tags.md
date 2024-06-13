# Tags
As **tags** são usadas para adicionar metadados a recursos da AWS. Essas tags são pares chave-valor que podem ser anexados a quase todos os recursos da AWS e são úteis para fins de gerenciamento, organização, automação e cobrança. As tags permitem categorizar e identificar recursos de maneira mais eficiente.

### Uso de Tags no Código

No exemplo fornecido, tags são adicionadas a um recurso EC2 (uma instância) usando a classe `Tags` do CDK. Vamos detalhar cada parte do código relevante para entender como as tags são aplicadas.

#### Definição da Stack e Criação da Instância EC2

```python
from aws_cdk import (
    Stack,
    CfnOutput,
    Tags,
    aws_ec2 as ec2,
    aws_s3_assets as s3_assets
)
from constructs import Construct

class MySampleAppWebserverStack(Stack):

    def __init__(self, scope: Construct, construct_id: str, my_vpc: ec2.Vpc, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)
        
        web_server = ec2.Instance(self, 'WebServer',
                                  machine_image=ec2.MachineImage.latest_amazon_linux2(),
                                  instance_type=ec2.InstanceType.of(instance_class=ec2.InstanceClass.T2,
                                                                    instance_size=ec2.InstanceSize.MICRO),
                                  vpc=my_vpc,
                                  vpc_subnets=ec2.SubnetSelection(subnet_type=ec2.SubnetType.PUBLIC),
                                  user_data_causes_replacement=True)
```

Neste trecho de código, uma instância EC2 é criada dentro da stack `MySampleAppWebserverStack`. A instância é configurada com uma imagem Amazon Linux 2, tipo de instância T2 Micro, e é implantada em uma VPC pública.

#### Adicionando Tags à Instância EC2

```python
# Adicionando tags para construções
Tags.of(web_server).add("category", "webserver")
Tags.of(web_server).add("subcategory", "primary",
                        include_resource_types = ['AWS::EC2::Instance'])
```

Aqui, duas tags são adicionadas à instância EC2 (`web_server`):

1. **Primeira Tag**: Adiciona uma tag com chave `"category"` e valor `"webserver"` à instância EC2.
    ```python
    Tags.of(web_server).add("category", "webserver")
    ```

2. **Segunda Tag**: Adiciona uma tag com chave `"subcategory"` e valor `"primário"` à instância EC2, mas especifica que essa tag deve ser aplicada apenas aos recursos do tipo `AWS::EC2::Instance`.
    ```python
    Tags.of(web_server).add("subcategory", "primário", include_resource_types = ['AWS::EC2::Instance'])
    ```

### Como Funciona

1. **Criação de Tags**: As tags são criadas usando o método `add` da classe `Tags`. Este método recebe pelo menos dois parâmetros: a chave da tag e o valor da tag.
2. **Aplicação de Tags**: As tags são aplicadas ao recurso especificado (neste caso, `web_server`, que é uma instância EC2). O método `of` da classe `Tags` é usado para especificar o recurso ao qual as tags serão aplicadas.
3. **Filtragem por Tipo de Recurso**: A segunda tag utiliza o parâmetro `include_resource_type` para garantir que a tag seja aplicada apenas aos recursos do tipo especificado (`AWS::EC2::Instance`).

### Benefícios das Tags

1. **Organização**: As tags ajudam a organizar os recursos, facilitando a busca e o agrupamento de recursos semelhantes.
2. **Gerenciamento**: Permitem categorizar e gerenciar recursos de forma eficiente, especialmente em ambientes com muitos recursos.
3. **Automação**: As tags podem ser usadas em scripts de automação e políticas de gerenciamento de recursos.
4. **Cobrança**: Facilita a alocação de custos e a análise de despesas, pois as tags podem ser usadas para rastrear os custos por projeto, equipe ou ambiente.

### Resumo

- **Tags**: Pares chave-valor que são anexados aos recursos AWS para fins de gerenciamento, organização, automação e cobrança.
- **Uso no CDK**: As tags são adicionadas aos recursos usando a classe `Tags` e o método `add`.
- **Exemplo**: O código fornecido adiciona tags a uma instância EC2 para categorizá-la como um "webserver" e como "primário".
- **Benefícios**: Organização, gerenciamento eficiente, automação facilitada e melhor alocação de custos.

Usar tags no AWS CDK é uma prática recomendada para manter a infraestrutura bem organizada e gerenciada, especialmente em ambientes complexos e dinâmicos.