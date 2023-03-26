---
#layout: post
title:  "Zabbix simple setup"
categories: [monitoring]
tags: [devops, monitoring]
---

# Zabbix 6.0 LTS

0 - O lab pode rodar em 4Ram 2vCpu 50G HD

1- A primeira coisa a se arrumar é o relógio do sistema, pois tudo do zabbix gira em torno de horário e data.

```bash
timedatectl status
// o timezone pega UTC,
timedatectl set-timezone America/Sao_Paulo
// timedatectl list-timezone | grep America

date // pra confirmar

dnf -y install chrony // é o ntp, que sincroniza a hora sempre

systemctl enable --now chronyd // dá start e força inicar com o sistema

```

E permitir acesso pelo firewall somente ao http

```bash
systemctl status firewalld // pega o status

fireawall-cmd --permanent --add-service=http //permite requisição http

firewall-cmd --reload // aplicar config

dnf install -y net-tools vim wget curl tcpdump //ferramentas gerais

getenforce // ve se tá permissive

sestatus // SELINUX não é necessário no zabbix 6.0

$ /etc/selinux/config
// talvez seja necessário mudar o SELINUX=permissive e dar reboot //no sistema
// SELINUX = feature de segurança do linux, pode bloquear algumas config

```

## MySQL para Zabbix

```bash
dnf info mysql-server // 8.0.26 já atende o zabbix

dnf -y install mysql-server // pra instalar um bd ,

systemctl enable --now mysqld

// em produção é melhor seccionar os acessos, logo o usuário a utilizar o acesso // não tera ao mysql se não for root, em lab usarei o root mesmo

```

### Conectando e Criando BD

```bash
mysql // pra conectar, como sudo apenas

create database zabbix character set utf8mb4 collate utf8mb4_bin;

create user 'zabbix'@'localhost' identified by 'Zabbix6!user';

grant all privileges on zabbix.* to 'zabbix'@'localhost';

flush privileges;

exit;
```

## PostGreeSQL para Zabbix

```bash
dnf install postgresql postgresql-contrib # pra isntalar

systemctl enable --now postgresql # pra isntalar
```

Na documentação oficial tem:

```bash
# Install the repository RPM:
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Disable the built-in PostgreSQL module:
sudo dnf -qy module disable postgresql

# Install PostgreSQL:
sudo dnf install -y postgresql14-server

# Optionally initialize the database and enable automatic start:
sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
sudo systemctl enable postgresql-14
sudo systemctl start postgresql-14
```

### Conectando e Criando o Banco

```bash
# pra criar o usuário e vai pedir senha 
sudo -u postgres createuser --pwprompt zabbix 
# pwd: Zabbix6!user

#criando banco de dados
sudo -u postgres createdb -O zabbix -E Unicode ou utf8mb4 -T template zabbix 
```

## Instalando Zabbix Server

```bash
# pegar o repositório do zabbix
rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-6.0-1.el8.noarch.rpm

# limpar repos antigos
dnf clean all

# instalar de fato instalação dos pacotes
dnf install zabbix-server-mysql zabbix-sql-scripts zabbix-selinux-policy
##OU##

# em caso de postgre 
dnf install zabbix-server-pgsql zzabbix-sql-scripts zabbix-selinux-policy

# vamos ao diretório de scripts de sql do zabbix
cd /usr/share/doc/zabbix-sql-scripts/mysql/

# e rodamos o script de server
zcat server.sql.gz | mysql -u zabbix -p zabbix // senha= Zabbix6!user

//verificar se o schema funcionou
mysql -u zabbix -p zabbix 

mysql> show tables; //173 rows
mysql> quit;
```

### Configurando Zabbix

Na documentação terá mais detalhes do que aqui, com certeza

```bash
vim /etc/zabbix/zabbix_server.conf

# vamos procurar por DBPassowrd e setar como
DBPassword=Zabbix6!user

#só salvar e depois
systemctl enable --now zabbix.service

# olhamos o log pra ver se statou
tail -n50 /var/log/zabbix/zabbix_server.log
```

### Instalando o FronEnd

Muitas vezes é recomendado o uso de instancias separadas para cada parte ( Zabbix | SQL | FrontEnd)

Não faremos isso nesse lab para testar como se sairá

```bash
dnf -y install zabbix-web-mysql zabbix-nginx-conf

# deposis da instalação comentamos de
vim /etc/nginx/nginx.conf

server {
#        listen       80 default_server; //essa
#        listen       [::]:80 default_server; //essa
#        server_name  _; //e essa linha
        root         /usr/share/nginx/html;

# e em seguida vamos pra
vim /etc/nginx/conf.d/zabbix.conf

#onde descomentamos e ALTERAMOS
server {
        listen          80;
        server_name     _; //orignal é example.com;

# logo em seguida permitimos
systemctl enable --now nginx php-fpm

```

### Configurando o FrontEnd

No final das contas abrimos no navegador o IP público da máquina e entraremos no Setup. Onde é só seguir o Passo a Passo.

## Instalando o Grafana

### Verificando Resposta

Instalamos o zabbix_get com

```bash
yum install zabbix-get
```

Pra ver se o agent tá ativo

```bash
zabbix_get -s $ip_do_host -k system.hostnam # vai cuspior o nome do host 
```

### Agent Linux

Instalamos, permitimos e configuramos o agent no linux

Partindo do princípio que seja **debian** based

#### Instalando

```bash
# adicionamos o repositório
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-1+ubuntu20.04_all.deb

# efetuamos o release
sudo dpkg -i zabbix-release_6.0-1+ubuntu20.04_all.deb

# damos update
sudo apt update

# instalamos o agent 6
sudo apt install zabbix-agent -y
```

### Permitindo

```bash
# permitindo no firewall
sudo ufw allow 10050/tcp
# resetando o firewall
sudo ufw disable
sudo ufw enable
```

### Configurando

```bash
# abrindo o arquivo de confi
sudo vim /etc/zabbix/zabbix_agentd.conf

# procuramos por Server e ServerActive e colocamos o endereço do zabbix
Serve=168.62.50.200 # em alguns casos é necessário colocar o ip INTERNO
ServeActive=168.62.50.200 # 10.11.0.4

# e alteramos o Hostname para o nome utilizado no Zabbix
# ao adicionar o host no zabbix adicionamos igualmente no arquivo de config
Hostname=$nomeHost

# salvamos e resetamos
sudo systemctl enable --now zabbix-agent
sudo systemctl restart zabbix-agent
```

> ⚠️ Testamos no servidor do Zabbix com o comando [Zabbix-get](https://www.notion.so/Zabbix-ec21cbd2c9b94a39b70cb4bddfb0207b)



## Integrando o Grafana

após a instalação vamos executar a configuração que integra o Grafana ao banco do Zabbix.
Este que vamos em **SERVER ADMIN** → **PLUGINS** → procuramos por Grafana e seguimos o manual:

Logamos no Servidor do Grafana e executamos o script de instalação:

```bash
grafana-cli plugins install alexanderzobnin-zabbix-app
```

E resetamos o serviço.

Depois seguimos o **[DOC](https://alexanderzobnin.github.io/grafana-zabbix/configuration/) oficial.**

# Links extras

[http://phucnw.blogspot.com/2014/11/cleaning-up-zabbix-database.html](http://phucnw.blogspot.com/2014/11/cleaning-up-zabbix-database.html)

[https://techexpert.tips/zabbix/monitor-iis-using-zabbix/](https://techexpert.tips/zabbix/monitor-iis-using-zabbix/)

# Configurações Adicionais

## Problems

Atualmente estamos lidando com alguns avisos de problemas recorrentes que podem ser evitados, mediante algumas configurações simples.

### HTTP Pooler

Nesse caso há aviso constante de que HttpPoolers passou dos 75% basta alterarmos isso de maneira que as requisições Web possam ocorrer sem exercer muito estresse no sistema todo.

```sh
sudo vim /etc/zabbix/zabbix_server.conf
// procuramos pelo seguinte controle abaixo

### Option: StartPollers
#       Number of pre-forked instances of pollers.
#
# Mandatory: no
# Range: 0-1000
# Default:
# StartPollers=5

// # comenta o código, vamo alterar o StartPoller para 10. e descomentar
// salvamos e reiniciamos o serviço
	sudo systemctl restart zabbix-server.service
```
