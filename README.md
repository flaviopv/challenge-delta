Ambiente utilizado em um desktop Windows 10 Home Single Language - 64 bits:

1) Instalação do VirtualBox para Windows,
sendo o link abaixo utilizado para download 
https://download.virtualbox.org/virtualbox/6.1.4/VirtualBox-6.1.4-136177-Win.exe

2) Instalação do VirtualBox Extension Package

3) Donwload do Debian 10.3, sendo o link abaixo utilizado para Download
http://debian.c3sl.ufpr.br/debian-cd/current/amd64/iso-cd/debian-10.3.0-amd64-xfce-CD-1.iso

4) Criação de uma máquina virtual com o Debian 10.3
10 GB de Disco
1 Vcpu
3,5 GB de RAM

5) Instalação do Debian 10.3 na máquina virtual acima

6) Atualizando o Debian
  apt-get update

7) Instalando os pacotes pre-requisitos do Docke CE
  apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common

8) Adicionando o GPG oficial do Docker 
    curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -

9) Adicionando o Repositório
    add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/debian \
    $(lsb_release -cs) \
    stable"

10) Atualizando o repositório
    apt-get update

11) instalando o Docker
    apt-get install docker-ce docker-ce-cli containerd.io

12) Entrar o Docker Hub e dar um Fork no repositório

13) Na máquina virtual, clonar o repositório
    git clone https://github.com/flaviopv/challenge-delta.git

14) Pegar a última versão do SQL Server
    docker pull mysql/mysql-server

15) Criação do diretório mysql
    mkdir mysql

16) Criação do diretório db, dentro do mysql
    mkdir db 
    mv database_schema.sql ./mysql/db

17) Criação do Dockerfile
FROM mysql:5.6 
RUN apt-get update -y
ENV MYSQL_DATABASE packages
COPY ./db/ /docker-entrypoint-initdb.d/

18) Criação do Container
    docker build -t hurbdb .
    
19) Execução do Container
    docker run -d -p 3306:3306 --name hurbdb -e MYSQL_ROOT_PASSWORD=rootpwd -e MYSQL_DATABASE=packages -e MYSQL_USER=offeruser -e MYSQL_PASSWORD=offerpwd hurbdb

20) Criação de uma Network
    docker create network mynetwork

21) Connectar o hurbdb na rede mynetwork
    docker network connect mynetwork hurbdb

22) Criação do Dockerfile para o container hurbnode
FROM node:8
WORKDIR /node/src/app
COPY package*.json ./
RUN npm install --no-fund
COPY . .
EXPOSE 8888
CMD [ "node", "server.js" ]

23) Criação do Container 
    docker build -t hurbnode .

24) Execução do container 
    docker run -d -p 8888:8888 --name hurbnode hurbnode

25) Connectar o hurbnode na rede mynetwork
    docker network connect mynetwork hurbnode

26) Criação do Docker file para ser utilizado pelo hurbproxy
FROM nginx
RUN rm /etc/nginx/conf.d/default.conf
COPY default.conf /etc/nginx/conf.d

27) Criação do arquivo de configuração do nginxserver 
{
    listen 80;
    server_name nginxserver;

    location / {
        root /usr/share/nginx/html;
    }
    
    location /packages {
        proxy_pass http://hurbnode:8888/packages;
    }

    error_page 500 501 502 503 504 /50x.html;

    location /50x.html {
        root /usr/share/nginx/html;
    }
}

28) Criação do Container nginx-proxy
    docker build -t hurbproxy .

29) Execução do container hurbproxy
    docker run -d -p 80:80 --name hurbproxy --net mynetwork hurbproxy


OBS:

Poderiamos usar o docker compose para criar um arquivo yaml para automatizar a execução de todos o containers
