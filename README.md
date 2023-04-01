# Zabbix-Vagrant

# Instalação com servidor Zabbix

Neste projeto estou subindo 3 máquinas virtuais com **Vagrant** um deles será o servidor, o proxy e o cliente eles são monitorado, mas agora vou está ensinando a subir as máquinas e instalar o servidor **Zabbix**

Primeiro vamos criar o `Vagrantfile` em nosso Vscode utilizando o terminal para fazer o `vagrant up` para criar nossa máquinas virtuais com o sistema operacional Debian 11

~~~yml
Vagrant.configure("2") do |config|
    # Setting up the quantity of VM's will be configured, in this case, 1 to 4.
        (1..3).each do |i| 
            # Using the number of VM "i" to compose it VirtualBox name.
            config.vm.define "zabbix_#{i}" do |zabbix|
                zabbix.vm.box = "debian/bullseye64"
                # Using the number of VM "i" to compose it hostname
                zabbix.vm.hostname = "zabbix#{i}"
    #           zabbix.vm.provision "routeros_command", name: "Teste", command: "/system resource print"
                zabbix.vm.network "public_network", bridge: "enp3s0"
    # 	        zabbix.vm.network "private_network", virtualbox__intnet: "lan#{i}", auto_config: false
                zabbix.vm.provider "virtualbox" do |v| 
                    # Setting up custom settings
                    v.memory    = 1024
                    v.name      = "zabbix#{i}"
                end
            end
        end 
end
~~~


Agora vamos acessar uma das máquinas, damos um `vagrant ssh zabbix_1` no meu caso vou fazer a instalação na zabbix_1 e agora vamos seguir o passo a passo. 

**Atualizar o sitema operacional**

`apt update && apt upgrade`

**Ver IP do servidor**

`ip add`

**Instalar o Servidor SSH**

`apt install openssh-server`

**Ver o status do serviço**

`systemctl status ssh`


**Para não ter que entrar como superusuário e e ter que repetir sempre o processo utilize o comando**

`nano /etc/profile` 
editamos para `PATH="/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr/local/games"`


**Para não ter que entrar como superusuário e e ter que repetir sempre o processo utilize o comando:
 echo "PATH="/sbin:/usr/sbin:/bin:/usr/bin:/usr/local/bin:/usr/local/sbin:/usr/games:/usr/local/games"" >> /etc/environment 

**Instalar o Apache , o PHP e alguns módulos PHP necessários**

`apt install -y apache2 apache2-bin apache2-data apache2-utils libapache2-mod-php libapache2-mod-php7.4 libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap libcurl4 libgd3 liblua5.3-0 libonig5 libsodium23 libxpm4 libxslt1.1 php php-bcmath php-common php-gd php-ldap php-mbstring php-mysql php-xml php7.4 php7.4-bcmath php7.4-cli php7.4-common php7.4-gd php7.4-json php7.4-ldap php7.4-mbstring php7.4-mysql php7.4-opcache php7.4-readline php7.4-xml ssl-cert`

**Verificar se o Apache está rodand**

`systemctl status apache2`

**Instalar o MariaDB para armazenar todos os dados do Zabbix. O MariaDB substitui o MySQL nas distribuições atuais do Debian.**

`apt install mariadb-server mariadb-client`

**Verificar se o MariaDB está rodando**

`systemctl status mariadb`

**Proteger a instalação do MariaDB**

~~~sql
mysql_secure_installation
Enter current password for root (enter for none): Press Enter
Switch to unix_socket authentication [Y/n] y
Change the root password? [Y/n] y
New password: <Criação do password>
Re-enter new password: <Criação do password>
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]:  Y
Reload privilege tables now? [Y/n]:  Y
~~~

**Criar o banco de dados par ao Zabbix**

~~~~sql
mysql -u root -p
MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;
MariaDB [(none)]> grant all privileges on zabbix.* to zabbix@localhost identified by '123456';
MariaDB [(none)]> set global log_bin_trust_function_creators = 1;
MariaDB [(none)]> quit; 
~~~~

**Ver a versão do Debian instalada**

`lsb_release -a`

**Selecionar a versão do Zabbix**

`Selecionar a versão do Zabbix: https://www.zabbix.com/br/download`

**Baixar o Zabbix no Debian**

`wget https://repo.zabbix.com/zabbix/6.4/debian/pool/main/z/zabbix-release/zabbix-release_6.4-1+debian11_all.deb`

**Preparar o pacote para instalação**

`dpkg -i zabbix-release_6.4-1+debian11_all.deb`

`apt update`

**Instalar o servidor, o frontend e o agente Zabbix**

`apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent`

**Importar o esquema do banco**

`zcat  /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p'Seu password' zabbix`

**Editar arquivo de configuração do servidor (Use CTRL+w no Nano para pesquisar)**

~~~sql	
  nano /etc/zabbix/zabbix_server.conf 
		-> DBHost=localhost
		-> DBName=zabbix
		-> DBUser=zabbix
		-> DBPassword= Seu password	
~~~

**Reiniciar o server,agente e o apache** 

`systemctl restart zabbix-server zabbix-agent apache2`

**Colocar os serviços com início automático**

`systemctl enable zabbix-server zabbix-agent apache2`

**Conecte-se ao frontend Zabbix instalado**

~~~
URL: http://server_ip_or_name/zabbix 
User: Admin
Senha: zabbix
~~~

# Instalação do Poroxy 

**Atualizar o sitema operacional**
`apt update && apt upgrade`

**Para não ter que entrar como superusuário e e ter que repetir sempre o processo utilize o comando**

`nano /etc/profile` 
editamos para `PATH="/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr/local/games"`


**Baixar o Zabbix no Debian***
wget https://repo.zabbix.com/zabbix/6.4/debian/pool/main/z/zabbix-release/zabbix-release_6.4-1+debian11_all.deb

**Preparar o pacote para instalação**
 
 `dpkg -i zabbix-release_6.4-1+debian11_all.deb`
 
 `apt update`

**Instalar os componentes**

`apt -y install zabbix-proxy-mysql zabbix-sql-scripts`

**Instalar o MariaDB para armazenar todos os dados do Zabbix. O MariaDB substitui o MySQL nas distribuições atuais do Debian.**

`apt install mariadb-server mariadb-client`

**Verificar se o MariaDB está rodando**
`systemctl status mariadb`

**Proteger a instalação do MariaDB**

~~~sql
mysql_secure_installation
	Enter current password for root (enter for none): Press Enter
	Switch to unix_socket authentication [Y/n] y
	Change the root password? [Y/n] y
	New password: <Enter root DB password>
	Re-enter new password: <Repeat root DB password>
	Remove anonymous users? [Y/n]: Y
	Disallow root login remotely? [Y/n]: Y
	Remove test database and access to it? [Y/n]:  Y
	Reload privilege tables now? [Y/n]:  Y
~~~~

**Criar o banco de dados par ao Zabbix**

~~~sql
mysql -u root -p
MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;
MariaDB [(none)]> grant all privileges on zabbix.* to zabbix@localhost identified by 'password';
MariaDB [(none)]> set global log_bin_trust_function_creators = 1;
MariaDB [(none)]> quit; 
~~~

**Popular o Banco de Dados**

`cat /usr/share/zabbix-sql-scripts/mysql/proxy.sql | mysql --default-character-set=utf8mb4 -uzabbix -p'password' zabbix`

**Editar o arquivo de configuração do zabbix proxy (Use CTRL+w no Nano para pesquisar):** 
 ~~~sql
 nano /etc/zabbix/zabbix_proxy.conf
  -> DBHost ->> localhost
  -> DBName ->>zabbix
  -> DBPassword ->>P@ssw0rd
  -> Hostname ->>zabbixproxy
  -> Server ->>IP-do-zabbixserver
~~~

**Iniciar o proxy**

`systemctl restart zabbix-proxy`

**Colocar o serviço em automático**

`systemctl enable zabbix-proxy`

**Registrar o Proxy no Servidor Zabbix**
~~~yml
Administração->Proxies

Ver se deu tudo certo, depois do registro no servidor: tail -f /var/log/zabbix/zabbix_proxy.log
~~~















