## Configure e install Jboss EAP 7

- 1.Create a service user
`sudo useradd --no-create-home --shel /bin/false/ jboss-eap`

- 2.Set the JBOSS_HOME AND JBOSS_USER
`sudo vim /opt/jboss-eap/bin/init.d/jboss-eap.conf`

- 3 - Copy the service files
`sudo cp /opt/jboss-eap/bin/init.d/jboss-eap.conf /etc/default/`
 `sudo cp /opt/jboss-eap/bin/init.d/jboss-eap-rhel.sh /etc/init.d/`

- 4.Make the script executable
`sudo chmod +x /etc/init.d/jboss-eap-rhel.sh`

- 5.Set the service to start automatically
`sudo chkconfig --add jboss-eap-rhel.sh`
`sudo chkconfig jboss-eap.sh on`
`systemctl enable`

- 6.Create the JBoss EAP's run directory and set the ownership
`sudo mkdir /var/run/jboss-eap`
`sudo mkdir /var/run/jboss-eap`

- 7.Change the ownership of the jboss_home directory
`sudo chown -R jboss-eap:jboss-eap /opt/jboss-eap/`

1- set up the enfocement file
vim ~/selinux/jboss-eap-rhel.te

################################

running the standalone.sh script to start the
EAP server:
$ ./standalone.sh -Djboss.server.base.dir=/path/to/base/directory \
-Djboss.home.dir=/path/to/home/directory

The port-offset attribute can be changed without editing standalone.xml by
using the jboss.socket.binding.port-offset property on the command line.
For example:
$ ./standalone.sh -Djboss.socket.binding.port-offset=10000
 
 
Using a customized configuration file

Copy the following folders from /opt/jboss-eap-7.0/standalone into the new EAP
location /home/student/JB248/labs/custom-eap:
* configuration
* deployments
* lib

[student@workstation ~]$ cd /opt/jboss-eap-7.0/standalone
[student@workstation standalone]$ cp -r configuration deployments lib \
~/JB248/labs/custom-eap

...
<socket-binding-group name="standard-sockets" default-interface="public" port-
offset="${jboss.socket.binding.port-offset:10000}">
...

student@workstation standalone]$ cd /opt/jboss-eap-7.0/bin
[student@workstation bin]$ ./standalone.sh \
-Djboss.server.base.dir=/home/student/JB248/labs/custom-eap/

################################

Verificação de loggs

standalone.xml >> logger

logger category="com.arjuna">
<level name="WARN"/>
</logger>
<logger category="org.jboss.as.config">
<level name="DEBUG"/>
</logger>
<logger category="sun.rmi">
<level name="WARN"/>
</logger>


Adicionar tipos de logs 

################################

Interfaces

<interfaces>
<interface name="management">
<inet-address value="${jboss.bind.address.management:127.0.0.1}"/>
</interface>
<interface name="public">
<inet-address value="${jboss.bind.address:127.0.0.1}"/>
</interface>
</interfaces>


Ouvir em todas as interfaces
<any-address/>




Os usuários podem configurar o endereço IP para acessar o Console de Gerenciamento ao
iniciar o EAP, usando a linha de comando variável -bmanagement. Por exemplo:

$ ./standalone.sh -bmanagement 127.0.0.1



<interfaces>
<interface name="ipv4-global">
<any-ipv4-address/>
</interface>
</interfaces>


Somente a um endereço IPv4:

<interfaces>
<interface name="ipv4-global">
<any-ipv4-address/>
</interface>
</interfaces>


Os critérios podem ser combinados para serem bastante específicos em relação a uma
definição de interface. Por exemplo, a interface a seguir define uma sub-rede específica com
suporte a multicast. O endereço deve estar funcionando e não poderá ser de ponto a ponto:

<interfaces>
<interface name="my_specific_interface">
<subnet-match value="192.168.0.0/16"/>
<up/>
<multicast/>
<not>
<point-to-point/>
</not>
</interface>
</interfaces>


E um ambiente LinuxTM, os usuários poderão definir uma interface para uma placa de rede específica:

<interface name="internal">
<nic name="eth1"/>
</interface>



OFFSET

[student@workstation standalone]$ sudo -u jboss \
/opt/jboss-eap-7.0/bin/standalone.sh \
-Djboss.server.base.dir=/opt/jboss/standalone2/ \
-Djboss.socket.binding.port-offset=10000


################################

CLI
https://docs.jboss.org/author/display/AS71/CLI+Recipes

# $JBOSS_HOME/bin/jboss-cli.sh
#version

To connect to a different host machine, use the --controller argument:

# $JBOSS_HOME/bin/jboss-cli.sh --connect --controller=ip:port


The following operations are very common:
* :read-resource: Reads a model resource's attribute values and either basic or complete information about any child resources.

* :read-operation-names: Gets the names of all the operations for the given resource.

* :read-operation-description: Gets the description of given operation.

* :reload: Reloads the server by shutting down all its services and starting them again. The JVM itself is not restarted.

* :read-attribute: Gets the value of an attribute for the selected resource.

* :write-attribute: Sets the value of an attribute for the selected resource.

* :remove: Removes the node.


Execução de um comando a partir de um script externo

# $JBOSS_HOME/jboss-cli.sh -c --controller=localhost:9990 \
--command="/subsystem=datasources:read-resource"

# ./jboss-cli.sh -c --controller=localhost:9990 --command="cd /subsystem=datasources,ls"

A entrada para a CLI é uma estrutura hierárquica que começa no nível do servidor
independente. Por exemplo, para visualizar as configurações no nível mais alto, digite o seguinte comando:
/:read-resource

A barra "/" é usada para separar níveis. Por exemplo, o comando a seguir lê os recursos
da interface pública:

/interface=public:read-resource

É possível usar um asterisco como curinga. Por exemplo, para ver todas as configurações
do subsistema undertow:
/subsystem=undertow/configuration=*:read-resource

O subsistema undertow é um servidor web responsável pelo processamento de páginas
da web baseadas em Java EE.


É possível visualizar todos os recursos do servidor, basta iniciar no nível superior e ative
a recursividade:

/:read-resource(recursive=true)

:read-resource(recursive=true) > /tmp/output.txt


Os dois pontos são usados para invocar uma operação, como, por exemplo, com read-
resource. Digite dois pontos e depois pressione Tab. Todas as operações disponíveis
serão exibidas para qualquer nível da operação que for executado. Por exemplo, no nível
raiz, digite dois pontos e pressione Tab para visualizar todas as operações no nível raiz:

[standalone@localhost:9990 /] :



Até agora, a CLI foi usada apenas para ler informações. No entanto, a operação write-
attribute pode ser usada para modificar os atributos de um recurso usando a CLI.
Para demonstrar essa operação, altere o atributo min-pool-size do recurso da fonte
de dados ExampleDS. A configuração da fonte de dados será tratará mais adiante, mas,
por enquanto, apenas observe que este atributo define o número mínimo de conexões
reservadas para uma fonte de dados.

[standalone@localhost:9990 /] cd /subsystem=datasources/data-source=ExampleDS
[standalone@localhost:9990 data-source=ExampleDS] :write-attribute\
(name=min-pool-size,value=5)
{"outcome" => "success"}

Use o comando :read-attribute e verifique se a alteração ocorreu:

:read-attribute \ (name=min-pool-size)


---------------
CLI ADD USERS AND MANAGEMENT USERS
Jboss root > /opt/jboss-eap/bin/add-user.sh -u "teddy" -p 'pokavergonha' -g 'dev'


[standalone@localhost:9990 access=authorization] cd /core-service=management/access=authorization
[standalone@localhost:9990 access=authorization] :write-attribute(name=provider, value=rbac)


[standalone@localhost:9990 access=authorization] :write-attribute(name=provider, value=simple)

################################

Implantação usando a CLI


* file_path: caminho do aplicativo a ser implantado.
* --url: URL onde o conteúdo da implantação está disponível para upload no repositório de
conteúdo da implantação.
* --name: o nome exclusivo da implantação. Se não for especificado um nome, o nome do
arquivo será utilizado.
* --runtime-name: opcional, define o nome do tempo de execução da implantação.
* --force: se a implantação com o nome especificado já existir, por padrão a implantação
será cancelada e a mensagem correspondente será exibida. O --force (-f) força a
substituição da implantação existente por outra especificada nos argumentos do comando.
* --disabled : indica que a implantação tem que ser adicionada ao repositório em estado
desativado.

################################
Iniciando Jboss como um controlador de domínio

$ ./domain.sh -Djboss.bind.address.management=192.168.0.14

Continuando com o mesmo exemplo, um controlador de host slave seria iniciado
para conectar-se ao master naquele endereço ao definir a propriedade do sistema
jboss.domain.master.address e também para ouvir comandos de gerenciamento
(do master) em outro endereço IP privado ao definir a propriedade do sistema
jboss.bind.address.management.

$ ./domain.sh -Djboss.bind.address.management=192.168.0.15 \
-Djboss.domain.master.address=192.168.0.14


$ ./domain.sh -Djboss.domain.base.dir=/usr/local/eap/configuration \
--domain-config=mydomain.xml --host-config=myhost.xml

$ ./domain.sh -Djboss.domain.base.dir=/usr/local/eap/configuration




CONFIGURAÇÃO DE MASTER

student@workstation ~]$ cd /home/student/JB248/labs/domain
[student@workstation domain]$ mkdir machine1
[student@workstation domain]$ cp -r /opt/jboss-eap-7.0/domain machine1/domain

Veja a linha seguir no início do arquivo host-master.xml
<host xmlns="urn:jboss:domain:4.1" name="master">

Editar a interface de gerenciamento
<interfaces>
<interface name="management">
<inet-address value="${jboss.bind.address.management:127.0.0.1}"/>
</interface>
</interfaces>


!!!No subsistema messaging-activemq do perfil full-ha, edite a tag <cluster>
(linha 1278):

<cluster password="${jboss.messaging.cluster.password:JBoss@RedHat123}" />

Inicie o controlador de domínio na VM workstation

[student@workstation domain]$ cd /opt/jboss-eap-7.0/bin

student@workstation bin]$ ./domain.sh \
-Djboss.domain.base.dir=/home/student/JB248/labs/domain/machine1/domain/ \
--host-config=host-master.xml

Essa instância de controlador de host NÃO está usando o bloco de
configuração padrão host.xml; portanto, a opção --host-config é
necessária. A opção --domain-config não é necessária porque ela ESTÁ
usando o arquivo de configuração de domínio gerenciado domain.xml
padrão


./domain.sh \
-Djboss.domain.base.dir=/opt/jboss-eap/domain/machine2/domain/ \
--host-config=host-slave.xml \
-Djboss.domain.master.address=192.168.99.106

################################
A Comparison of domain.xml and host.xml8



################################
Configuration File Backups and Snapshots


/opt/jboss-eap/standalone/configuration/standalone_xml_history
/opt/jboss-eap/standalone/configuration/standalone_xml_history/snapshot

[domain@localhost:9999 /]  /host=master:take-snapshot
[standalone@localhost:9990 /] :take-snapshot 
{
    "outcome" => "success",
    "result" => "/opt/jboss-eap/standalone/configuration/standalone_xml_history/snapshot/20200404-114515685standalone.xml"
}


[standalone@localhost:9990 /] :list-snapshots
{
    "outcome" => "success",
    "result" => {
        "directory" => "/opt/jboss-eap/standalone/configuration/standalone_xml_history/snapshot",
        "names" => [
            "20200404-114515685standalone.xml",
            "20200404-114609110standalone.xml"
        ]
    }
}



[standalone@localhost:9990 /] :delete-snapshot(name=20200404-114609110standalone.xml)
{"outcome" => "success"

################################
Deploy methods

1 - Deploy using the management CLI
- Single command-line interface with the ability to create an run deployment scripts

[]deploy /opt/apps/apps/welcome.war

Undeploy
[]undeploy welcome.war

[]deployment-info

2 - Deploying using the management console
- The benefit of a graphical interface that is easy to use

3 - Deploying using the Deployment Scanner
Monitors the deployment directory for application to deploy and scans every 5 seconds for changes

4 - Deploying using Maven
Uses Wildfly Maven Plugin for simple o

Domain controller settingsperations to deploy and undeploy applications

5 - Deploying using the HTTP API
Allows for a curl command to deploy an application


################################
EAR and WAR FILES

descompact jar

jar -xvf example.war


Compact 

jar -cvf example.war example
added manifest
..
.
.
.
.
################################
Configuring a Domain Controller
Domain controller settings

The domain controller settings are found in two locations: host.xml and domain.xml.he
domain.xml file has almost the same structure as the standalone.xml file.

The <server-groups> section is for defining and grouping servers, something that is not
possible in standalone mode.

- standalone.xml can only define a single profile. domain.xml can define any number of
different profiles. 

- domain.xml contains a <server-groups> section for defining a common setup for a group
of EAP servers and mapping them to a profile. Because a standalone server is composed of a
single server instance, the concept of server groups is nonsensical.

- Each server is managed by the host controller and not by the domain controller and each server
is declared as part of the host.xml configuration file.

################################
Configuring a managed domain with CLI

[domain@172.25.250.254:9990 /] cd host
domain@172.25.250.254:9990 host] ls
host2
host3
master
[domain@172.25.250.254:9990 /] cd /host=host2/server-config
[domain@172.25.250.254:9990 server-config] ./server-c:add\
(auto-start=true, group=main-server-group)
[domain@172.25.250.254:9990 /] /host=host2/server-config/server-c:\
remove()

Another important item on the host level is the ability to get runtime information. For example,
to get memory runtime metrics from a server, use the following commands:

[domain@172.25.250.254:9990 /] cd /host=host2/server=server-one/
[domain@172.25.250.254:9990 server=server-one] cd core-service=platform-mbean/
[domain@172.25.250.254:9990 core-service=platform-mbean] ./type=memory:\
read-attribute(name=heap-memory-usage)


ADD GROUP 

domain@172.25.250.254:9990 core-service=platform-mbean] cd /
[domain@172.25.250.254:9990 /] /server-group=production:add\
(profile=ha,socket-binding-group=ha-sockets)
[domain@172.25.250.254:9990 /] /server-group=production:remove()

To remove a server
[domain@172.25.250.254:9990 /] /host=serverb/server-config=server-two:remove()

################################
CONFIGURING SERVERS IN A MANAGED DOMAIN


Understanding hosts and servers
* Domain controller: Responsible for all configuration management using profiles.

* Host controller: Responsible for managing server instances. In a production environment, a
host controller is usually installed on a separate host (physical or VM).

* Server Groups: A logical grouping of EAP server instances that are configured and managed
together as a single unit. Server groups can span any number of servers across multiple host
controllers.

* Server: Responsible for running JavaEE applications (JAR, EAR, WAR files).

(A server cannot be part of multiple server groups.)

Configuring Server Groups
Each server group is assigned a unique profile. A profile consists of the list of EAP
subsystems and their configuration that will be used by the different subsystems that make up
the profile.

Configuring server groups using the JBoss EAP CLI
domain@workstation:9990 /] /server-group=Group1:add\
(profile=full,socket-binding-group=full-sockets)

To verify that the server groups have been created
[domain@workstation:9990 /] /server-group=Group1:read-resource

To remove a server group
[domain@workstation:9990 /] /server-group=Group1:remove()

Unlike the standalone mode, it is not possible to deploy using the deployment-scanner
subsystem. The reason for this is that is not possible to guarantee that the application will be
available for the entire group (manually deployed for each server).


Application deployment using the JBoss EAP CLI

[domain@workstation:9990 /] deploy /path/to/example.war --all-server-groups

[domain@workstation:9990 /] deploy /path/to/example.war \
--server-groups=Group1,Group2,Group3

To undeploy and remove
the application from the entire domain, the name argument and the
[domain@workstation:9990 /] undeploy example.war --all-relevant-server-groups

An explicit list of server groups
[domain@workstation:9990 /] undeploy example.war --server-groups=Group1,Group2

To keep the application in the repository
[domain@172.25.250.254:9990 /] undeploy myapp.war --all-relevant-server-groups \
--keep-content

[domain@172.25.250.254:9990 /] undeploy myapp.war --server-groups=main-server-group \
--keep-content

IMPORTANTE
If the application is assigned to two or more groups, the --keep-content argument
is required because the application content cannot be removed.

################################
Configuring Servers

Configuring and managing servers using the JBoss
EAP CLI

[domain@workstation:9990 /] /host=host2/server-config=serverA:add\
(auto-start=true,group=Group1,socket-binding-port-offset=100)

[domain@workstation:9990 /] /host=host2/server-config=serverA:start
[domain@workstation:9990 /] /host=host2/server-config=serverA:stop
[domain@workstation:9990 /] /host=host2/server-config=serverA:remove

[domain@workstation:9990 /] /host=host2/server-config=serverA:remove()


[domain@workstation:9990 /] /server-group=Group1:start-servers(blocking=true)

[domain@workstation:9990 /] /host=host2:shutdown(timeout=1000)

################################

Configuring JDBC Drivers

The syntax for the module add command is:
[disconnected /] module add --name=<module_name> --resources=<JDBC_Driver> --
dependencies=<library1>,< library2>,...

For example, the following command in the EAP CLI will create the JDBC MySQL 5.5 module by
using the file mysql-connector-java.jar that was downloaded from the database vendor:

[disconnected /] module add --name=com.mysql \
--resources=/opt/jboss-eap-7.0/bin/mysql-connector-java-5.1.38-bin.jar \
--dependencies=javax.api,javax.transaction.api


for REMOVE MODULE

[disconnected /] module remove --name=<module_name>

For the managed domain mode, the syntax is as follows:

[domain@172.25.250.254 /] /profile=default/subsystem=datasources/jdbc-driver=mysql:add(\
driver-module-name=com.mysql,driver-name=mysql\)


-------------
Use the following command to define the MySQL driver by specifying the EAP module
installed in the previous step:

[standalone@localhost:9990 /]
/subsystem=datasources\
/jdbc-driver=mysql:add(driver-name=mysql,driver-module-name=com.mysql)


[standalone@localhost:9990 /] /subsystem=datasources/jdbc-driver=mysql:read-resource


Test connection
[standalone@localhost /] /subsystem=datasources/data-source=datasource_name:test-connection-in-pool


-ADD DATASOURCE MYSQL STANDALONE

data-source add --name=MySqlDS2 --jndi-name=java:jboss/MySqlDS \
--driver-name=mysql \
--connection-url=jdbc:mysql://localhost:3306/bksecurity \
--user-name=root --password=pokavergonha \
--validate-on-match=true --background-validation=false \
--valid-connection-checker-class-name=\
org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLValidConnectionChecker \
--exception-sorter-class-name=\
org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLExceptionSorter


-ADD DATASOURCE MYSQL DOMAIN MODE

[domain@localhost] data-source add --name=MySqlDS --profile=full-ha \ --jndi-
name=java:jboss/MySqlDS --driver-name=mysql \
--connection-url=jdbc:mysql://localhost:3306/jbossdb \
--user-name=admin --password=admin \
--validate-on-match=true --background-validation=false \
--valid-connection-checker-class-name=\
org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLValidConnectionChecker \
--exception-sorter-class-name=\
org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLExceptionSorter



- ADD DATASOURCE XA

xa-data-source add --name=bookstorexa \
--jndi-name=java:jboss/datasources/bookstorexa --driver-name=mysql \
--xa-datasource-class=com.mysql.jdbc.jdbc2.optional.MysqlXADataSource \
--user-name=bookstore --password=redhat \
--xa-datasource-properties=[{"ServerName"=>"localhost"},\
{"DatabaseName"=>"bookstore"}]

################################

CONFIGURING THE LOGGING
SUBSYSTEM


The EAP logging subsystem has three main components:

Handlers: a handler determines "where" and "how" an event will be logged. EAP comes
with several handlers out-of-the-box that can be used to log messages from applications.

Loggers: a logger (also called log category) organizes events into logically related
categories. A category usually maps to a certain package namespace that is running within
the Java Virtual Machine (Could be both EAP as well as custom application package names)
and defines one or more handlers that will process and log the messages.

Root Logger: Loggers are organized in a parent-child hierarchy and the root logger resides at the top of the logger hierarchy.

Suffix*: 
- Representa o rotacionamento do log
a cada hora: yyyy-MM-dd.HH
a cada mês: yyyy-MM
a cada dia: yyyy-MM-dd


configuração de registro em log assíncrono

[standalone@localhost:9990 /] /subsystem=logging\
/size-rotating-file-handler=FILE_BY_SIZE:read-resource

################################
Configure Journals and Other Settings

Os diários têm as seguintes propriedades:
* O diário consiste em um conjunto de arquivos no sistema de arquivos de sua máquina.
* Cada arquivo é criado com um tamanho fixo e recebe preenchimento para maximizar o
desempenho de acesso ao disco.
* As mensagens são acrescentadas ao diário. O conjunto de arquivos de diário é lido e gravado como um buffer de anéis para que o espaço de mensagens já consumidas seja
reutilizado por novas mensagens publicadas.
* Quando um arquivo de diário está cheio, o ActiveMQ passa para o seguinte. Se não houver arquivos de diário vazios restantes, o ActiveMQ criará um novo.
* O ActiveMQ também tem um algoritmo de compactação que recupera o espaço usado por
mensagens já consumidas e elimina a fragmentação nos diários de mensagens. Isso pode
deixar alguns arquivos de diário vazios e elegíveis para remoção.
* O diário também tem suporte completo a operações transacionais se necessário,
suportando tanto transações locais como XA


Tratamento de falhas de entrega de mensagens
Os objetos address-setting do ActiveMQ permitem ao administrador definir:
* Um tempo de atraso para entregar novamente uma mensagem com falha, na esperança de
que o problema não ocorra na próxima tentativa, usando o atributo redelivery-delay.

* Um destino para encaminhamento da mensagem após um número de falhas de tentativa
de entrega, usando os atributos max-delivery-attempts e dead-letter-address.

As configurações padrão de messaging-activemq incluem a seguinte definição de
endereço:

[standalone@localhost:9990 server=default] ./address-setting=#:read-resource
{
"outcome" => "success",
"result" => {
...
"dead-letter-address" => "jms.queue.DLQ",
...
"max-delivery-attempts" => 10,
...
"redel

Isso significa que, para todos os destinos (o nome do objeto de address-setting é #),
as mensagens que falharem em serem consumidas após 10 tentativas (max-delivery-
attempts) serão movidas para o destino jms.queue.DLQ (dead-letter-address). Há zero
atraso entre tentativas de entrega (redelivery-delay).

para ter um atraso de cinco segundos entre tentativas de entrega do destino
jms-queue=MyQueue e a transferência das mensagens para jms-queue=MyDLQ após três
tentativas, use o seguinte comando:

[standalone@localhost:9990 server=default] ./address-setting=jms.queue.MyQueue:add(\
redelivery-delay=5000, dead-letter-address=jms.queue.MyDLQ, max-delivery-attempts=3)
################################
CHAPTER 8 
JMS

Um aplicativo pode fazer referência a um ConnectionFactory do JMS por meio de um
nome JNDI que talvez não exista. Assim, o administrador deve criar um novo recurso
pooled-connection-factory no mesmo JNDI fazendo referência ao conector correto.
Por exemplo, para usar o nome JNDI java:/MyCF e apontar para o servidor de mensagens
integrado, adicione o seguinte ao subsistema messaging-activemq:

<pooled-connection-factory name="mycf" transaction="xa" connectors="in-vm"
entries="java:/MyCF"/>

O comando da CLI a seguir cria o recurso a partir da listagem anterior no perfil full para um domínio gerenciado do EAP 7:

domain@localhost:9990 /] cd /profile=full/subsystem=messaging-activemq/server=default
[domain@localhost:9990 server=default] ./pooled-connection-factory=mycf:add(\
connectors=[in-vm], entries=[java:/jms/MyCF])


Para criar um destino usando a CLI, crie um recurso jms-queue ou jms-topic. Por exemplo,os comandos a seguir criam um tópico JMS chamado stocks vinculado ao nome JNDI java:/jms/broker/StockUpdates:

[domain@localhost:9990 server=default] ./jms-topic=stocks:add(\
entries=[java:/jms/broker/StockUpdates])

O comando anterior adiciona o seguinte aos arquivos de configuração do modo de domínio:
<jms-topic name="stocks" entries="java:/jms/broker/StockUpdates"/>

Monitoração de destinos JSM
No modo servidor standalone, as operações e os atributos de monitoramento ficam
disponíveis no mesmo objeto que define o recurso como parte do perfil do servidor. Porexemplo, para verificar o número de mensagens aguardando para serem consumidas no
jms-queue chamado MyQueue, use o comando a seguir para ler o atributo message-count

[standalone@localhost:9990 /] cd /subsystem=messaging-activemq/server=default
[standalone@localhost:9990 server=default] ./jms-queue=MyQueue:read-attribute(\
name=message-count)


No modo do servidor de domínio gerenciado, os mesmos atributos e operações estão
disponíveis em uma instância do servidor em execução e NÃO na configuração do
subsistema. Por exemple, para listar as mensagens aguardando para serem consumidas no jms-queue chamado MyQueue a partir do servidor srv1 no host devhost, use o comando a seguir para invocar a operação list-messages:

[domain@localhost:9990 /] cd /host=devhost/server=srv1
[domain@localhost:9990 server=srv1] cd ./subsystem=messaging-activemq/server=default
[domain@localhost:9990 server=default] ./jms-queue=MyQueue:list-messages



################################
CHAPTER 9
SECURING JBOSS EAP


Configuring a Database Security Domain (and Guided
Exercise)

The EAP security subsystem consists of a number of security-domains, which define
how applications are authenticated and authorized. Users can define their own security
domain backed by a system that stores identity information (database, LDAP, and so on) and
configure applications to use the newly defined security-domain in the application's deployment
descriptors.

Security domains
A security domain is a set of Java Authentication and Authorization Service (JAAS) security
configurations that one or more applications use to control authentication, authorization,
auditing, and mapping.

By default, EAP has four security domains defined:

jboss-ejb-policy: The default authorization security domain for EJB modules in applications
that have not defined a security-domain explicitly.

jboss-web-policy: The default authorization security domain for web modules in applications
that have not defined a security-domain explicitly.

other: This security domain is used internally within JBoss EAP for authorization and is
required for correct operation.

jaspitest: The jaspitest security domain is a simple Java Authentication Service Provider
Interface for Containers (JASPIC) security domain included for development purposes.

--- Configuring a security domain in an application
In order to utilize a security domain, users need to add a reference to the security domain in the
jboss-web.xml. The following is an example of the <security-domain> tag in the jboss-
web.xml:
<security-domain>Security_Domain_Name</security-domain>

The web.xml file allows users to configure the security of the application, by creating security
restrictions to specific paths, restricting based on roles, and defining the authentication type.

<security-constraint>
    <web-resource-collection>
        <web-resource-name>All resources</web-resource-name>
        <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>*</role-name>
    </auth-constraint>
</security-constraint>

The <login-config> tag within the web.xml defines the type of authentication to be used.

<login-config>
    <auth-method>BASIC</auth-method>
</login-config>


Steps:
- Configuring a data source
- Authentication Module.
    /subsystem=security/security-domain\
    =bksecurity/authentication\
    =classic:add
- Create a login module that will connect to the datasource created in the previous step.
[standalone@localhost:9990] /subsystem=security/security-domain=bksecurity/\
authentication=classic/login-module=database:add( \
code=Database, \
flag=required, \
module-options=[ \
("dsJndiName"=>"java:jboss/datasources/bksecurity-ds"), \
("principalsQuery"=>"select password from users where username=?"), \
("rolesQuery"=>"select role, 'Roles' from roles where username=?"), \
("hashAlgorithm"=>"SHA-256"), \
("hashEncoding"=>"base64") \
])



CLI Commands for Adding the Database Login Module

/subsystem=security/security-domain=testDB:add

/subsystem=security/security-domain=testDB/authentication=classic:add

/subsystem=security/security-domain=testDB/authentication=classic/login-module=Database:add(code=Database,flag=required,module-options=[("dsJndiName"=>"java:/MyDatabaseDS"),("principalsQuery"=>"select passwd from Users where username=?"),("rolesQuery"=>"select role, 'Roles' from UserRoles where username=?")])

reload


--------------------------
CHAPTER 9
SECURING JBOSS EAP

Configuring an LDAP Security Domain


# Install openldap
yum -y install openldap-servers openldap-clients

systemctl start slapd.service
systemctl enable slapd.service

# Customize openldap configuration
curl -so /etc/openldap/slapd.conf-jboss http://materials.example.com/ldap/slapd.conf-jboss
rm -rf /etc/openldap/certs/*
mv /etc/openldap/slapd.d /etc/openldap/slapd.d-rhtorig
ln -s /etc/openldap/slapd.conf-jboss /etc/openldap/slapd.conf
chown root:ldap /etc/openldap/slapd.conf-jboss
chmod 640 /etc/openldap/slapd.conf-jboss
systemctl restart slapd.service


# Load the LDAP database
cd /tmp
curl -sO http://materials.example.com/ldap/load.ldif
/usr/bin/ldapadd -x -D "cn=Manager,dc=redhat,dc=com" -w 43etq5 -f load.ldif 


/subsystem=security/security-domain=jb248_ldap:add(cache-type=default)

/subsystem=security/security-domain=jb248_ldap/authentication=classic:add


/subsystem=security/security-domain=jb248_ldap/authentication=classic/login-module=ldap_login:add(code=Ldap, flag=required, module-options=[("java.naming.factory.initial"=>"com.sun.jndi.ldap.LdapCtxFactory"),("java.naming.provider.url"=>"ldap://localhost:389"), ("java.naming.security.authentication"=>"simple"), ("principalDNPrefix"=>"uid="), ("principalDNSuffix"=>",ou=people, dc=redhat, dc=com"), ("rolesCtxDN"=>"ou=Roles, dc=redhat, dc=com"), ("uidAttributeID"=>"member"), ("matchOnUserDN"=>"true"), ("roleAttributeID"=>"cn"), ("roleAttributeIsDN"=>"false")])




--------------------------
SECURING JBOSS EAP

Configuring the Password Vault
The vault encrypts sensitive strings, stores them in an encrypted keystore, and decrypts them
for applications and verification systems. To avoid leaving sensitive credentials readable in the
standalone.xml or domain.xml files, users can store passwords or other attributes in the
vault and then reference the vault from within the server configuration file. Using a vault creates
a level of abstraction and obfuscates data which could otherwise be read by anyone who has
access to the configuration files.

1.Create a Java keystore.
2. Initialize the EAP vault with the keystore.
3. Store the sensitive information in the vault.
4. Update the server configuration to include the vault information.
5. Reference the stored attribute in the server configuration file.


* alias: The alias is a unique identifier for the vault or other data stored in the keystore.
* keyalg: The algorithm to use for encryption.
* storetype: The keystore type. jceks is the recommended type.
* keysize The size of the encryption key which determines the difficulty in brute forcing the
key.
* keystore: The file path and file name in which the keystore's values are stored.

1
[student@workstation ~]$ keytool -genseckey -alias vault \
-keyalg AES -storetype jceks -keysize 128 \
-keystore /home/student/vault.keystore

$ keytool -genseckey -alias vault -storetype jceks -keyalg AES -keysize 128 -storepass vault22 -keypass vault22 -validity 730 -keystore vault.keystore


2
[student@workstation ~]$ cd /opt/jboss-eap-7.0/bin
[student@workstation bin]$ ./vault.sh


JAVA_OPTS="-Djboss.modules.system.pkgs=com.sun.crypto.provider"
$ JAVA_OPTS="-Djboss.modules.system.pkgs=com.sun.crypto.provider" ../vault.sh --keystore /opt/jboss-eap/bin/vault/vault.keystore --keystore-password vault22 --alias vault --vault-block bksecurity --attribute password --sec-attr redhat --enc-dir /opt/jboss-eap/bin/vault/ --iteration 120 --salt 1234abcd


********************************************
Vault Block:bksecurity-ds
Attribute Name:password
Configuration should be done as follows:
VAULT::bksecurity-ds::password::1
********************************************
WFLYSEC0048: Vault Configuration in WildFly configuration file:
********************************************
...
</extensions>
<vault>
  <vault-option name="KEYSTORE_URL" value="/opt/jboss-eap/bin/vault/vault.keystore"/>
  <vault-option name="KEYSTORE_PASSWORD" value="MASK-6aNfa/L4PVS"/>
  <vault-option name="KEYSTORE_ALIAS" value="vault"/>
  <vault-option name="SALT" value="12345678"/>
  <vault-option name="ITERATION_COUNT" value="120"/>
  <vault-option name="ENC_FILE_DIR" value="/opt/jboss-eap/bin/vault/"/>
</vault><management> ...
********************************************

DATA SOURCE POSTGRESQL

yum install postgresql postgresql-server -y 

systemctl initdb postgresql

su -u postgres

psql -U postgres 

create user admin with password 'admin123';
create database emp owner admin;

\c emp
create table customer(custid int, fullname varchar(100), phone char(10));
insert into customer(custid, fullname,phone) values(001,'LuizEduardo','1202017');

grant all privileges on table customer to admin;
grant all privileges on database emp to admin;


select * from customer;

SHOW hba_file;

vim /var/lig/ṕgsql/data/postgresql.conf

netstat -vatuln| grep 5432



module add --name=com.postgres --resources=/opt/jboss/bin/postgresql.jar --dependencies=javax.api,javax.transaction.api


/subsystem=datasources/data-source=DATASOURCE_NAME:test-connection-in-pool



/subsystem=datasources/jdbc-driver=postgresql:add(driver-name=postgresql,driver-module-name=com.postgresql,driver-xa-datasource-class-name=org.postgresql.xa.PGXADataSource)


data-source add --profile=default --name=PotgresDs --jndi-name=java:jboss/datasources/PotgresDs --driver-name=postgresql --connection-url=jdbc:postgresql://192.168.99.110:5432/emp --enabled=true --user-name=postgres --password=postgres --min-pool-size=5 --max-pool-size=30 --use-java-context=true




xa-data-source add --name=PostgresXADS --jndi-name=java:jboss/PostgresXADS --driver-name=postgresql --user-name=admin --password=admin --validate-on-match=true --background-validation=false --valid-connection-checker-class-name=org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLValidConnectionChecker --exception-sorter-class-name=org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLExceptionSorter --xa-datasource-properties={"ServerName"=>"localhost","PortNumber"=>"5432","DatabaseName"=>"postgresdb"}






################################
CHAPTER 10
CONFIGURING THE JAVA
VIRTUAL MACHINE

runtime> server groups> viwe > jvm 
or
/opt/jboss-eap/bin/standalone.conf
/opt/jboss-eap/bin/domain.conf
OR

- Adicionando JVM para uma instância

[domain@192.168.99.110:9990 /] /host=serverb/server-config=server-one/jvm=server-one-jmv:add(heap-size=1200,max-heap-size=1200m,jvm-options=["-XX:+AggressiveOpts"])

- restart 

[domain@192.168.99.110:9990 /] /host=serverb/server-config=server-one/:restart(blocking=true)




################################
CHAPTER 11
Configure subsystens


[standalone@localhost:9990 /] deploy \
/home/student/JB248/labs/root-web/welcome.war

Open the web browser on the workstation and navigate to http://localhost:8080/
welcome to test if the application was deployed successfully. You should see the welcome application:

-### Setting the root web location

[standalone@localhost:9990 /] /subsystem=undertow/server=\
default-server/host=default-host/location=\/:remove

The previous command will remove the / location from the default-host. The / location is the root application.

[standalone@localhost:9990 /] reload

[standalone@localhost:9990 /] /subsystem=undertow/server=\
default-server/host=default-host:\
write-attribute(name=default-web-module,value=welcome.war)

[standalone@localhost:9990 /] reload


-### Configure server with listeners
The host configuration within the server element is responsible for creating virtual hosts. 

Virtual hosts groups web applications based on the DNS names. Each server can configure a set of hosts:

/subsystem=undertow/server=default-server/host=\
version-host:add(alias=[version.myapp.com])

/subsystem=undertow/server=default-server/host=\
version-host:write-atribute(name=default-web-module, value=version.war)

in the previous example, requests to http://version.myapp.com will by default serve the version.war application. The following XML will reflect the previous modification:

If the new host is no longer required, delete it with:
/subsystem=undertow/server=default-server/host=\
version-host:remove()


## Configure SSL connections
To create a new keystore within a self-signed certificate, use the following command:

`keytool -genkeypair -alias appserver -storetype jks -keyalg RSA -keysize 2048  \ -keystore /path/identity.jks`

`keytool -genkeypair -alias appserver -storetype jks -keyalg RSA -keysize 2048 -keypass password1 -keystore /opt/standalone/configuration/identity.jks -storepass password1 -dname "CN=appserver,OU=Sales,O=Systems Inc,L=Raleigh,ST=NC,C=US" -validity 730 -v`


The following parameters were specified:
* genkeypair: Responsible for defining that a new self-signed certificate should be created.
* alias: Defines the certificate name inside the keystore.
* storetype: Defines the type of the keystore.
* keyalg: Specifies the algorithm to be used to generate the certificate.
* keysize: Specifies the size of each key to be generated.
* keystore: Defines where the keystore should be saved.

The previous command requests two passwords. The first is the password to open the identity keystore and the second is to access the appserver certificate. Use the same value for both passwords to enable the HTTPS protocol in EAP 7

- After creating the keystore, a new security realm should be created for each host:

`/host=servera/core-service=management/security-realm=HTTPSRealm:add()`

After adding the security realm, it should be configured for loading the created keystore:

/host=servera/core-service=management/security-realm=HTTPSRealm/server-identity=
 ssl:add(keystore-path=/opt/identity.jks,keystore-password=changeit,alias=appserver)

A reload is required for the changes to take effect.

/:reload-severs

To enable the HTTPS protocol, a new HTTPS listener must be created on the default server. This listener must use the created security realm:

/profile=full/subsystem=undertow/server=default-server/https-listener=https:\
add(socket-binding=https,security-realm=HTTPSRealm)


- Enable the AJP protocol
standalone@localhost:9990 /] /subsystem=undertow/server=\
default-server/ajp-listener=ajp:add(socket-binding=ajp)

[standalone@localhost:9990 /] /subsystem=undertow/server=\
default-server/ajp-listener=ajp:read-resource


[standalone@localhost:9990 /] /subsystem=undertow/server=\
default-server/http-listener=default:\
write-attribute(name=max-connections, value=200)


- Other components, One of the Undertow components is the buffer cache.

<subsystem xmlns="urn:jboss:domain:undertow:3.0">
<buffer-cache name="default" buffer-size="1024" buffers-per-region="1024" max-
regions="10"
...
</subsystem>

It is possible to update the buffer size using the CLI:

/subsystem=undertow/buffer-cache=default:write-attribute name=buffer-size,value=2048)

If a cache is not required, it can be removed:
/subsystem=undertow/buffer-cache=default:remove

/subsystem=undertow/servlet-container=default:read-resource

To set the default session timeout to one hour, use the following operation:
/subsystem=undertow/servlet-container=default:write-attribute(name=default-session-
timeout,value=60)

To create a new file handler, use the following CLI command:
/subsystem=undertow/configuration=handler/file=photos\
:add(path=/var/photos, directory-listing=true)

Attach the new file handler to a location in a virtual host:
/subsystem=undertow/server=default-server/host=default-host/location=photos\:add(handler=photos)

This content will be served using the ip:port/context/photos.


-### The I/O subsystem 
/subsystem=undertow/server=default-server/http-listener=\
default:read-attribute(name=worker)

The worker named default is defined in the I/O subsystem. Use the following command to list the attributes from the worker:
/subsystem=io/worker=default:read-resource


################################
CHAPTER 12
DEPLOYING CLUSTERED APPLICATIONS

-----
CLUSTERED APPLICATIONS
sudo -u jboss-eap /opt/jboss-eap/bin/standalone.sh -Djboss.server.base.dir=/opt/jboss-eap/cluster1 -Djboss.bind.address=192.168.99.110 -Djboss.bind.address.management=192.168.99.110 -Djboss.socket.binding.port-offset=150 -Djboss.node.name=cluster1 --server-config=standalone-full-ha.xml

/opt/apps/materials/solutions/clustering-jgroups/new-tcp-stack.cli

sudo -u jboss-eap /opt/jboss-eap/bin/standalone.sh -Djboss.base.server.dir=/opt/jboss-eap/cluster1 --server-config=standalone-full-ha.xml -Djboss.bind.address=192.168.99.110 -Djboss.socker.bind.port-offset=100 -Djboss.node.name=jgroups-cluster1 -Djboss.bind.address.private=192.168.99.110
-----

-----
Configuring Load Balancer

Configuring Undertow as a dynamic load balancer


To add the mod_cluster filter and configure it to default EAP settings, use the following
command:
/profile=ha/subsystem=undertow/configuration=filter/mod_cluster=lb:add(\
management-socket-binding=http, advertise-socket-binding=modcluster)

* management-socket-binding: informs Undertow where to receive connection information
and load balance metrics from back-end web servers. It should point to the socket-binding
where EAP receives HTTP requests, which is http by default.

* advertise-socket-binding informs Undertow where to send advertisement messages,
that is, the multicast address and UDP port, by referring to a socket-binding name.

After creating the filter, it should be enabled in the desired Undertow (virtual) hosts:

/profile=ha/subsystem=undertow/server=default-server/host=default-host/filter-ref=lb:add

To configure the advertise key on the application server instances, that is, on the cluster
members:

/profile=ha/subsystem=modcluster/mod_cluster-config=configuration:\
write-attribute(name=advertise-security-key,value=secret)

To configure the key on the load balancer server instance:

/profile=ha/subsystem=undertow/configuration=filter/mod_cluster=lb:\
write-attribute(name=security-key,value=secret)


- Disable Advertise and Configure a Proxy List

/profile=ha/subsystem=undertow/configuration=filter/mod_cluster=lb:\
write-attribute(name=advertise-frequency,value=0)

/profile=ha/subsystem=undertow/configuration=filter/mod_cluster=lb:\
write-attribute(name=advertise-socket-binding,value=null)

/profile=ha/subsystem=modcluster/mod_cluster-config=configuration:\
write-attribute(name=advertise,value=false)

Cluster members now require a proxy list to know which mod_cluster load balancer they should
send connection parameters and application status to. Each load-balancer instance has to be
configured as an outbound socket binding. Assuming a single load balancer instance:

/socket-binding-group=ha-sockets/remote-destination-outbound-socket-binding=lb:\
add(host=10.1.2.3, port=8080)

These socket bindings are then used to configure the proxies list on the modcluster
subsystem:

/profile=ha/subsystem=modcluster/mod_cluster-config=configuration:\
write-attribute(name=proxies,value=[lb])


- Configuring Undertow as a static load balancer

Each cluster member IP address and AJP port has to be configured as an outbound socket
binding. Assuming there are two cluster members:

/socket-binding-group=ha-sockets/remote-destination-outbound-socket-binding=\
cluster-member2/:add(host=10.1.2.13, port=8009)

/socket-binding-group=ha-sockets/remote-destination-outbound-socket-binding=\
cluster-member1/:add(host=10.1.2.3, port=8009)

The reverse-proxy handler is created in the undertow subsystem
/profile=ha/subsystem=undertow/configuration=handler/reverse-proxy=lb:add

Each cluster member is added as a route to the reverse-proxy handler by using a host child:
/profile=ha/subsystem=undertow/configuration=handler/reverse-proxy=lb/host=member1:\
add(outbound-socket-binding=cluster-member1,scheme=ajp,instance-id=hosta:server1,\
path=/app)


/profile=ha/subsystem=undertow/configuration=handler/reverse-proxy=lb/host=member2:\
add(outbound-socket-binding=cluster-member2,scheme=ajp,instance-id=hostb:server2,\
path=/app)


Enable the reverse-proxy handler on the desired Undertow (virtual) hosts:

/profile=ha/subsystem=undertow/server=default-server/host=default-host/\
location=\/app:add(handler=lb)

- Configuring the LB ---
1 - Crie um proxy reverso chamado cluster-handler:
[standalone@127.0.0.1:9090 /] /subsystem=undertow/configuration=\
handler/reverse-proxy=cluster-handler:add

2 - Crie uma nova ligação de soquete que será usada pelo protocolo A JP em execução
no servidor chamado server1 (172.25.250.254:8109). Dê a ele o nome remote-
server1:
[standalone@127.0.0.1:9090 /] /socket-binding-group=standard-sockets\
/remote-destination-outbound-socket-binding=remote-server1\
:add(host=172.25.250.254, port=8109)

3 - Crie no Undertow uma referência a uma ligação de soquete anteriormente criada no
server1 e vincule-a ao esquema A JP; isso deverá acessar o caminho de contexto do
cluster:
[standalone@127.0.0.1:9090 /] /subsystem=undertow/configuration=handler\
/reverse-proxy=cluster-handler/host=server1\
:add(outbound-socket-binding=remote-server1, scheme=ajp, \
instance-id=server1, path=/cluster)

4 - Crie um novo local de proxy reverso chamado /cluster fazendo referência ao
manipulador cluster-handler:

[standalone@127.0.0.1:9090 /] /subsystem=undertow/server=default-server/\
host=default-host\
/location=\/cluster:add(handler=cluster-handler)

