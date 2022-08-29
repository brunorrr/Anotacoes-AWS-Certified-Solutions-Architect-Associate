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

### EC2 Nitro

EC2 Nitro é o nome da plataforma utilizada para a próxima geração de instâncias EC2.

Novidades com o nitro:

* Nova tecnologia de virtualização.
* Performance Melhorada:

    * Opções de conexão de rede mais performáticas (enhanced networking, HPC, IPv6)
    * Maior velocidade de comunicação com o EBS (64.000 IOPS comparado com 32.000 das gerações anteriores)
    * Segurança aprimorada

Você saberá se está utilizando uma instância do tipo Nitro pela geração da mesma, ex: C5, C5a, D3. Gerações mais antigas não são do tipo Nitro.

### Costomizando a utilização de vCPUs

As instâncias da AWS, tal como um computador covencional, rodam em processadores multicore e multithread. Porém, você é cobrado pelo total de cores e threads que você utiliza, ex: instância do tipo r4.2xlarge te permite usar 4 cores com 2 threads rodando por core. Você pode customizar isso, se você estiver rodando algum serviço que deixará algum core oscioso, ao criar a instância diga que quer utilizar menos cores ou menos threads por core, desta forma você consome menos recurso e paga menos. Só é possível customizar a quantidade de cores e thredas na inicialização da instância.

### Reservas de capacidade EC2

Permitem que você reserve instâncias EC2 sem ter que seguir a regra de 1-3 anos. Você deve especificar quantas AZs irá utilizar, quantas instâncias pretende utilizar e as características delas, tipo, SO, plataforma, etc...

Use isso combinado com o sistema principal de reserva de instâncias e Saving Plans para economizar mais.

## EBS (Elastic Block Store)

EBS é um serviço de drives de rede que podem ser utilizados juntamente com instâncias EC2. Um volume EBS emula um disco físico, ainda que seja, na verdade, um disco virtual disponível na rede. Cada volume EBS pode ser associado à apenas uma instância EC2 por vez, enquanto que uma instância EC2 pode ter N volumes EBS em uma relação 1-N, sendo N > 0. Cada volume EBS está associado à uma AZ e se comunica com sua instância por meio de uma interface de rede(como um drive acessado pela rede no SO).

EBS é elegível para o Free Tier com a disponibilidade de 30GB do tipo SSD ou HD por mês.

Volumes EBS podem ser adquiridos e precificados com base no seu tamanho(em GB) e tráfego(em IOPS). Ambas as opções são modificáveis com o tempo.

Ao criar uma instância EC2 e um volume EBS, você pode determinar se o volume será excluido após o termino da instância, a depender da sua necessidade, isso pode ser feito pelo console, por CLIs ou scripts.

### Snapshots EBS

É possível criar snapshots se seus volumes EBSs, elas conterão um backup de todos os arquivos do volume. Não é necessário desligar o volume de uma instância(ainda que seja recomendado), você pode criar várias snapshots do mesmo volume em momentos diferentes, também pode copiar essas snapshots para outras AZs e Regions, desta forma, pode espalhar o volume EBS para AZs diferentes.

#### Snapshot Archive

São storages de armazenamento de longo prazo em que você pode deixar seus backups. Podem chegar a ser 75% mais barato que o armazenamento de snapshots regular. Uma vez que tenha armazenado uma snapshot no archive, leva-se entre 24 e 72 horas para restaura-la e deixa-la pronta para uso.

#### Lixeira de Snapshots(EBS Snapshot Recycle Bin)

A lixeira das Snaphots é uma regra que você aplica a snapshots em que você pretende excluir, essas lixeiras guardam as snapshots por um tempo, de forma que se você excluir uma acidentalmente, você pode recupera-la sem problemas. O tempo de retenção de uma snapshot na lixeira pode variar de 1 dia a 1 ano a depender da sua configuração. Após este periodo a snapshot é excluida definitivamente.

## AMI (Amazon Machine Images)

AMIs são customizações que podem ser feitas em cima de instâncias EC2, exemplo: Você quer criar uma instância EC2 em Linux com a JRE 17 instalada, ao invés de criar uma instância EC2 e instalar a JRE nela, você cria uma imagem de instância(AMI) e, ao criar a instância, cria ela a partir desta imagem.

Você pode criar suas próprias AMIs, de acordo com sua necessidade, pode utilizar AMIs públicas criadas pela própria AWS ou, ainda, pode obter(de graça ou não) imagens disponíveis na AWS Marketplace, onde outros usuários disponibilizam(de graça ou não) AMIs criadas por eles para outros usuários.

Para criar uma AMI você deve criar uma instância EC2 tradicional, customiza-la, para-la e construir uma AMI em cima desta, isso também criará uma snapshot do volume EBS, a partir daí, você poderá criar instâncias EC2 a partir desta AMI.
