Atividade AWS - Docker - DevSecOps Compass UOL
Este repositório tem como objetivo documentar as etapas que realizei para a execução da atividade de AWS - Docker do Programa de Bolsas AWS e DevSecOps - Compass UOL.

Requisitos da atividade:
Instalação e configuração do DOCKER ou CONTAINERD no host EC2.
Ponto adicional para o trabalho: Utilize a instalação via script de Start Instance (user_data.sh).
Efetuar implantar uma aplicação WordPress com contêiner de aplicação RDS banco de dados MySQL.
Configuração da utilização do serviço EFS AWS para estáticos do container de aplicação WordPress.
Configuração do serviço de Load Balancer AWS para aplicação WordPress.
Pontos de atenção:
Não utilize IP público para saída dos serviços WordPress (Evite publicar o serviço WordPress via IP público).
Sugestão para o tráfego: Internet sair pelo LB (Load Balancer Classic).
Pastas públicas e estáticas do WordPress sugestão de uso do EFS (Elastic File System).
Fica a classificação de cada membro que usa Dockerfile ou Docker Compose.
Necessário demonstrar a aplicação WordPress funcionando (tela de login).
A aplicação WordPress precisa estar rodando na porta 80 ou 8080.
Utilize o repositório git para versionamento.
Criar documentação.
Etapas de execução
Configuração da Rede:
Acesse o console AWS e entrei no serviço VPC .
No menu lateral esquerdo, na seção Virtual private cloud selecionei Your VPCs .
Dentro de Your VPCs cliquei no botão Create VPC .
Alterei as seguintes configurações:
Em Recursos para criar seleçãoi VPC e muito mais .
Em Geração automática de name tag coloquei o nome "docker-vpc".
Em Número de Zonas de Disponibilidade (AZs) selecionei 2 .
Em gateways NAT selecionei In 1 AZ .
Em VPC endpoints selecione None .
Clique em Criar VPC .
Visualização


Configuração dos Grupos de Segurança:
Acessei o console AWS e entrei no serviço EC2 .

No menu lateral esquerdo, na seção Rede e Segurança , selecione Grupos de Segurança .

Dentro de Grupos de segurança , clique no botão Criar grupo de segurança .

Criei e configurei os seguintes grupos de segurança usando uma VPC criada anteriormente:

Balanceador de carga – regras de entrada
Tipo	Protocolo	Faixa de portas	Fonte
HTTP	TCP	80	0.0.0.0/0
Servidor Web EC2 – Regras de entrada
Tipo	Protocolo	Faixa de portas	Fonte
SSH	TCP	22	EC2 GELO
HTTP	TCP	80	Balanceador de carga
EC2 ICE - Regras de saída
Tipo	Protocolo	Faixa de portas	Fonte
SSH	TCP	22	Servidor Web EC2
RDS – Regras de entrada
Tipo	Protocolo	Faixa de portas	Fonte
MySQL/Aurora	TCP	3306	Servidor Web EC2
EFS – Regras de entrada
Tipo	Protocolo	Faixa de portas	Fonte
NFS	TCP	2049	Servidor Web EC2
Criando o Elastic File System:
Acessei o console AWS e entrei no serviço de EFS .

Na tela do Elastic File System cliquei no botão Create file system .

Depois cliquei no botão Customize .

Executei a seguinte configuração:

Passo 1 - Configurações do sistema de arquivos:
No campo Nome digitei "efs-docker".
Cliquei em Próximo .
Passo 2 - Acesso à rede:
No campo Virtual Private Cloud (VPC) selecionei a VPC que foi criada anteriormente.
No campo Subnet ID selecionei as sub-redes privadas de cada AZ.
No campo Grupos de segurança selecionei o grupo "EFS" que foi criado anteriormente.
Cliquei em Próximo .
Etapa 3 – opcional – Política do sistema de arquivos:
Cliquei em Próximo .
Passo 4 - Revise e crie:
Revise e clique em Criar para finalizar.
Criando o serviço de banco de dados relacional:
Acesse o console AWS e entre no serviço de RDS .
Na tela Dashboard cliquei no botão Create database .
Executei a seguinte configuração:
Na seção Opções do mecanismo selecione MySQL .
Na seção Modelos selecionei Nível gratuito .
Na seção Configurações de credenciais adicionei uma senha mestra e confirmei.
Na seção Conectividade , no campo Virtual private cloud selecionei a VPC criada anteriormente.
No campo Existing VPC security groups selecionei o grupo "RDS" que foi criado anteriormente.
Na seção Configuração adicional , no campo Nome inicial do banco de dados coloquei o nome "dockerdb".
Revise e clique em Criar banco de dados para finalizar.
Criando o Classic Load Balancer:
Acessei o console AWS e entrei no serviço EC2 .
No menu lateral esquerdo, na seção Load Balancing selecionei Load Balancers .
Dentro de Load Balancers clique no botão Create load balancer .
Em Load balancer types cliquei em Classic Load Balancer e depois em Create .
No campo Nome do balanceador de carga digitei "ws-clb".
Na seção Mapeamento de rede , no campo VPC selecionei a VPC criada anteriormente.
No campo Mapeamentos selecionei as duas AZ's e suas respectivas sub-redes públicas.
No campo de grupos de segurança selecionei o grupo "Load Balancer" que foi criado anteriormente.
Na seção Verificações de integridade , no campo Ping path adicionadoi o caminho "/wp-admin/install.php".
Clique em Criar balanceador de carga para finalizar.
Gerando um par de chaves:
Acessei o console AWS e entrei no serviço EC2 .
No menu lateral esquerdo, na seção Rede e Segurança selecione Pares de chaves .
Dentro de Key pairs cliquei no botão Create key pair .
No campo Nome digitei "MinhaChaveSSH".
No campo Tipo de par de chaves selecionei RSA .
No campo Formato de arquivo de chave privada selecionei .pem .
Clique no botão Criar par de chaves .
Salve o arquivo .pem.
Criando o modelo de lançamento:
Acessei o console AWS e entrei no serviço EC2 .
No menu lateral esquerdo, na seção Instâncias selecionei Launch Templates .
Dentro de Launch Templates cliquei no botão Create launch template .
No campo Launch template name digitei "ws-lt".
No campo Descrição da versão do modelo digitei "docker-wordpress".
Em Application and OS Images cliquei em Quick Start , depois cliquei em Amazon Linux e selecionei Amazon Linux 2023 AMI.
Na seção Tipo de instância selecionei o tipo t3.small.
No campo Nome do par de chaves selecionei o par de chaves criado anteriormente.
Em Configurações de rede , no campo Grupos de segurança selecionei o grupo "EC2 Web Server" que foi criado anteriormente.
Em Resource tags cliquei em Add new tag e adicionei as tags de Key "Name", "CostCenter" e "Project" para os Resource types Instâncias e Volumes.
Em Detalhes avançados , no campo Dados do usuário adicionadosi o script abaixo:
#!/bin/bash
#Atualizar os pacotes do sistema
sudo yum update -y

#Instalar, iniciar e configurar a inicialização automática do docker
sudo yum install docker -y 
sudo systemctl start docker
sudo systemctl enable docker

#Adicionar o usuário ec2-user ao grupo docker
sudo usermod -aG docker ec2-user

#Instalação do docker-compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

#Instalar, iniciar e configurar a inicialização automática do nfs-utils
sudo yum install nfs-utils -y
sudo systemctl start nfs-utils
sudo systemctl enable nfs-utils

#Criar a pasta onde o EFS vai ser montado
sudo mkdir /efs

#Montagem e configuração da montagem automática do EFS
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport ID-EFS:/ efs
sudo echo "ID-EFS:/ /efs nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev 0 0" >> /etc/fstab

# Criar uma pasta para os arquivos do WordPress
sudo mkdir /efs/wordpress

# Criar um arquivo docker-compose.yaml para configurar o WordPress
sudo cat <<EOL > /efs/docker-compose.yaml
version: '3.8'
services:
  wordpress:
    image: wordpress:latest
    container_name: wordpress
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: RDS-Endpoint
      WORDPRESS_DB_USER: RDS-Master username
      WORDPRESS_DB_PASSWORD: RDS-Master password
      WORDPRESS_DB_NAME: RDS-Initial database name
      WORDPRESS_TABLE_CONFIG: wp_
    volumes:
      - /efs/wordpress:/var/www/html
EOL

# Inicializar o WordPress com docker-compose
docker-compose -f /efs/docker-compose.yaml up -d
Clique em Criar modelo de lançamento para finalizar.
Criando grupos de Auto Scaling:
Acessei o console AWS e entrei no serviço EC2 .
No menu lateral esquerdo, na seção Auto Scaling selecione Auto Scaling Groups .
Dentro dos grupos de Auto Scaling, clique no botão Criar grupo de Auto Scaling .
Executei a seguinte configuração:
Passo 1 – Escolha o modelo de lançamento:
No campo Nome do grupo do Auto Scaling digitei "ws-asg".
Na seção Launch template selecionei o template criado anteriormente.
Cliquei em Próximo .
Passo 2 – Escolha as opções de inicialização da instância:
Na seção Network , no campo VPC selecionei a VPC criada anteriormente.
No campo Zonas de disponibilidade e sub-redes selecionei como duas sub-redes privadas criadas anteriormente.
Cliquei em Próximo .
Passo 3 – Configurar opções avançadas:
Na seção Balanceamento de carga selecione Anexar a um balanceador de carga existente .
Na seção Anexar a um balanceador de carga existente, clique em Escolher entre Classic Load Balancers e selecione o balanceador de carga criado anteriormente.
Na seção Verificações de integridade marque a opção Ativar verificações de integridade do Elastic Load Balancing .
Cliquei em Próximo .
Etapa 4 – Configurar o tamanho e o dimensionamento do grupo:
No campo Capacidade desejada digitei "2".
Em Scaling , no campo Capacidade mínima desejada digitei "2".
No campo Capacidade máxima desejada digitei "4".
Em Escalabilidade automática selecione a opção Política de escalabilidade de rastreamento de destino
No campo Tipo de métrica deixei selecionado Utilização média da CPU .
No campo Valor alvo digitei "75".
Cliquei em Próximo .
Etapas 5, 6 e 7:
Cliquei em Próximo .
Cliquei em Próximo .
Revise e clique em Criar grupo de Auto Scaling para finalizar.
Configuração do EC2 Instance Connect Endpoint:
Acesse o console AWS e entrei no serviço VPC .
No menu lateral esquerdo, na seção Nuvem privada virtual selecione Endpoints .
Dentro de Endpoints clique no botão Create endpoint .
Alterei as seguintes configurações:
Em Name tag coloquei o nome "ws-ep".
Na categoria de serviço selecionei EC2 Instance Connect Endpoint .
Em VPC selecionei a VPC criada anteriormente.
Em Grupos de segurança, selecionei o grupo "EC2 ICE" que foi criado anteriormente.
Em Subnet, selecionei uma sub-rede privada que foi criada anteriormente.
Clique em Criar endpoint .
Instalando o WordPress:
Acessei o nome DNS do Load Balancer através do navegador.
Na tela de instalação do WordPress , mantenha o idioma padrão e clique em Continuar .
Na tela a seguir apresentamos os dados para criação de um usuário.
Clique em Instalar WordPress para finalizar.
Testando os serviços:
Acessando a página do WordPress via Load Balancer:

Coloquei o nome DNS do Load Balancer através do navegador para acessar uma página do WordPress .
Acessando uma instância via EC2 Instance Connect Endpoint:

Configurei as credenciais da conta AWS no terminal do PowerShell .
Utilize o comando abaixo para visualizar os ID's das instâncias que estão em execução:
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running" | Select-String "InstanceId"
Copiei o ID de uma delas.
Use o comando a seguir para fazer o acesso SSH na instância via EC2 Instance Connect Endpoint passando pelo ID da instância:
 aws ec2-instance-connect ssh --instance-id <instance-id>
Testando a montagem do EFS:

Utilize o comando df -hpara verificar se o EFS está montado.
Utilize o comando cat /etc/fstabpara verificar se a configuração persistente está configurada.
Testando o docker e docker-compose:

Utilize o comando docker pspara verificar se o container wordpress está executando.
Utilize o comando abaixo para verificar se o docker-compose está funcionando:
docker-compose -f /efs/docker-compose.yaml ps
Acessando o banco de dados da aplicação WordPress:

Copiar o ID do container wordpress .
Para acessar o container executei o comando abaixo passando pelo ID do container:
docker exec -it <container-id> /bin/bash
Dentro do container utilizei o comando apt-get updatepara atualizar a lista de pacotes dos repositórios do container.
Utilize o comando abaixo para instalar o cliente mysql .
apt-get install default-mysql-client -y
Para acessar o MySQL executei o comando abaixo passando pelo endpoint, porta e usuário do RDS :
mysql -h <RDS-endpoint> -P 3306 -u <Master username> -p
Digite a senha do usuário.
Utilize o comando show databases;para listar os bancos de dados disponíveis.
Utilize o comando use dockerdbpara selecionar o banco de dados dockerdb .
Utilize o comando show tables;para listar todas as tabelas criadas dentro do banco de dados dockerdb .
Referências:
Criar uma instância de banco de dados do Amazon RDS
Crie um Classic Load Balancer com um listener HTTPS
Instalação do Docker - Linux
Instale o Compose autônomo
WordPress – Como usar esta imagem
Criar um modelo de execução para um grupo do Auto Scaling
Crie um grupo do Auto Scaling usando um modelo de execução
Usar o EC2 Instance Connect para se conectar à sua instância do Linux com uma AWS CLI
Conexão a uma instância de banco de dados executando o mecanismo de banco de dados do MySQL
Comandos básicos do MySQL no terminal
