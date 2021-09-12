# Java-Message-Service
:smile: # Java Message Service (JMS) com o ActiveMQ
### Exemplos funcionais de como usar o Java Message Service (JMS) com o ActiveMQ

# Prerequisites
### Habilitar atributos do VPC
Para garantir que seu corretor esteja acessível dentro de sua VPC, você deve habilitar oenableDnsHostnameseenableDnsSupportAtributos do VPC. Para obter mais informações, consulteSupport a DNS em sua VPCnoAmazon VPC User Guide.

### Habilitar conexões de entrada
- Faça login noAmazon MQ;
- Na lista de agentes, escolha o nome do operador (por exemplo,MyBroker);
- NoMyBroker, noConexões do, observe os endereços e as portas do URL do console da Web do broker e dos protocolos de nível de fio;
- NoDetalhes, emSegurança e rede, escolha o nome do security group do ou a página Grupos de segurança do painel do EC2 é exibida;
- Na lista de security group, escolha seu security group;
- Na parte inferior da página, escolha Entrada e a seguir selecione Editar;
- NoEditar regras de entrada, adicione uma regra para cada URL ou endpoint que você deseja que seja acessível publicamente (o exemplo a seguir mostra como fazer isso para um console da Web do broker).

  - Escolha Adicionar regra;
  - Em Tipo, selecione TCP personalizado;
  - para oIntervalo de Portas, digite a porta do console da Web (8162);
  - para oOrigem, sairCustom (Personalizado)selecionado e digite o endereço IP do sistema que você deseja ser capaz de acessar o console da Web (por exemplo,192.0.2.1);
  - Escolha Save (Salvar).
  - 
Agora seu operador pode aceitar conexões de entrada.

# Adicionar dependências de Java

### Adicione oactivemq-client.jareactivemq-pool.jarao caminho da classe Java. O exemplo a seguir mostra essas dependências em um arquivo pom.xml do projeto Maven.

```
<dependencies>
    <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-client</artifactId>
        <version>5.15.8</version>
    </dependency>
    <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-pool</artifactId>
        <version>5.15.8</version>
    </dependency>
</dependencies>
```

# Importante
No código de exemplo a seguir, produtores e consumidores são executados em um único segmento. Para sistemas de produção (ou para testar o failover de instância do corretor), certifique-se de que seus produtores e consumidores sejam executados em hosts ou threads separados.

```
/*
 * Copyright 2010-2019 Amazon.com, Inc. or its affiliates. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License").
 * You may not use this file except in compliance with the License.
 * A copy of the License is located at
 *
 *  https://aws.amazon.com/apache2.0
 *
 * or in the "license" file accompanying this file. This file is distributed
 * on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either
 * express or implied. See the License for the specific language governing
 * permissions and limitations under the License.
 *
 */
    
    import org.apache.activemq.ActiveMQConnectionFactory;
    import org.apache.activemq.jms.pool.PooledConnectionFactory;
    
    import javax.jms.*;
    
    public class AmazonMQExample {
    
    // Specify the connection parameters.
    private final static String WIRE_LEVEL_ENDPOINT 
            = "ssl://b-1234a5b6-78cd-901e-2fgh-3i45j6k178l9-1.mq.us-east-2.amazonaws.com:61617";
    private final static String ACTIVE_MQ_USERNAME = "MyUsername123";
    private final static String ACTIVE_MQ_PASSWORD = "MyPassword456";
    
    public static void main(String[] args) throws JMSException {
        final ActiveMQConnectionFactory connectionFactory =
                createActiveMQConnectionFactory();
        final PooledConnectionFactory pooledConnectionFactory =
                createPooledConnectionFactory(connectionFactory);
    
        sendMessage(pooledConnectionFactory);
        receiveMessage(connectionFactory);
    
        pooledConnectionFactory.stop();
    }
    
    private static void
    sendMessage(PooledConnectionFactory pooledConnectionFactory) throws JMSException {
        // Establish a connection for the producer.
        final Connection producerConnection = pooledConnectionFactory
                .createConnection();
        producerConnection.start();
    
        // Create a session.
        final Session producerSession = producerConnection
                .createSession(false, Session.AUTO_ACKNOWLEDGE);
    
        // Create a queue named "MyQueue".
        final Destination producerDestination = producerSession
                .createQueue("MyQueue");
    
        // Create a producer from the session to the queue.
        final MessageProducer producer = producerSession
                .createProducer(producerDestination);
        producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
    
        // Create a message.
        final String text = "Hello from Amazon MQ!";
        final TextMessage producerMessage = producerSession
                .createTextMessage(text);
    
        // Send the message.
        producer.send(producerMessage);
        System.out.println("Message sent.");
    
        // Clean up the producer.
        producer.close();
        producerSession.close();
        producerConnection.close();
    }
    
    private static void
    receiveMessage(ActiveMQConnectionFactory connectionFactory) throws JMSException {
        // Establish a connection for the consumer.
        // Note: Consumers should not use PooledConnectionFactory.
        final Connection consumerConnection = connectionFactory.createConnection();
        consumerConnection.start();
    
        // Create a session.
        final Session consumerSession = consumerConnection
                .createSession(false, Session.AUTO_ACKNOWLEDGE);
    
        // Create a queue named "MyQueue".
        final Destination consumerDestination = consumerSession
                .createQueue("MyQueue");
    
        // Create a message consumer from the session to the queue.
        final MessageConsumer consumer = consumerSession
                .createConsumer(consumerDestination);
    
        // Begin to wait for messages.
        final Message consumerMessage = consumer.receive(1000);
    
        // Receive the message when it arrives.
        final TextMessage consumerTextMessage = (TextMessage) consumerMessage;
        System.out.println("Message received: " + consumerTextMessage.getText());
    
        // Clean up the consumer.
        consumer.close();
        consumerSession.close();
        consumerConnection.close();
    }
    
    private static PooledConnectionFactory
    createPooledConnectionFactory(ActiveMQConnectionFactory connectionFactory) {
        // Create a pooled connection factory.
        final PooledConnectionFactory pooledConnectionFactory =
                new PooledConnectionFactory();
        pooledConnectionFactory.setConnectionFactory(connectionFactory);
        pooledConnectionFactory.setMaxConnections(10);
        return pooledConnectionFactory;
    }
    
    private static ActiveMQConnectionFactory createActiveMQConnectionFactory() {
        // Create a connection factory.
        final ActiveMQConnectionFactory connectionFactory =
                new ActiveMQConnectionFactory(WIRE_LEVEL_ENDPOINT);
    
        // Pass the username and password.
        connectionFactory.setUserName(ACTIVE_MQ_USERNAME);
        connectionFactory.setPassword(ACTIVE_MQ_PASSWORD);
        return connectionFactory;
    }
    }
```

