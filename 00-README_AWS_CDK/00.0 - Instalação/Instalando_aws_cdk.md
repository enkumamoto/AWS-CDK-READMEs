## Como Instalar AWS CDK no Ubuntu
Como instalar o AWS Cloud Development Kit (CDK) e suas dependências no Ubuntu 22.04.

## Passo 1: Atualize seu sistema

Primeiro, certifique-se de que seu sistema está atualizado. Abra o terminal e execute:

```
sudo apt update
sudo apt upgrade -y
```

## Passo 2: Instale o Node.js e o npm

O AWS CDK é baseado no Node.js. Você precisa instalá-lo junto com o npm (gerenciador de pacotes do Node.js). No Ubuntu, você pode usar o NodeSource para instalar a versão mais recente do Node.js.
Execute os seguintes comandos:

```
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs

```

Verifique a instalação do Node.js e npm:

```
node -v
npm -v
```

## Passo 3: Instale o AWS CDK
Agora que você tem o Node.js e o npm instalados, você pode instalar o AWS CDK. Execute o seguinte comando:

```
npm install -g aws-cdk
```

Verifique a instalação do AWS CDK

```
cdk --version
```

## Passo 4: Configure as credenciais da AWS

Para que o AWS CDK possa interagir com seus recursos na AWS, você precisa configurar suas credenciais da AWS. Se você ainda não tem o AWS CLI instalado, faça isso com o comando:

```
sudo apt install -y awscli
```

Depois, configure suas credenciais executando:

```
aws configure
```

## Passo 5: Inicialize um novo projeto AWS CDK

Crie um novo diretório para o seu projeto CDK e navegue até ele:

```
mkdir meu-projeto-cdk
cd meu-projeto-cdk
```

Inicialize um novo projeto CDK:

```
cdk init app --language python
```

## Passo 6: Configurar ambiente python
É recomendável usar um ambiente virtual para gerenciar suas dependências Python. Crie e ative um ambiente:

```
# Crie um ambiente virtual
python3 -m venv .venv

# Ative o ambiente virtual
source .venv/bin/activate

```

Agora, com o ambiente virtual ativado, você pode instalar o AWS CDK para Python:

```
pip install aws-cdk-lib
pip install constructs
```


## Passo 7: Instalar Dependências Adicionais

Durante a inicialização do projeto, o CDK criará um arquivo requirements.txt. Para instalar as dependências listadas, execute:

```
pip install -r requirements.txt
```

## Passo 8: Bootstrap da Stack

Antes de implantar a stack, você precisa "bootstrap" do ambiente. Isso cria recursos necessários no CloudFormation para suportar o CDK.

```bash
cdk bootstrap aws://<account-id>/<region>
```

Substitua `<account-id>` pelo ID da sua conta AWS e `<region>` pela região onde você deseja implantar os recursos (por exemplo, `us-east-1`).

## Passo 9: Sintetizar e Implantar a Pilha

Depois de configurar tudo, você pode sintetizar (gerar) o CloudFormation template e implantar a pilha:

```
# Sintetizar a pilha
cdk synth

# Implantar a pilha
cdk deploy
```

## Passo 10: Limpar Recursos

Para excluir os recursos criados, você pode usar o comando:

```
cdk destroy
```

---

# Conceitos de Contruções e Stacks

No AWS CDK (Cloud Development Kit), os conceitos de **construções** (constructs) e **stacks** são fundamentais para a organização e gerenciamento de recursos da infraestrutura como código.