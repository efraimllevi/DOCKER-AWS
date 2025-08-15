# DOCKER-AWS

Documentação para Implementação do
WordPress com Docker Compose na AWS EC2
1. Introdução
Esta documentação descreve os passos para configurar um ambiente de WordPress com
MySQL utilizando Docker Compose em uma instância EC2 da AWS. O objetivo é automatizar a
instalação do Docker, configurar containers para o WordPress e o banco de dados MySQL, e
configurar os serviços adicionais como EFS e Load Balancer para o ambiente WordPress.
2. Pré-requisitos
Antes de começar a instalação, os seguintes pré-requisitos são necessários:
Uma instância EC2 rodando no Ubuntu ou Amazon Linux.
Acesso SSH à instância EC2.
Permissões para criar e gerenciar serviços na AWS, como RDS e EFS.
Docker e Docker Compose instalados na instância EC2.
Passos para instalar o Docker e Docker Compose:
2.1. Instalar o Docker
Execute os seguintes comandos para instalar o Docker na instância EC2:
bash
Copiar
# Atualizar pacotes e instalar dependências
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
# Adicionar a chave do Docker ao sistema
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# Adicionar o repositório Docker
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu
$(lsb_release -cs) stable"
# Atualizar novamente os pacotes e instalar o Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
# Verificar se o Docker foi instalado corretamente
sudo docker --version
2.2. Instalar o Docker Compose
Para instalar o Docker Compose, execute os seguintes comandos:
bash
Copiar
# Baixar a versão mais recente do Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/dockercompose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# Definir permissões de execução
sudo chmod +x /usr/local/bin/docker-compose
# Verificar a instalação
docker-compose --version
3. Configuração do WordPress com Docker Compose
3.1. Criar o arquivo docker-compose.yml
Crie um diretório para o projeto WordPress e dentro dele, crie o arquivo docker-compose.yml:
bash
Copiar
# Criar diretório
mkdir ~/wordpress-docker
cd ~/wordpress-docker
# Criar o arquivo docker-compose.yml
nano docker-compose.yml
Adicione o seguinte conteúdo ao arquivo docker-compose.yml:
yaml
Copiar
version: '3.8'
services:
 wordpress:
 image: wordpress:latest
 container_name: wordpress
 ports:
 - "8080:80"
 environment:
 WORDPRESS_DB_HOST: mysql-db:3306
 WORDPRESS_DB_NAME: wordpress
 WORDPRESS_DB_USER: root
 WORDPRESS_DB_PASSWORD: example
 volumes:
 - wordpress_data:/var/www/html
 networks:
 - wordpress_network
 mysql-db:
 image: mysql:5.7
 container_name: mysql-db
 environment:
 MYSQL_ROOT_PASSWORD: example
 MYSQL_DATABASE: wordpress
 volumes:
 - mysql_data:/var/lib/mysql
 networks:
 - wordpress_network
volumes:
 wordpress_data:
 driver: local
 mysql_data:
 driver: local
networks:
 wordpress_network:
 driver: bridge
3.2. Rodar o Docker Compose
Execute o comando abaixo para iniciar o Docker Compose e criar os containers para WordPress
e MySQL:
bash
Copiar
docker-compose up -d
1. 
2. 
3. 
Este comando irá:
Baixar as imagens necessárias.
Criar e iniciar os containers.
Expor o WordPress na porta 8080.
3.3. Acessar o WordPress
Após os containers estarem em execução, você pode acessar o WordPress pelo navegador:
text
Copiar
http://<IP-da-instância-EC2>:8080
Na primeira vez que você acessar, será exibida a tela de configuração do WordPress.
4. Configuração do EFS (Elastic File System)
O EFS será utilizado para armazenar dados estáticos do WordPress. Siga as instruções abaixo
para configurar o EFS.
4.1. Criar um sistema de arquivos EFS
Acesse o console da AWS e vá até o serviço EFS.
Crie um sistema de arquivos EFS com as configurações padrão.
Certifique-se de que a instância EC2 tenha permissão para acessar o EFS.
4.2. Montar o EFS na instância EC2
No seu EC2, instale o pacote nfs-common para poder montar o EFS:
bash
Copiar
sudo apt-get install -y nfs-common
Crie o diretório de montagem e monte o EFS:
bash
Copiar
sudo mkdir -p /mnt/efs
sudo mount -t nfs -o vers=4.1 <efs-dns-name>:/ /mnt/efs
Substitua <efs-dns-name> pelo nome DNS do seu EFS.
1. 
2. 
3. 
4.3. Atualizar o arquivo docker-compose.yml
Para que o WordPress utilize o EFS para armazenar arquivos estáticos, modifique o dockercompose.yml para adicionar o volume montado:
yaml
Copiar
 wordpress:
 image: wordpress:latest
 container_name: wordpress
 ports:
 - "8080:80"
 environment:
 WORDPRESS_DB_HOST: mysql-db:3306
 WORDPRESS_DB_NAME: wordpress
 WORDPRESS_DB_USER: root
 WORDPRESS_DB_PASSWORD: example
 volumes:
 - /mnt/efs:/var/www/html/wp-content
 networks:
 - wordpress_network
Isso vai permitir que o conteúdo estático do WordPress (como imagens e uploads) seja
armazenado diretamente no EFS.
5. Configuração do Load Balancer
Para evitar exposição direta ao público, utilize um Load Balancer para gerenciar o tráfego para
o WordPress.
5.1. Criar um Load Balancer
No console da AWS, crie um Classic Load Balancer.
Selecione as instâncias EC2 para o balanceamento de carga.
Certifique-se de que o Load Balancer esteja configurado para aceitar tráfego na porta 80.
6. Conclusão
Você agora tem uma instância EC2 rodando WordPress e MySQL, utilizando Docker e Docker
Compose. O WordPress está acessível via Load Balancer e utiliza o EFS para armazenar arquivos
estáticos.
bash
Copiar
docker-compose down
Referências:
Docker Documentation
WordPress Docker Image
AWS Elastic File System (EFS)
