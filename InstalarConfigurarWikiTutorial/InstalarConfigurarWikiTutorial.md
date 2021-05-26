# Como instalar e configurar sua própria Wiki - LAMP e Docker

## LAMP

Esta sessão apresenta os passos de installing do da TikiWiki e seus requisitos em uma instalação "padrão" de um servidor web LAMP, em um Ubuntu Server, usando Apache2, MySQL5 e PHP5/PHP7. Estes passos são baseados na [documentação oficial da Tiki](https://doc.tiki.org/Ubuntu-Install) e [neste tutorial do LinuxHelp](https://www.linuxhelp.com/how-to-install-tiki-wiki-cms-on-ubuntu19-04), em conjunto de várias horas experimentando combinações de configurações e passos diferentes.

O LAMP, que é um acrônimo de Linux, Apache, Mysql e PHP, forma a base para publicar um servidor Web para sites desenvolvidos em PHP. Estes quatro ~~Cavaleiros do Apocalipse~~ softwares combinados formam uma das maneiras mais tradicionais colocar um site ou CMS online.

Neste tutorial, como já foi citado acima, será utilizado o Ubuntu Server, mas os passos podem ser adaptados para outras distribuições, especialmente se elas forem debian-based.

O único recurso opcional, porém recomendado, é possuir um servidor SSH Server instalado neste servidor, para facilitar a ~~cópia~~  configuração via comandos remotamente, sem precisar acessar o servidor diretamente.

### Requisitos
Inicialmente, antes de fazer qualquer instalação de novos pacotes, garanta que o sistema do seu servidor está atualizado. Isso pode ser feito com os comandos abaixo:
```ShellSession
sudo apt update
sudo apt upgrade
```

#### Pacotes

Em seu servidor (Linux), instale o restante dos pacotes necessários (Apache, Mysql, PHP) para o funcionamento da Wiki:

```ShellSession
sudo apt install mysql-server mysql-client apache2 curl composer php php-tidy php-pear memcached php-gd php-xmlrpc phpmyadmin php-mbstring libapache2-mod-php php-mysql php-apcu php-curl php-intl php-sqlite3 php-zip postfix subversion php-memcache php7.4-gettext php-pspell php-zip poppler-utils php-memcached bsdmainutils pstotext catdoc elinks man-db odt2txt php-pear pstotext php-common php-intl php7.4-opcache php7.4-xml php7.4-zip php7.4-ldap
```

Para executar tudo em um comando, copie e execute o comando a seguir no terminal do seu servidor:

```ShellSession
cd /tmp && wget https://raw.githubusercontent.com/lucassfank/knowledge/main/InstalarConfigurarWikiTutorial/extra/lamp/install.sh && chmod +x install.sh && ./install.sh
```

## docker

DockerHub links
[DockerHub haproxy](https://hub.docker.com/_/haproxy)
[DockerHub tikiwiki](https://hub.docker.com/r/tikiwiki/tikiwiki)
[DockerHub mariadb](https://hub.docker.com/_/mariadb)

```ShellSession
nano docker-compose.yml
```


```
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


```
show databases;
```


```
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('NovaSenhaForte!!!!');
```


```
FLUSH PRIVILEGES;

```


```
exit
```



```
cp /var/www/html/tikiwiki/img/Logo.png
```

Favicon browser tab
"Place a single favicon named favicon-16x16.png in your themes favicons directory to continue using traditional favicons on your website."
```
mkdir themes/default/favicons
cp /var/www/html/tikiwiki/themes/default/favicons/favicon-16x16.png
```



```
nano .htaccess
```


```
php_value memory_limit 512M
php_value max_execution_time 90
php_value max_input_time 360
php_value upload_max_filesize 50M
php_value post_max_size 80M
```


```
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

