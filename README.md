---
description: Passos simples para a instalação do apache tomcat 9 no ubuntu 18.04
---

# Instalação do Apache Tomcat 9 no Ubuntu 18.04 LTS

## Introdução

O apache **tomcat** é um servidor web e servlet container que é usado para servir aplicações java. O **tomcat** é uma implementação java servlet e paginas javaservlet. Esta documentação contempla apenas os passos basicos da instalação dessa ferramenta no ubuntu 18.04.

## Pre-requisitos

Antes de começarmos a seguir os passos deste documento, devemos possuir uma conta com privilegios de super usuario.

## Preparando o sistema

O Tomcat requer que ja tenha instalado o Java, assim qualque aplicação java web pode ser executada. Iremos suprir essa necessidade instalando o **OpenJDK** atravez do terminal linux utilizando o comando apt.

### Atualize o apt

Primeiro devemos atualizar a lista interna do nosso sistema gerenciador atravez do comando:

```text
sudo apt update
```

em seguida instale o kit de desenvolvimento java:

```text
sudo apt install default-jdk
```

Agora que instalamos o Java, nos podemos criar um usuario tomcat, o qual sera usado para rodar o service do Tomcat.

## Criando o usuario Tomcat

Por motivos de segurança, o tomcat deve rodar em um usuario sem permissões, criaremos um novo usuario e grupo que ira rodar os serviços do Tomcat.

Primeiro, crie um novo tomcat grupo:

```text
sudo groupadd tomcat
```

Em seguida, crie um novo usuario, chamaremos de tomcat. Iremos fazer esse usuario ser parte do nosso grupo tomcat, com um diretorio home em /opt/tomcat - local onde iremos instalar o tomcat- , e com o shell /bin/false, dessa forma ninguem podera logar nessa conta:

```text
sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
```

Agora que nosso usuario tomcat esta configurado, podemos baixar e instalar o tomcat.

## Instalando o Tomcat

A melhor maneira de instalar o Tomcat é fazendo o download da ultima versao binaria lançada e entao configura-la manualmente.

Voçê pode encontrar a ultima versao em [https://tomcat.apache.org/download-90.cgi](https://tomcat.apache.org/download-90.cgi). Navegue na pagina ate encontrar a seccao de distribuiçao binaria, entao na lista copie o link que termina com a extensao "tar.gz".

Em seguida no terminal mude seu diretorio atual para /tmp, este é um bom diretorio para colocarmos itens temporarios, os quais nao precisaremos manter em nossa maquina apos extrairmos.

```text
cd /tmp
```

Usaremos o curl para fazer o download do tomcat atravez do link que copiamos na seccao binaria do site oficial da ferramenta:

```text
curl -O http://mirror.cc.columbia.edu/pub/software/apache/tomcat/tomcat-9/v9.0.10/bin/apache-tomcat-9.0.10.tar.gz
```

nos instalaremos o tomcat em /opt/tomcat, Crie o diretorio e extraia os arquivos com esses comandos:

```text
sudo mkdir /opt/tomcat
sudo tar xzvf apache-tomcat-9*tar.gz -C /opt/tomcat --strip-components=1
```

## Atualizando permissões

O usuario tomcat que configuramos precisa de alguns acessos para permitir a instalação. Iremos configurar agora.

Va para o diretorio que descompactamos o tomcat:

```text
cd /opt/tomcat
```

De permissao ao grupo tomcat

```text
sudo chgrp -R tomcat /opt/tomcat
```

Mude as permissoes do grupo tomcat sobre o diretorio:

```text
sudo chmod -R g+r apache-tomcat-9.0.27/conf
sudo chmod g+x apache-tomcat-9.0.27/conf
```

Agora faça do usuario Tomcat priprietario dos diretorios webapps, work, temp e logs:

```text
sudo chown -R tomcat apache-tomcat-9.0.27/webapps/ 
sudo chown -R tomcat apache-tomcat-9.0.27/work/
sudo chown -R tomcat apache-tomcat-9.0.27/temp/
sudo chown -R tomcat apache-tomcat-9.0.27/logs/
```

Agora que as permissoes foram configuradas, podemos criar um serviço de sistemas para gerenciar os processos do tomcat.

## Criando o arquivo de serviço de sistema

Nos precisamos abilitar o Tomcat para que seja executado como um serviço, entao iremos configurar o arquivo de serviço para isso.

O Tomcat precisa saber onde Java esta instalado. Este caminho geralmente esta referenciado em "JAVA\_HOME", a forma facil de ver essa localizaçao é atravez do comando:

```text
sudo update-java-alternatives -l
```

{% code-tabs %}
{% code-tabs-item title="OUTPUT" %}
```text
java-1.8.0-openjdk-amd64       1081       /usr/lib/jvm/java-1.8.0-openjdk-amd64
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="JAVA\_HOME" %}
```text
/usr/lib/jvm/java-1.8.0-openjdk-amd64
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Seguindo esta informação, podemos criar o arquivo de serviço. Abra o arquivo chamado tomcat.service no /etc/systemd/system digitando:

```text
sudo nano /etc/systemd/system/tomcat.service
```

Cole o conteudo a seguir dentro do arquivo aberto pelo nano. Modifique os valor de JAVA\_HOME se necessario para que seja igual ao seu, como foi mostrado nos comandos acima. Voce tambem deve modificar a alocaçao de memoria com algumas configuraçoes especificas em CATALINA\_OPTS:

```text

[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

Quando finalisar, salve e feche o arquivo.

Agora recarregue o sistema daemon:

```text
sudo systemctl daemon-reload
```

Inicie o serviço Tomcat digitando:

```text
sudo systemctl start tomcat
```

{% code-tabs %}
{% code-tabs-item title="OUTPUT" %}
```text
 tomcat.service - Apache Tomcat Web Application Container
   Loaded: loaded (/etc/systemd/system/tomcat.service; disabled; vendor preset: enabled)
   Active: active (running) since Fri 2019-11-01 15:52:55 -03; 27s ago
  Process: 5308 ExecStart=/opt/tomcat/bin/startup.sh (code=exited, status=0/SUCCESS)
 Main PID: 5316 (java)
    Tasks: 34 (limit: 4915)
   CGroup: /system.slice/tomcat.service
           └─5316 /usr/lib/jvm/java-1.8.0-openjdk-amd64/bin/java -Djava.util.logging.config.file=/opt/t

nov 01 15:52:55 silvio systemd[1]: Starting Apache Tomcat Web Application Container...
nov 01 15:52:55 silvio startup.sh[5308]: Using CATALINA_BASE:   /opt/tomcat
nov 01 15:52:55 silvio startup.sh[5308]: Using CATALINA_HOME:   /opt/tomcat
nov 01 15:52:55 silvio startup.sh[5308]: Using CATALINA_TMPDIR: /opt/tomcat/temp
nov 01 15:52:55 silvio startup.sh[5308]: Using JRE_HOME:        /usr/lib/jvm/java-1.8.0-openjdk-amd64
nov 01 15:52:55 silvio startup.sh[5308]: Using CLASSPATH:       /opt/tomcat/bin/bootstrap.jar:/opt/tomc
nov 01 15:52:55 silvio startup.sh[5308]: Using CATALINA_PID:    /opt/tomcat/temp/tomcat.pid
nov 01 15:52:55 silvio startup.sh[5308]: Tomcat started.
nov 01 15:52:55 silvio systemd[1]: Started Apache Tomcat Web Application Container.


```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Ajuste o Firewall e Teste o servidor Tomcat

Agora que iniciamos o tomcat, poderemos testar para ter certeza que a pagina padrao esta ativa.

Antes de fazermos isso, precisamos ajustar o firewall para permitir nossas requisições ao servidor.

```text
sudo ufw allow 8080
```

Com o firewall modificado, voce pode acessar a pagina padrao seguindo seu dominio ou endereço ip com :8080 em um navegador:

{% code-tabs %}
{% code-tabs-item title="Abra no navegado" %}
```text
http://server_domain_or_IP:8080
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Voce deve ver a pagina padrao do tomcat, o nosso proximo passo sera configurar a interface gerenciadora.

Se tudo der certo ate aqui, voce pôde acessar a pagina do tomcat, agora é um bom momento para abilitar o arquivo do tomcat para iniciar automaticamente no boot:

```text
sudo systemctl enable tomcat
```

## Interface gerenciadora do Tomcat

Para acessarmos o gerenciador que web do tomcat, nos precisamos adicionar um login ao nosso servidor. Now iremos editar o tomcat-users.xml para fazer isso.

```text
sudo nano /opt/tomcat/conf/tomcat-users.xml
```

Voce precisara adicionar um usuario que possa acessar o manager-gui e admin-gui\(sao web apps que vem com o tomcat\). Voce pode fazer isso definindo um usuario, similar ao comando abaixo entre tomcat-users.

```text
<tomcat-users>
    <user username="admin" password="password" roles="manager-gui, admin-gui"/>
</tomcat-users>
```

Salve  e feche o arquivo apos finalizar.

Para utilizarmos nossas configuraçoes, devemo reiniciar o tomcat seguindo o comando:

```text
sudo systemctl restart tomcat
```

## Conclusão

Agora voce terminou de instalar o tomcat, agora voce pode fazer o deploy de suas aplicações java localmente.

Mas lembre-se, ainda existem funcionalidades que podem ser instaladas e/ou modificadas a depender de sua necessidade, isso que dizer que dependendo do que voce queira fazer, voce precisara modificar ou instalar alguma coisa para que funcione.

## Autor <a id="autor"></a>

* ​[Linkedin](https://www.linkedin.com/in/silvio-antonio-de-oliveira-junior-621813142/)​
* ​[Github](https://github.com/silvioantonio)​
* ​[website](http://silvioantonio.ml/)​

> Compartilhar conhecimento é a melhor forma de aprender
>
> -OLIVEIRA, Silvio Antonio Junior.

