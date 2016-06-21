# Instalando Nagios 4.1.1 #

Tutorial baseado no seguinte [post](http://www.unixmen.com/how-to-install-nagios-core-4-1-1-in-ubuntu-15-10/).

Outros posts que foram úteis:  

- [https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/eventhandlers.html](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/eventhandlers.html)
- [https://thiagolucas.wordpress.com/2014/03/28/nagios-debian-instalacao-detalhada/](https://thiagolucas.wordpress.com/2014/03/28/nagios-debian-instalacao-detalhada/)

> Nagios é um sistema utilizado para monitorar a infraestrutura de rede. Usando Nagios, podemos monitorar servidores, switches, aplicações e serviços etc. Alertar o administrador do sistema quando algo der errado e também alertar quando esses erros forem corrigidos.

## Principais funcionalidades ###

- Monitorar toda sua infraestrutura de TI.
- Saber imediatamente quando problemas surgirem
- Compartilhar dados de disponibilidade com as partes interessadas
- Detectar violações de segurança
- Planos e orçamento para atualizações na infraestrutura de TI.
- Reduza as perdas de tempo de inatividade e negócios.

## Pré-requisitos ###

**Para facilitar os passos a seguir execute-os em modo *root*.**

Atualizar o servidor
  
	apt-get update  
	apt-get upgrade 

Instale as dependências do nagios

	apt-get install libgd2-xpm-dev libsnmp-perl libssl-dev openssl build-essential apache2 libapache2-mod-php5 unzip

Crie um novo usuário do ***nagios***

	useradd -m nagios
	passwd nagios

Crie um novo grupo de usuários ***nagcmd*** para permitir que comandos externos sejam enviados através da interface web. Adicione o usuário nagios e o usuário do apache ao grupo.

	groupadd nagcmd
	usermod -a -G nagcmd nagios
	usermod -a -G nagcmd www-data	


## Baixar o Nagios e Plugins ##

Crie um diretório para armazenar os downloads

	mkdir /usr/src/nagios
	cd /usr/src/nagios

Vá para a página do [Nagios](https://www.nagios.org/downloads/nagios-core/thanks/?t=1466426447) para obter a versão mais recente, a versão mais recente durante a escrita desse documento é a 4.1.1

	wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.1.1.tar.gz

E baixe também os [plugins](https://www.nagios.org/downloads/nagios-plugins/) do nagios

	wget http://www.nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz

Caso algum erro relacionado a certificados ocorra durante a executação do **wget**, execute utilizando o parâmetro **--no-check-certificate**

	wget --no-check-certificate link

## Instalar Nagios e Plugins ##

### Instalar Nagios ###

Va até a pasta onde baixou o nagios e extraia os arquivos:

	tar xzf nagios-4.1.1.tar.gz

Acesse o diretório extraído:

	cd nagios-4.1.1/


Execute os seguintes comandos um a um para compilar e instalar o nagios.

	./configure --with-command-group=nagcmd
	make all
	make install
	make install-init
	make install-config
	make install-commandmode

### Instalar a interface web do Nagios ###

Execute o seguinte comando para compilar e instalar a interface web.

	make install-webconf

Caso o seguinte erro ocorra: 
> /usr/bin/install -c -m 644 sample-config/httpd.conf /etc/httpd/conf.d/nagios.conf
> /usr/bin/install: cannot create regular file ‘/etc/httpd/conf.d/nagios.conf’: No such file or directory
> Makefile:296: recipe for target 'install-webconf' failed
> make: *** [install-webconf] Error 1

A mensagem de erro acima descreve que o nagios está tentando criar o arquivo **nagios.conf** dentro do diretório **/etc/httpd.conf/**. Mas em sistemas derivados do Ubuntu o **nagios.conf** deve ser colocado no diretório /etc/apache2/sites-enabled/

Para tal, execute o seguinte comando no lugar do **make install-webconf**.

	/usr/bin/install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-enabled/nagios.conf

Verifique se o **nagios.conf** está no diretório /etc/apache2/sites-enabled.

	ls -l /etc/apache2/sites-enabled/

O output desse comando deve ser semelhante a este:

>total 4  
lrwxrwxrwx 1 root root 35 Nov 28 16:49 000-default.conf -../sites-available/000-default.conf   
-rw-r--r-- 1 root root 1679 Nov 28 17:02 nagios.conf 

Crie uma conta ***nagiosadmin*** para a autenticação na interface web do Nagios. **Lembre da senha que você definir**, ela será utilizada para logar na interface web.

	htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

Reinicie o Apache para que as novas configurações tenham efeito.

	service apache2 restart

### Instalar Nagios Plugins ###

Va para o diretório (provavelmente o **/usr/src/nagios**) onde você baixou o nagios plugins, e extraia os arquivos 

	tar xzf nagios-plugins-2.1.1.tar.gz

Acesse o diretório extraído:

	cd nagios-plugins-2.1.1/

Execute os seguintes comandos um a um para compilar e instalar.

	./configure --with-nagios-user=nagios --with-nagios-group=nagios
	make
	make install

## Configurar o nagios ##

Os arquivos com configurações de exemplo do Nagios estão localizados no diretório **/usr/local/nagios/etc**.Esses arquivos de exemplo devem funcionar bem para uma introdução ao Nagios. No entando, se você quiser, precisará colocar o seu ID e e-mail para receber alertas.

Para tal, edite o arquivo de configuração **/usr/local/nagios/etc/objects/contacts.cfg** e altere o e-mail associado ao contato *nagiosadmin*. Este e-mail será onde os alertas serão enviados

	nano /usr/local/nagios/etc/objects/contacts.cfg

Encontre a linha a seguir e informe o seu e-mail:
 
	 [...]  
	 define contact {  
	 contact_name                    nagiosadmin             ; Short name of user  
	 use                             generic-contact         ; Inherit default values from generic-contact template (defined above)    
	 alias                           Nagios Admin            ; Full name of user   
	 email                           seuemailaqui.com    	 ; <<CHANGE THIS TO YOUR EMAIL ADDRESS
	 }  
	 [...]

Salve e feche o arquivo.

<!--
Então, edite o arquivo **/etc/apache2/sites-enabled/nagios-conf**

	nano /etc/apache2/sites-enabled/nagios.conf

E edite as seguintes linhas se você desejar acessar o nagios console administrativo de uma determinada
 faixa de IP.

	[...]
	## Comment the following lines 
	#   Order allow,deny
	#   Allow from all
	
	## Uncomment and Change lines as shown below
	Order deny,allow
	Deny from all
	Allow from 127.0.0.1 192.168.1.0/24 #seu ip vai aqui
	[...]
-->

Habilitar os módulos do Apache e reconfiguração e cgi

	a2enmod rewrite
	a2enmod cgi

Reiniciar o serviço Apache
	
	service apache2 restart

Verifique se o arquivo **nagios.cfg** contém algum erro de sintaxe:

	/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

Caso o comando acima não tenha apontado erros, podemos iniciar o serviço nagios: 

	service nagios start

Nas versões mais recentes do Ubuntu/Debian é possível que o comando acima lance o seguinte erro:

	Failed to start nagios.service: Unit nagios.service failed to load: No such file or directory.

Para corrigir esse erro é necessário copiar o arquivo **/etc/init.d/skeleton** para **/etc/init.d/nagios** utilizando o seguinte comando:

	cp /etc/init.d/skeleton /etc/init.d/nagios

Edite o arquivo **/etc/init.d/nagios**

	nano /etc/init.d/nagios

Adicione as seguintes linhas:

	DESC="Nagios"
	NAME=nagios
	DAEMON=/usr/local/nagios/bin/$NAME
	DAEMON_ARGS="-d /usr/local/nagios/etc/nagios.cfg"
	PIDFILE=/usr/local/nagios/var/$NAME.lock

Salve e feche o arquivo.

É necessário alterar as permissões do arquivo.

	chmod +x /etc/init.d/nagios

Agora é possível iniciar o serviço nagios utilizando o comando:

	/etc/init.d/nagios start 

## Acessar a interface web do Nagios ##

Abra seu navegador da web e navegue até **http://{ip-do-server-nagios}/nagios** e digite o nome de usuário como **nagiosadmin** e sua senha que criamos nos passos anteriores.

![](http://www.unixmen.com/wp-content/uploads/2015/11/192.168.1.103-nagios-Google-Chrome_001.jpg)

Aqui esta um exemplo da página de administração do Nagios:

![](http://www.unixmen.com/wp-content/uploads/2015/11/Nagios-Core-Google-Chrome_002.jpg)

Clique na seção de **"Hosts"** no painel esquerdo. Você verá os hosts que estão sendo monitorados pelo servidor Nagios. Nós ainda não adicionamos nenhum host. Então apenas o localhost será exibido na lista.

![](http://www.unixmen.com/wp-content/uploads/2015/11/Nagios-Core-Google-Chrome_003.jpg)

Clique no *localhost* para exibir mais detalhes

![](http://www.unixmen.com/wp-content/uploads/2015/11/Nagios-Core-Google-Chrome_004.jpg)


É isso, instalamos e configuramos o Nagios!

## Adicionar clintes monitorados ao servidor Nagios ##

Agora, vamos adicionar alguns clientes para serem monitorados pelo servidor Nagios.
Para isso precisamos instalar o **nrpe** e o **nagios-plugin** em nossos alvos de monitoramento.

### No Debian/Linux ###

	apt-get update
	apt-get install nagios-nrpe-server nagios-plugins

Configure os alvos de monitoramento, edite o arquivo **/etc/nagios/nrpe.cfg**,

	nano /etc/nagios/nrpe.cfg

Adicione o ip do seu servidor Nagios:

	[...]
	## Encontre as seguintes linhas e adicione o ip do servidor Nagios ##
	allowed_hosts=127.0.0.1 192.168.1.103
	[...]
	
Inicie o serviço nrpe:

	/etc/init.d/nagios-nrpe-server restart

Agora, **volte ao Nagios server** e adicione os clientes (no arquivo de configuração).
Para tal, Edite o arquivo **"/usr/local/nagios/etc/nagios.cfg"**,

	nano /usr/local/nagios/etc/nagios.cfg

e remova o comentário da seguinte linha

	## Encontre e remova o comentário da seguinte linha ##
	cfg_dir=/usr/local/nagios/etc/servers

Crie um diretório chamado **"servers"** dentro da pasta **"/usr/local/nagios/etc/"**

	mkdir /usr/local/nagios/etc/servers

Crie um arquivo de configuração para monitorar o cliente:

	nano /usr/local/nagios/etc/servers/clients.cfg

Adicione as seguintes linhas:

	define host{
	
	use                             linux-server
	host_name                       server.unixmen.local
	alias                           server
	address                         192.168.1.104 ##informe o ip do seu client aqui
	max_check_attempts              5
	check_period                    24x7
	notification_interval           30
	notification_period             24x7
	
	}  

Nesse caso, **192.168.1.104** é o ip do meu cliente Nagios e server.unixmen.local é o hostname.

Por fim, reinicie o serviço do nagios.

	service nagios restart

Aguarde alguns segundos e atualize a página administrativa do nagios no browser, e etão navegue até a seção de **hosts** no painel da esquerda. Agora, você verá o cliente que foi adicionado recentemente. Click nele para visualizer se tem alguma coisa errada ou algum alerta.

![](http://www.unixmen.com/wp-content/uploads/2015/11/Nagios-Core-Google-Chrome_005.jpg) 

Click no cliente para visualizar informações detalhadas:

![](http://www.unixmen.com/wp-content/uploads/2015/11/Nagios-Core-Google-Chrome_006.jpg)

Da mesma forma, você pode definir mais clientes criando um arquivos de configuração separados para cada cliente. 

### Definindo serviço ###

Acabamos de definir um cliente para ser monitorado. Agora, vamos adicionar alguns serviços do cliente. Por exemplo, para monitor o serviço **ssh**, adicione as seguintes linhas no arquivo **"/usr/local/nagios/etc/servers/clients.cfg"**.

	nano /usr/local/nagios/etc/services/clients.cfg

Adicione as seguintes linhas:
	

	[...]
	##ADICIONE AS LINHAS ABAIXO \/
	define service {
        use                             generic-service
        host_name                       server.unixmen.local
        service_description             SSH
        check_command                   check_ssh
        notifications_enabled           0
    }

 Salve e feche o arquivo. Reinicie o Nagios.

	service nagios restart

Aguarde alguns segundos (**90** segundos por padrão), e verifique pelos serviços adicionados (exemplo: ssh) na interface web do nagios. Navegue para a seção **Services** no menu esquerdo, você verá o serviço **ssh** na lista.

![](http://www.unixmen.com/wp-content/uploads/2015/11/Nagios-Core-Google-Chrome_007.jpg)
