# Zabbix_Install_5.0.2

Antes de instalarmos o Zabbix, é necessário instalar todos esses pacotes: https://github.com/gabrielteixeiranetwork/Servidor-WEB-NGINX-PHP-PostgreSQL-phpPgAdmin-Letsencrypt-no-Debian-11

Instalação no Debian 11

    su -
    cd /tmp
    apt install wget
Respositório Debian 11

    wget https://repo.zabbix.com/zabbix/5.0/debian/pool/main/z/zabbix-release/zabbix-release_5.0-2+debian11_all.deb
    dpkg -i zabbix-release_5.0-2+debian11_all.deb
Atualize o repositório

    apt update ; apt upgrade
Realize a instalação

    apt install zabbix-server-pgsql zabbix-frontend-php zabbix-nginx-conf zabbix-agent
Vamos criar uma base de dados chamada zabbix e um usuário também chamado de zabbix no PostgreSQL

    su - postgres
    createuser --pwprompt zabbix
    Digite a senha para a nova role:  <SENHA ZABBIX>
    Digite-a novamente: <SENHA ZABBIX>
    Senha: <SENHA POSTGRES CASO TENHA DEFINIDO NA INSTALAÇÃO DO MESMO>
 
    createdb -O zabbix zabbix
    Senha: <SENHA POSTGRES>
Importe o esquema inicial e os dados sem sair do banco

    zcat /usr/share/doc/zabbix-server-pgsql*/create.sql.gz | psql -U zabbix -d zabbix &>/dev/null
    Senha para usuário zabbix: : <SENHA ZABBIX>
    exit
Edite o arquivo zabbix_server.conf para informar os dados para conexão com do PostgreSQL

    vim /etc/zabbix/zabbix_server.conf
Procure por DBPassword=/timeoute=/CacheSize=

    DBPassword=<SENHA ZABBIX>
    Timeout= 30
    CacheSize= 64M
Ajuste o arquivo /etc/zabbix/php-fpm.conf

    ; php_value[date.timezone] = Europe/Riga
    para
    php_value[date.timezone] = America/Sao_Paulo   
Ajuste as configurações do nginx  vim /etc/nginx/conf.d/zabbix.conf
    
    server {
        listen 80;
        listen [::]:80;
        server_name    zabbix.gabrielteixeiraconsultoria.com.br localhost;
        #server_name   OU_SEU_IP;
 
        # Metodo simples para quem quer rodar em uma determinada porta
        #listen 8181;
        #listen [::]:8181;
        #server_name     _;
 
        root    /usr/share/zabbix;
        index   index.php;
 
        # Desmomente para deixar restringido apenas para determinados prefixos
        #allow  192.168.87.0/24;
        #allow  127.0.0.1;
        #allow  2001:0db8::/32;
        #allow  ::1;
        #deny   all;
        #error_page  403   http://www.gabrielteixeiraconsultoria.com.br;
 
        location = /favicon.ico {
                log_not_found   off;
        }
 
        location / {
                try_files       $uri $uri/ =404;
        }
 
        location /assets {
                access_log      off;
                expires         10d;
        }
 
        location ~ /\.ht {
                deny            all;
        }
 
        location ~ /(api\/|conf[^\.]|include|locale) {
                deny            all;
                return          404;
        }
 
        location ~ [^/]\.php(/|$) {
                fastcgi_pass    unix:/var/run/php/zabbix.sock;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_index   index.php;
 
                fastcgi_param   DOCUMENT_ROOT   /usr/share/zabbix;
                fastcgi_param   SCRIPT_FILENAME /usr/share/zabbix$fastcgi_script_name;
                fastcgi_param   PATH_TRANSLATED /usr/share/zabbix$fastcgi_script_name;
 
                include fastcgi_params;
                fastcgi_param   QUERY_STRING    $query_string;
                fastcgi_param   REQUEST_METHOD  $request_method;
                fastcgi_param   CONTENT_TYPE    $content_type;
                fastcgi_param   CONTENT_LENGTH  $content_length;
 
                fastcgi_intercept_errors        on;
                fastcgi_ignore_client_abort     off;
                fastcgi_connect_timeout         60;
                fastcgi_send_timeout            180;
                fastcgi_read_timeout            180;
                fastcgi_buffer_size             128k;
                fastcgi_buffers                 4 256k;
                fastcgi_busy_buffers_size       256k;
                fastcgi_temp_file_write_size    256k;
        }
    }
Inicie o servidor Zabbix
    
    systemctl enable zabbix-server zabbix-agent
    systemctl restart zabbix-server zabbix-agent nginx php7.4-fpm
Acesse em seu navegador http://seu_ip:porta ou http://zabbix.seudominio.com.br
![image](https://user-images.githubusercontent.com/94009104/234943543-fa9c75c0-d23e-439d-8df7-d4f3786ba8f4.png)
![image](https://user-images.githubusercontent.com/94009104/234943638-6b603905-e909-419c-ba8b-298fe082abde.png)
![image](https://user-images.githubusercontent.com/94009104/234943727-5b350c24-cf77-4f73-a58f-2679e28ed32f.png)
![image](https://user-images.githubusercontent.com/94009104/234943781-67ca5845-3c2e-41b1-a7f8-5ea2ff2c8be7.png)
![image](https://user-images.githubusercontent.com/94009104/234943817-8c8e0c03-879e-4427-bd32-f3924bfdb0d1.png)
![image](https://user-images.githubusercontent.com/94009104/234943903-ae04b35d-ed7a-48ac-af7e-1d74fe0246a9.png)

Entre com o Usuário Admin e a senha zabbix

![image](https://user-images.githubusercontent.com/94009104/234944017-e869f4fe-2022-4453-aa61-9608a89975d8.png)

Configuração do zabbix segue nesse link
       
https://github.com/gabrielteixeiranetwork/Configuration_Zabbix_5.0.2
