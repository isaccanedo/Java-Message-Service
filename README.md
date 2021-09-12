# Java-Message-Service
:smile: # Java Message Service (JMS) com o ActiveMQ
### Exemplos funcionais de como usar o Java Message Service (JMS) com o ActiveMQ

Prerequisites
Habilitar atributos do VPC
Para garantir que seu corretor esteja acessível dentro de sua VPC, você deve habilitar oenableDnsHostnameseenableDnsSupportAtributos do VPC. Para obter mais informações, consulteSupport a DNS em sua VPCnoAmazon VPC User Guide.

Habilitar conexões de entrada
Faça login noAmazon MQ.

Na lista de agentes, escolha o nome do operador (por exemplo,MyBroker).

NoMyBroker, noConexões do, observe os endereços e as portas do URL do console da Web do broker e dos protocolos de nível de fio.

NoDetalhes, emSegurança e rede, escolha o nome do security group do ou  .

A página Grupos de segurança do painel do EC2 é exibida.

Na lista de security group, escolha seu security group.

Na parte inferior da página, escolha Entrada e a seguir selecione Editar.

NoEditar regras de entrada, adicione uma regra para cada URL ou endpoint que você deseja que seja acessível publicamente (o exemplo a seguir mostra como fazer isso para um console da Web do broker).

Escolha Adicionar regra.

Em Tipo, selecione TCP personalizado.

para oIntervalo de Portas, digite a porta do console da Web (8162).

para oOrigem, sairCustom (Personalizado)selecionado e digite o endereço IP do sistema que você deseja ser capaz de acessar o console da Web (por exemplo,192.0.2.1).

Escolha Save (Salvar).

Agora seu operador pode aceitar conexões de entrada.

Adicionar dependências de Java

