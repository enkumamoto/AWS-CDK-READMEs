# Cross-Stack
Cross-stacks referem-se à prática de criar múltiplos stacks que compartilham recursos entre si, permitindo uma separação lógica e modularização da infraestrutura. Isso é útil para organizar melhor os recursos e facilitar a manutenção do código.

No exemplo fornecido, temos dois arquivos principais de stacks: `network_stack.py` e `my_sample_app_webserver_stack.py`. O primeiro define uma VPC (Virtual Private Cloud), e o segundo cria uma instância EC2 que usa essa VPC. Abaixo, explico cada parte relevante do código:

### network_stack.py
Este arquivo define o stack que cria a VPC:

```python
from aws_cdk import (
    Stack,
    aws_ec2 as ec2,
)
from constructs import Construct

class NetworkStack(Stack):

    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        self.vpc = ec2.Vpc(self, 'MyVpc', nat_gateways=0)
```

- **NetworkStack**: Cria uma VPC sem NAT Gateways (para simplificação).
- **self.vpc**: O recurso VPC é exposto como um atributo da classe para que possa ser referenciado em outros stacks.

### my_sample_app_webserver_stack.py
Este arquivo define o stack que cria a instância EC2:

```python
from aws_cdk import (
    Stack,
    CfnOutput,
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

        # Restante do código...
```

- **MySampleAppWebserverStack**: Cria uma instância EC2 que usa a VPC passada como parâmetro.
- **my_vpc**: Este parâmetro permite que a instância EC2 use a VPC definida no `NetworkStack`.

### app.py
Este arquivo orquestra a criação dos stacks:

```python
#!/usr/bin/env python3
import os

import aws_cdk as cdk

from my_sample_app_webserver.my_sample_app_webserver_stack import MySampleAppWebserverStack
from my_sample_app_webserver.network_stack import NetworkStack

app = cdk.App()

network_stack = NetworkStack(app, 'NetworkStack')

MySampleAppWebserverStack(app, "MySampleAppWebserverStack",
                          my_vpc=network_stack.vpc)

app.synth()
```

- **network_stack**: Instancia o `NetworkStack`, criando a VPC.
- **MySampleAppWebserverStack**: Cria o stack do servidor web, passando a VPC criada pelo `NetworkStack`.

### Resumo
- **Cross-stacks** permitem que recursos como VPC sejam definidos em um stack e usados em outro, promovendo reutilização e modularidade.
- No exemplo, o stack `NetworkStack` cria uma VPC, e o stack `MySampleAppWebserverStack` utiliza essa VPC para lançar uma instância EC2.
