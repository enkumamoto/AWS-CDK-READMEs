# Aspect
Um **aspecto** (aspect) é uma funcionalidade que permite aplicar ações ou modificações a uma coleção de construções (constructs) dentro de uma pilha (stack) de maneira consistente e sistemática. Os aspectos são usados para inspecionar ou modificar as construções depois que elas foram criadas, mas antes que a síntese (synth) seja realizada, proporcionando uma maneira de aplicar lógica transversal em várias partes da aplicação.

### Definindo um Aspecto

### jsii

A biblioteca **jsii** é um componente fundamental do AWS CDK que permite a criação de bibliotecas de infraestrutura que podem ser usadas em diferentes linguagens de programação. O jsii (JavaScript Interoperability Interface) é uma tecnologia que permite escrever código em TypeScript e, automaticamente, gerar bindings para outras linguagens, como Python, Java, C#, entre outras.

**Principais pontos do jsii:**

- **Interoperabilidade:** Permite que bibliotecas escritas em uma linguagem (principalmente TypeScript) sejam usadas em outras linguagens sem a necessidade de reescrever o código.
- **Reutilização de Código:** Facilita a reutilização de bibliotecas e componentes de infraestrutura em diferentes projetos e linguagens.
- **Integração com AWS CDK:** É a base que permite que o AWS CDK suporte múltiplas linguagens, oferecendo uma experiência consistente para desenvolvedores, independentemente da linguagem escolhida.

### IAspect

A interface **IAspect** no AWS CDK define um contrato para implementar aspectos. Aspectos são usados para inspecionar e potencialmente modificar construções (constructs) dentro de uma pilha (stack). Ao implementar a interface IAspect, você cria classes que podem ser aplicadas a uma coleção de construções, permitindo aplicar lógica transversal a essas construções.

**Principais componentes da interface IAspect:**

- **Método `visit`:** Este é o único método que precisa ser implementado. Ele define a lógica que será aplicada a cada construção visitada. O método recebe um parâmetro que representa a construção que está sendo visitada.

### Resumindo

- **jsii**: Permite escrever bibliotecas de infraestrutura em TypeScript e usá-las em múltiplas linguagens, facilitando a interoperabilidade e a reutilização de código no AWS CDK.
- **IAspect**: Define um contrato para implementar aspectos que podem inspecionar e modificar construções dentro de uma pilha, permitindo aplicar lógica transversal de forma consistente.

No exemplo, um aspecto chamado `MyFirstAspect` é definido implementando a interface `IAspect`:

```python
import jsii
from aws_cdk import (
    IAspect,
)

@jsii.implements(IAspect)
class MyFirstAspect:

    def visit(self, construct_visited):
        print(f"{construct_visited.node.path} - {construct_visited.__class__.__name__}")
```

- `MyFirstAspect` é uma classe que implementa a interface `IAspect`.
- O método `visit` é chamado para cada construção dentro da pilha.
- `construct_visited` representa a construção que está sendo visitada.
- A linha `print(f"{construct_visited.node.path} - {construct_visited.__class__.__name__}")` imprime o caminho e o nome da classe da construção visitada.

### Aplicando o Aspecto à Pilha

Para aplicar o aspecto à pilha, ele é anexado usando `cdk.Aspects.of(root_stack).add(MyFirstAspect())`:

```python
app = cdk.App()

root_stack = cdk.Stack(app, "RootStack")

network_stack = NetworkStack(root_stack, 'NetworkStack')

application_stack = MySampleAppWebserverStack(root_stack, "MySampleAppWebserverStack", my_vpc=network_stack.vpc)

cdk.Aspects.of(root_stack).add(MyFirstAspect())

cdk.Tags.of(network_stack).add("category", "networking")

cdk.Tags.of(application_stack).add("category", "application", priority=200)

app.synth()
```

- `root_stack` é a pilha raiz onde os aspectos serão aplicados.
- `cdk.Aspects.of(root_stack).add(MyFirstAspect())` adiciona o aspecto `MyFirstAspect` à pilha raiz.
- Quando `app.synth()` é chamado, o CDK percorre todas as construções na pilha e aplica o método `visit` do aspecto a cada construção.

### Objetivo dos Aspectos

Os aspectos são úteis para vários casos, como:

1. **Aplicação de Tags:** Adicionar tags de forma consistente a recursos.
2. **Validações:** Validar se construções específicas atendem a certos critérios.
3. **Modificações:** Alterar propriedades de construções existentes.
4. **Auditoria:** Registrar informações sobre as construções para auditoria ou monitoramento.

### Observações
1. O AWS CDK permite ter multiplos arquivos para aspects diferentes.
2. Tembém pode configurar multiplos aspects em um mesmo arquivo.

### Conclusão

Os aspectos no AWS CDK fornecem uma poderosa ferramenta para aplicar lógica transversal em construções, permitindo a inspeção e modificação das construções de maneira uniforme. No exemplo, o aspecto `MyFirstAspect` imprime o caminho e o nome da classe de cada construção visitada, demonstrando como os aspectos podem ser utilizados para inspecionar construções dentro de uma pilha.