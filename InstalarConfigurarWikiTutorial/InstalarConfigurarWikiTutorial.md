# Como instalar e configurar sua própria Wiki - LAMP e Docker

## LAMP

Esta sessão apresenta os passos de installing do da TikiWiki e seus requisitos em uma instalação "padrão" de um servidor web LAMP, em um Ubuntu Server, usando Apache2, MySQL5 e PHP5/PHP7. Estes passos são baseados na [documentação oficial da Tiki](https://doc.tiki.org/Ubuntu-Install) e [neste tutorial do LinuxHelp](https://www.linuxhelp.com/how-to-install-tiki-wiki-cms-on-ubuntu19-04), em conjunto de várias horas experimentando combinações de configurações e passos diferentes.

O LAMP, que é um acrônimo de Linux, Apache, Mysql e PHP, forma a base para publicar um servidor Web para sites desenvolvidos em PHP. Estes quatro ~~Cavaleiros do Apocalipse~~ softwares combinados formam uma das maneiras mais tradicionais colocar um site ou CMS online.

Neste tutorial, como já foi citado acima, será utilizado o Ubuntu Server, mas os passos podem ser adaptados para outras distribuições, especialmente se elas forem debian-based. Não sejá feita uma abordagem a fundo em _hardening_ de servidores ou configurações do servidor, além da instalação e configuração da TikiWiki.

O único recurso opcional, porém recomendado, é possuir um servidor SSH Server instalado neste servidor, para facilitar a ~~cópia~~  configuração via comandos remotamente, sem precisar acessar o servidor diretamente.

### Requisitos

Inicialmente, antes de fazer qualquer instalação de novos pacotes, garanta que o sistema do seu servidor está atualizado. Isso pode ser feito com os comandos abaixo:

```ShellSession
sudo apt update
sudo apt upgrade
```
Alguns dos pacotes abaixo podem vir a ser necessários, dependendo de sua preferência e escolhas no passos mais adiante. Você pode deixar todos instalados ou instalar conforme necessidade durante o processo.

```ShellSession
sudo apt install wget nano vim tar gzip xzip zip unzip rar unrar
```

#### Pacotes

Em seu servidor (Linux), instale o restante dos pacotes necessários (Apache, Mysql, PHP) para o funcionamento da Wiki:

```ShellSession
sudo apt install mysql-server mysql-client apache2 curl composer php php-tidy php-pear memcached php-gd php-xmlrpc phpmyadmin php-mbstring libapache2-mod-php php-mysql php-apcu php-curl php-intl php-sqlite3 php-zip postfix subversion php-memcache php7.4-gettext php-pspell php-zip poppler-utils php-memcached bsdmainutils pstotext catdoc elinks man-db odt2txt php-pear pstotext php-common php-intl php7.4-opcache php7.4-xml php7.4-zip php7.4-ldap
```
### Crie o banco de dados para a aplicação


```ShellSession
sudo mysql -u root -p
```

```sql
create database tikiwiki;
```

```sql
 create user 'tikiuser'@localhost identified by 'TutorialWikiPlus';
```

```sql
grant all privileges on tikiwiki.* to 'tikiuser'@localhost;
```

```sql
flush privileges;
```

```sql
exit
```

### Baixe e instale o CMS

Para executar tudo em um comando, copie e execute o comando a seguir no terminal do seu servidor:

Baixe a versão desejada, seja ela a mais recente ou alguma anterior, de acordo com preferencia. Para obter recursos mais recentes e seguros, sempre é recomendado usar versão estável mais recente. Neste tutorial irei utilizar a versão 22.x Corona Borealis, mais precisamente a versão 22.1.

Ao escolher o arquivo para download, você pode escolher entre algumas variações de formatos de empacotamento e compressão de arquivos. Esta escolha vai de preferência

```ShellSession
wget -c https://sourceforge.net/projects/tikiwiki/files/Tiki_22.x_Corona_Borealis/22.1/tiki-22.1.tar.gz
```

```ShellSession
tar -vzxf tiki-22.1.tar.gz
```

```ShellSession
sudo sudo mv tiki-22.1 /var/www/html/tikiwiki
```

```ShellSession
sudo chown -R www-data.www-data /var/www/html/tikiwiki
```

```ShellSession
sudo chmod -R 755 /var/www/html/tikiwiki
```

Talvez precise, só fazer se pedir:

```ShellSession
sudo chmod -R 755 /var/www/html/tikiwiki/db
sudo chmod -R 755 /var/www/html/tikiwiki/dump/
sudo chmod -R 755 /var/www/html/tikiwiki/img/trackers/
sudo chmod -R 755 /var/www/html/tikiwiki/img/wiki_up/
sudo chmod -R 755 /var/www/html/tikiwiki/img/wiki
sudo chmod -R 755 /var/www/html/tikiwiki/modules/cache/
sudo chmod -R 755 /var/www/html/tikiwiki/temp/
sudo chmod -R 755 /var/www/html/tikiwiki/templates/
```

```ShellSession
sudo nano /etc/apache2/sites-available/tiki.conf
```

```ShellSession
sudo a2dissite 000-default.conf
```

```ShellSession
sudo a2ensite tiki.conf
```

```ShellSession
sudo a2enmod rewrite
```

```ShellSession
sudo systemctl restart apache2
```


#### Automatizar


```ShellSession
cd /tmp && wget https://raw.githubusercontent.com/lucassfank/knowledge/main/InstalarConfigurarWikiTutorial/extra/lamp/install.sh && chmod +x install.sh && ./install.sh
```


### Instalação e configuração

Abrir o navegador pelo IP ou se tiver um domínio

```
http://<ip_ou_dominio>/tiki-install.php
```


## docker

DockerHub links
[DockerHub haproxy](https://hub.docker.com/_/haproxy)
[DockerHub tikiwiki](https://hub.docker.com/r/tikiwiki/tikiwiki)
[DockerHub mariadb](https://hub.docker.com/_/mariadb)

```ShellSession
nano docker-compose.yml
```

```yml
version: '3.7'
...
```

```ShellSession
sudo docker-compose up -d
```

```ShellSession
sudo docker ps -a
```

```ShellSession
sudo docker-compose ps
```

```ShellSession
sudo docker ps -a
```

```ShellSession
sudo docker-compose exec tiki php console.php database:install
```

```ShellSession
sudo docker exec -it mysql_db_1 mysql -utikiuser -p
```

```sql
show databases;
```

```sql
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('NovaSenhaForte!!!!');
```

```sql
FLUSH PRIVILEGES;
```

```sql
exit
```

```ShellSession
cp /var/www/html/tikiwiki/img/Logo.png
```

Favicon browser tab
"Place a single favicon named favicon-16x16.png in your themes favicons directory to continue using traditional favicons on your website."

```ShellSession
mkdir themes/default/favicons
cp /var/www/html/tikiwiki/themes/default/favicons/favicon-16x16.png
```

```ShellSession
nano .htaccess
```

```
php_value memory_limit 512M
php_value max_execution_time 90
php_value max_input_time 360
php_value upload_max_filesize 50M
php_value post_max_size 80M
```

```ShellSession
nano /var/www/html/lang/pt-br/language.php
```

### Restaurando os dados

```ShellSession
sudo cp tikiwiki.sql data/data/
```

```ShellSession
sudo docker exec -it mysql_db_1 bash
```

```ShellSession
mysql -utikiuser -p tikiwiki < /var/lib/mysql/data/tikiwiki.sql
```

```ShellSession
exit
```

```ShellSession
sudo docker exec -it mysql_db_1 mysql -utikiuser -p
```
