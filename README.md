# Anotacoes-AWS-Certified-Solutions-Architect-Associate

## EC2

WIP: Trazer definições básicas de EC2 do repo de cloud practitioner.

### IPs

Cada instância EC2 possui oque é chamado de IP privado e IP público. IP privado é limitado ao escopo das instâncias EC2 da AWS de forma que apenas as instâncias conseguem acessar umas às outras por IP privado. O IP público é feito para ser acessado de fora do escopo das instâncias EC2, como um acesso SSH de sua máquina ao serviço. Tantos os IPs públicos, quanto privados, são recriados quando uma instância é reiniciada.

As instâncias EC2 também possuem oque é chamado de IP Elástico, são IPs que podem ser reassociado entre as instâncias, se uma instância, ou interface, possui um IP elástico e para de funcionar, este mesmo IP pode ser associado à outra instância, ou interface. IPs elásticos não são recomendados pois existem alternativas mais adequadas e escaláveis para fazer o gerenciamento de rede de instâncias.

### Placement Groups

EC2 é um serviço do tipo IaaS, portanto, a interação com o hardware fica emulada pela AWS, porém, se necessário, você pode solicitar que estas instâncias ficem alocadas em locais específicos dentro das AZs, existem 3 técnicas de alocação de instâncias EC2, elas são oque é chamado de Placement Group.

#### Cluster

Nesta técnica, o grupo de instâncias alterado fica dentro da mesma AZ e do mesmo rack, de forma que a latência de comunicação entre elas é ínfima, também permite que haja uma grande banda de conexão. O ponto negativo deste tipo de abordagem é a fraca proteção contra indisponibilidade, se o rack, ou a AZ caírem, você perde todas as instâncias de uma vez. Casos de uso: Funções batch de BigData que precisam lidar com uma quantidade astronômica de dados e processa-los no menor tempo possíevl, Outras aplicações que possuem uma carga alta de transferência entre instâncias e necessitam de respostas rápidas.

#### Spread

Nesta técnica, diferente do Cluster, você divide as instâncias em racks separados e divide estes racks entre AZs separadas, desta forma você só possui uma instância por rack. Fazendo isso você reduz o risco de perda total das instâncias a quase zero. O ponto negativo desta abordagem é o fato de que existem uma quantidade limite de racks por region e, portanto, você está limitado a colocar **7** instâncias dentro de um Placement Group nesta técnica. Casos de uso: Aplicações que necessitam ter uma alta disponibilidade, mesmo que isto ocorra em detrimento da performance. Aplicações críticas que precisam estar isoladas umas das outras.

#### Partition

Nesta técnica, você distribui pelas AZs oque é chamado de partições, cada partição irá rodar em um rack de hardware único, cada partição pode ter de 1 a 100 instâncias, cada Placement Group pode ter de 1 a 7 partições. As instâncias rodando dentro da mesma partição terão uma latência reduzida para comunicarem entre si. Esta abordagem pode ser colocada como um misto entre o cluster e o spread. Você pode descobrir detalhes sobre as partições acessando os arquivos de metadados delas. Casos de uso: Sistemas de BigData, Banco de Dados e Filas.

### [ENI (Elastic Network Interface)](https://aws.amazon.com/pt/blogs/aws/new-elastic-network-interfaces-in-the-virtual-private-cloud/)

ENIs são componentes lógico (Virtual Network Card) que atuam como interfaces de redes permitindo que instâncias EC2 se comuniquem com a rede.
Cada ENI possui os seguintes atributos:

* Um IP **IPv4** privado primário e mais um secundário.
* Um IP elástico (IPv4) por IPv4 privado.
* Um IPv4 público.
* Um ou mais security groups.
* Um endereço MAC.

Apesar de estarem anexadas à instâncias EC2, as ENIs podem ser desassociadas destas e religadas em outras instâncias, isso é útil em caso de falha em alguma instância e para se ter um melhor controle sobre as conexões de suas instâncias. Também, cada ENI está associada à uma AZ.

### Hibernando instâncias EC2

EC2 Hibernate é o processo de colocar uma instância EC2 em modo hibernação, desta forma, tal como um SO comum, a instância irá interromper todos os processos, incluindo o SO, porém, manterá a memória RAM intácta de forma que ao reiniciar o sistema, todas as configurações e variáveis já estarão disponíveis na memória e, portanto, não precisarão ser carregadas do zero. Ao colocar uma instância para hibernar, o EC2 cria um arquivo na raiz do volume EBS principal e aloca neste arquivo toda a memória RAM da instância.

Casos de uso:

* Processos a serem executados a longo prazo.
* Mantendo a memória RAM salva.
* Serviços que precisam de um bom tempo para inicializar.

Condições:

* O volume EBS associado à instância precisa ser criptografado.
* A maioria  dos tipos de instâncias EC2 são permitidas(C3, C4, C5, I3, M3, etc...)
* A memória RAM da instância deve ser menor que 150GB.
* Não disponível para instâncias do tipo "Bare Metal".
* Várias AMIs permitidas(Amazon Linux 2, Linux AMI, Ubuntu, Windows, etc...).
* Disponível para instâncias On-Demand, Reservadas e Spot.
* O período de hibernação não pode ultrapassar 60 dias.
