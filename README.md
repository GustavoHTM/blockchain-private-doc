# Blockchain na AWS

## Introdução

Para implementar uma solução de blockchain utilizando AWS, vamos usar um EC2 para executar os contêineres Docker que irão gerenciar nós e membros da blockchain. Este guia irá cobrir a configuração do EC2, a criação dos contêineres Docker e a integração com outros serviços AWS para garantir uma comunicação eficiente e segura.

# Introdução ao Blockchain e Livro-Razão Distribuído

Antes de começar a configurar a sua aplicação blockchain, é importante entender alguns conceitos fundamentais. Uma blockchain é um tipo de livro-razão distribuído que registra todas as transações realizadas em uma rede de forma imutável e transparente. Cada bloco na cadeia contém um conjunto de transações, e esses blocos são ligados uns aos outros, formando uma cadeia.

## Blockchain como um Livro-Razão Distribuído
Uma blockchain funciona como um livro-razão (ledger) distribuído, onde todos os participantes da rede (nós) têm uma cópia idêntica desse livro. Isso garante que todas as transações sejam verificáveis e que nenhuma entidade única controle a rede.

### Canais na Blockchain

Um canal (ou *channel*) na blockchain é uma sub-rede de comunicação onde transações e dados são compartilhados de forma isolada entre os membros autorizados. Em essência, ele permite que diferentes grupos de membros dentro da mesma rede de blockchain tenham um livro-razão compartilhado, mas independente, garantindo privacidade e controle sobre quem pode ver e interagir com os dados.

#### Por que Usar Canais?

- **Segurança e Privacidade:** Os canais permitem que apenas membros específicos tenham acesso a determinadas transações e dados. Isso é crucial para manter a privacidade em ambientes onde diferentes organizações compartilham a mesma infraestrutura de blockchain.
  
- **Isolamento de Transações:** Cada canal mantém seu próprio livro-razão, o que significa que as transações em um canal não afetam ou se misturam com as transações em outro. Isso reduz o risco de conflitos e mantém a integridade dos dados.

#### Quantos Canais Usar?

- **1 Canal (Melhor para Sistemas Simples):** Para a maioria dos sistemas simples, um único canal é a solução mais eficiente. Ele simplifica a arquitetura, reduz a complexidade na gestão e facilita a auditoria, pois todas as transações estão centralizadas em um único livro-razão.
  
- **Vários Canais (Necessário para Casos Complexos):** Em sistemas mais complexos, onde diferentes partes precisam acessar e processar dados de forma independente, ou onde há requisitos rigorosos de privacidade, pode ser vantajoso criar múltiplos canais. Cada canal pode ser personalizado para atender às necessidades específicas de um grupo de membros ou de um tipo específico de transação.

Em resumo, a escolha do número de canais depende das necessidades do sistema. Para sistemas simples e diretos, um único canal é normalmente a melhor solução. Já em sistemas complexos, onde a segregação de dados e a privacidade são cruciais, múltiplos canais podem ser necessários.

## Tipos de Componentes Blockchain

### Membro
Os membros são usuários da blockchain e têm a capacidade de realizar processos administrativos. Por exemplo, dois membros podem remover um terceiro membro, enquanto um só não pode. O percentual de poder de decisão de cada membro pode ser configurado, e a soma das porcentagens deve totalizar 100%. Cada membro deve possuir pelo menos um nó.

### Nó
Os nós são responsáveis por armazenar e replicar o "livro razão" da blockchain. Normalmente, um nó gerencia um único canal, onde todas as informações da blockchain são armazenadas e replicadas. O livro razão é essencialmente uma cópia do estado da blockchain, garantindo que todos os nós tenham uma visão consistente dos dados.

# Diferenciação entre Membros e Nós

Na arquitetura de blockchain, é crucial entender a diferença entre membros e nós:

- **Membros:** Representam entidades ou organizações dentro da rede blockchain. Cada membro pode ter um ou mais nós associados a ele.
  
- **Nós:** São os componentes que realizam transações e mantêm uma cópia do livro-razão. Os nós podem ser de diferentes tipos, como nós de validação ou nós de peer.

Essa distinção é fundamental, pois influencia como as transações são gerenciadas e validadas dentro da rede.

## Estrutura de Diretórios e Arquivos

- **Dockerfile**: Arquivo para construir a imagem do Docker para o nó e o membro.
- **docker-compose.yml**: Arquivo para orquestrar múltiplos contêineres Docker (opcional).
- **config.yaml**: Arquivo de configuração para a blockchain e para os contêineres.

## Configuração da AWS

### EC2 para Blockchain
Para iniciar, você precisará de um EC2 configurado para executar contêineres Docker. Para configurar o EC2:

1. **Escolher uma AMI**: Use uma AMI compatível com Docker, como Ubuntu.
2. **Configurar o EC2**: Defina o tipo de instância e as regras de segurança apropriadas.
3. **Instalar Docker**: Após iniciar a instância, instale Docker e Docker Compose.

### Dockerfile
Crie um Dockerfile para configurar o ambiente dos nós e membros da blockchain:

```Dockerfile
# Escolher uma imagem base
FROM hyperledger/fabric-peer:latest

# Copiar arquivos de configuração
COPY config.yaml /etc/hyperledger/fabric/

# Definir comandos para iniciar o nó
CMD ["start-peer"]
```

docker-compose.yml
Se necessário, use o Docker Compose para orquestrar múltiplos contêineres:

```yaml
Copiar código
version: '3'
services:
  peer:
    build: .
    ports:
      - "7051:7051"
  member:
    image: hyperledger/fabric-member:latest
    ports:
      - "7052:7052"
```

## Fluxo de Dados

### Interação com a Blockchain
A comunicação com a blockchain será intermediada pela AWS, envolvendo os seguintes componentes:

1. **API Gateway**: Recebe requisições do sistema e as encaminha para a função Lambda.
2. **Lambda Function**: Configura e valida a requisição, autentica o usuário e interage com a blockchain.
3. **Blockchain**: Armazena e replica os registros na rede.

![Blockchain-Workflux](blockchain-workflux.png)

### Tipos de Ações

- **Invoke**: Armazena informações na blockchain. A resposta inclui o número do bloco onde os dados foram registrados.
- **Query**: Busca informações na blockchain para verificação de auditoria. Retorna os blocos contendo os dados solicitados.

## Exemplos de Implementação

### Função Lambda
A função Lambda deve configurar e validar a requisição, autenticar o usuário e interagir com a blockchain para realizar as ações de armazenamento e consulta.


```python
import json
import boto3

def lambda_handler(event, context):
    # Configuração e validação
    api_gateway = boto3.client('apigateway')
    # Processar a requisição
    # Interagir com a blockchain
    return {
        'statusCode': 200,
        'body': json.dumps('Requisição processada com sucesso')
    }
```

### BlockchainUtils
A classe `BlockchainUtils` será responsável por enviar dados para a blockchain de forma assíncrona. Ela comunicará com a API Gateway, que por sua vez interagirá com a função Lambda para executar as operações necessárias.

```java
Copiar código
public class BlockchainUtils {

    public void sendToBlockchain(String jsonData) {
        // Enviar dados para a API Gateway
        // Configurar e enviar requisição
    }

    public String queryBlockchain(String query) {
        // Enviar consulta para a API Gateway
        // Receber e processar resposta
    }
}
```

## Contratos Inteligentes
Os contratos inteligentes podem ser utilizados para validar transações antes de serem armazenadas na blockchain. Você pode criar e configurar contratos personalizados para validação de regras de negócios.

Para criar e configurar um contrato inteligente, consulte o exemplo no repositório [Smart Contract JS](https://github.com/GustavoHTM/smart-contract-js).

## Referências
- [Construindo uma Aplicação Blockchain com Java na AWS](https://aws.amazon.com/pt/blogs/database/building-a-blockchain-application-in-java-using-amazon-managed-blockchain/)
- [Tutorial de Introdução ao Hyperledger Fabric na AWS](https://docs.aws.amazon.com/managed-blockchain/latest/hyperledger-fabric-dev/managed-blockchain-get-started-tutorial.html)
