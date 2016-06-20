# Instalando Nagios 4.1.1 #

Tutorial baseado no seguinte [post](http://www.unixmen.com/how-to-install-nagios-core-4-1-1-in-ubuntu-15-10/).

> Nagios é sistema, que pode ser usado para monitoramento de infra-estrutura de rede. Usando Nagios, podemos monitorar servidores, switches, aplicações e serviços etc. Alerta o administrador do sistema quando algo der errado e também alertas quando esses erros tenham sido corrigidos.

## Principais funcionaliades ###

- Monitorar toda sua infraestrutura de ti.
- Identify problems before they occur.
- Identificar problemas antes que eles aconteçam
- Saber imediatamente quando problemas surgirem
- Compartilhar dados de disponibilidade com as partes interessadas
- Detectar violações de segurança
- Planos e orçamento para upgrades de TE.
- Reduza as perdas de tempo de inatividade e negócios.

## Pré-requisitos ###

**Para facilitar os passos a seguir execute-os em modo *root*.**

Atualizar o servidor
  
	apt-get update  
	apt-get upgrade 

Instale as dependências do nagios

	apt-get install libgd2-xpm-dev libsnmp-perl libssl-dev openssl build-essential apache2 libapache2-mod-php5

Crie um novo usuário do ***nagios***

	useradd nagios -g nagios
	passwd nagios

Crie um novo grupo de ***nagcmd*** para permitir que comandos externos sejam enviados através da interface web. Adicione o usuário nagios e o usuário do apache ao grupo.

	sudo groupadd nagcmd
	sudo usermod -a -G nagcmd nagios
	sudo usermod -a -G nagcmd www-data	


## Baixar o Nagios e Plugins ##

Crie um diretório para armazenar os downloads

	mkdir /usr/src/nagios
	cd /usr/src/nagios

Vá para a página do [Nagios](https://www.nagios.org/downloads/nagios-core/thanks/?t=1466426447) para obter a versão mais recente, a versão mais recente durante a escrita desse documento é a 4.1.1

	wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.1.1.tar.gz

E, baixe também os [plugins](https://www.nagios.org/downloads/nagios-plugins/) do nagios

	wget http://www.nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz

## Instalar Nagios e Plugins ##

### Instalar Nagios ###

Va até a pasta onde baixou o nagios, e extraia os arquivos:

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

A mensagem de erro a cima descreve que o nagios está tentando criar o arquivo **nagios.conf** dentro do diretório **/etc/httpd.conf/ **. Mas em sistemas derivados do Ubuntu o **nagios.conf** deve ser colocado no diretório /etc/apache2/sites-enabled/

Para tal, execute o seguinte comando no lugar do **make install-webconf**.

	sudo /usr/bin/install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-enabled/nagios.conf

Verifique se o **nagios.conf** está no diretório /etc/apache2/sites-enabled.

	sudo ls -l /etc/apache2/sites-enabled/

O output desse comando deve ser semelhante a:

>total 4  
lrwxrwxrwx 1 root root 35 Nov 28 16:49 000-default.conf -../sites-available/000-default.conf   
-rw-r--r-- 1 root root 1679 Nov 28 17:02 nagios.conf 

Crie um conta ***nagiosadmin*** para se autenticar a interface web do Nagios. **Lembre da senha que você definir**, ela será utilizada para logar na interface web.

	htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

Reinicie o Apache para que as novas configurações tenham efeito.

	service apache2 restart

### Instalar Nagios Plugins ###

Va para o diretório onde você baixou o nagios plugins, e extraia os arquivos.

	tar xzf nagios-plugins-2.1.1.tar.gz

Acesse o diretório extraído:

	cd / nagios-plugins-2.1.1/

Execute os seguintes comandos um a um para compilar e instalar.

	./configure --with-nagios-user=nagios --with-nagios-group=nagios
	make
	make install

## Configurar o nagios ##

Os arquivos com configurações de exemplo do Nagios estão localizados no diretório **/usr/local/nagios/etc**.Esses arquivos de exemplo devem funcionar bem para uma introdução ao Nagios. No entando, se você quiser, precisará colocar o seu ID e e-mail para receber alertas.

Para tal, edite o arquivo de configuração **/usr/local/nagios/etc/objects/contacts.cfg** e altere o e-mail associado a contato *nagiosadmin* este e-mail será onde os alertas serão enviados

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

Save and close the file.

Então, edite o arquivo **/etc/apache2/sites-enabled/nagios-conf**

	nano /etc/apache2/sites-enabled/nagios.conf

E edite as seguintes linhas, se você desejar acessar o nagios console administrativo de uma determinada
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

Habilitar os módulos do Apache e reconfiguração e cgi

	a2enmod rewrite
	a2enmod cgi

Reiniciar o serviço Apache
	
	service apache2 restart

Verifique se o arquivo **nagios.cfg** contém algum erro de sintaxe:

	/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

Caso o comando a cima não tenha apontado erros, podemos iniciar o serviço nagios: 

	service nagios start

## Acessar a interface web do Nagios ##

Abra seu navegador da web e navegue até **http://{ip-do-server-nagios}/nagios** e digite o nome de usuário como **nagiosadmin** e sua senha que criamos nos passos anteriores.

![](http://www.unixmen.com/wp-content/uploads/2015/11/192.168.1.103-nagios-Google-Chrome_001.jpg)

Aqui esta um exemplo da página de administração do Nagios:

![](http://www.unixmen.com/wp-content/uploads/2015/11/Nagios-Core-Google-Chrome_002.jpg)

Clique na seção de "Hosts" no painel esquerdo. Você verá os hosts que estão sendo monitorados pelo servidor Nagios. Nós ainda não adicionou nenhum host. Então ele simplesmente monitora o localhost apenas.

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

Configure os alvos de monitoramento

Edite o arquivo **/etc/nagios/nrpe.cfg**,

	nano /etc/nagios/nrpe.cfg

Adicione o ip do seu servidor Nagios:

	[...]
	## Encontre as seguintes linhas e adicione o ip do servidor Nagios ##
	allowed_hosts=127.0.0.1 192.168.1.103
	[...]
	
Inicie o serviço nrpe:

	/etc/init.d/nagios-nrpe-server restart

Agora, **volte ao Nagios server** e adicionei os clientes (no arquivo de configuração).
Para tal, Edite o arquivo **"/usr/local/nagios/etc/nagios.cfg"**,

	nano /usr/local/nagios/etc/nagios.cfg

e remova o comentário da seguinte linha

	## Encontre e remova o comentário da seguinte da seguinte linha ##
	cfg_dir=/usr/local/nagios/etc/servers

Crie um diretório chamado **"servers"** dentro da pasta "/usr/local/nagios/etc/"

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

Aguarde alguns segundos, e atualize a página administrativa do nagios no browser e navegue até a sessão de **hosts** no painel da esquerda. Agora, você verá o cliente que foi adicionado recentemente. Click nele para visualizer se tem alguma coisa errada ou algum alerta.

![](http://www.unixmen.com/wp-content/uploads/2015/11/Nagios-Core-Google-Chrome_005.jpg) 

Click no cliente para visualizar informações detalhadas:

![](http://www.unixmen.com/wp-content/uploads/2015/11/Nagios-Core-Google-Chrome_006.jpg)

Da mesma forma, você pode definir mais clientes criando um arquivos de configuração separados para cada cliente. 
Definindo serviços  