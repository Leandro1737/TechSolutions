# Projeto Final - Administração de Redes de Computadores
# TechSolutions

## Leandro Maciel e Kauã Luiz Pereira Lopes

Trabalho final da disciplina de Administração de Redes de Computadores, curso Sistemas de Informação do Instituto Federal Goiano Campus Ceres.
A prosposta do projeto final é "projetar, implantar e gerenciar uma rede empresarial usando tecnologia Linux, com ênfase em serviços como DHCP, DNS, Web, FTP, NFS e virtualização."

# Visão Geral
# 1. Introdução
Este projeto implementa uma rede interna utilizando pfSense como firewall/roteador e duas máquinas Linux Mint para teste de DHCP e conectividade.

# 2. Topologia da Rede 
A topologia utilizada segue o modelo estrela, no qual o pfSense centraliza o tráfego interno e externo.

                       
                       INTERNET
                          |
                     pfSense (Firewall)
                    WAN: 192.168.1.5/24
                          |
                    LAN: 172.16.0.35/24
                          |
          -------------------------------------
          |                |                 |
   

  

    Servidor Linux    Estação Cliente    Demais dispositivos
    Mint (Serviços)    Linux Mint        da rede (simulados)
    172.16.0.42        172.16.0.40

O pfSense funciona como o gateway padrão da rede e distribui endereços IP automaticamente pela interface LAN.

# DHCP
Para configurar o servidor DHCP, foi necessário acessar a interface visual por meio do IP, recebido via DHCP na interface WAN, colocando-o no navegador. Ao acessar o pfSense no navegador, é necessário ir na aba Serviços -> Servidor DHCP. Após, é necessário marcar a opção de "Habilitar servidor DHCP na interface LAN". Por padrão, já vem pré-estabelecido com a mesma rede do IP configurado manualmente na LAN durante a configuração do pfSense, sendo neste caso a rede 172.16.0.0/24 foi usado o range começando no IP 172.16.0.40 e acabando no IP 172.16.0.130. Para a configuração do DNS em que DHCP vai ofertar, foi colocado o IP do servidor pfSense na LAN (172.16.0.35), pois o servidor DNS também será configurado posteriormente. Na opção "Gateway" também é atribuído o IP do servidor pfSense, o dominío atribuído é o "lan". .

### Em resumo:
#### 1 - Habilitar servidor DHCP na interface LAN
#### 2 - Subrede: 172.16.0.0/24, Subrede range: 172.16.0.1 - 172.16.0.254 (Vem pré-estabelecido de acordo com o IP da LAN, atribuído manulmente no pfSense)
#### 3 - Address Pool Range: 172.16.0.40 - 172.16.0.130 (Usado neste exemplo de rede)
#### Essa segmentação acomoda até 91 dispositivos internos, suficiente para:

#### Laptops e desktops dos funcionários

#### Servidores internos

#### Impressoras de rede

#### Telefones VoIP

#### Dispositivos de IoT corporativos

#### WAN (simulada para acesso externo)

#### Endereço: 192.168.1.5/24
#### Obtido via rede interna pelo VirtualBox, usado exclusivamente para acesso à internet da VM pfSense
#### 4 - Servidores DNS: 172.16.0.35 (IP do próprio servidor pfSense, pois também será configurado um servidor DNS)
#### 5 - Gateway: 172.16.0.35 (IP do próprio servidor)
#### 6 - Domain name: lan (Dominío criado para esta rede)



4. Serviços Implementados no Projeto
4.1 Firewall e Roteamento – pfSense

O pfSense cumpre os seguintes requisitos:

# DNS
Para configurar o DNS resolver, é necessário ir na barra de navegação e procurar por Serviços -> DNS Resolver. Ao acessar à opção é necessário marcar a opção "Ativar o resolvedor de DNS" e a porta de escuta permanece a padrão 53. Na parte de "Interface de Rede" deve-se deixar a opção "Todos", estabelecendo que o DNS vai aceitar e responder consultas de todas as interfaces. Em "Interfaces de Rede de Saída" configurar como "WAN", definindo que a WAN vai ser a interface que vai enviar as consultas a DNS externos. 

### Em resumo: 
#### 1 - Ativar o resolvedor de DNS
#### 2 - Porta de escuta: 53 (Porta padrão do DNS)
#### 3 - Interface de rede: Todos (O servidor DNS vai aceitar e responder consultar de todas as interfaces)
#### 4 - Interface de rede de saída: WAN (A WAN vai ser a interface responsável por enviar consultas a DNS externos)

# Apache 
Nesta rede, o servidor Apache foi criado e testado de modo simples. Iniciando-se com a instalação do próprio servidor, na máquina virtual que é servidor (172.16.0.42). Após a instalação verificar se o serviço está rodando e testá-lo colocando o IP  172.16.0.42 e aparecendo a página padrão do servidor Apache.

### Passo a passo: 
#### 1 - sudo apt install apache2 -y (Instalar o apache)
#### 2 - systemctl status apache2 (Consultar se está rodando, se estiver terá algo parecido com "Active: active (running)")
#### 3 - Para testar é só colocar o IP do servidor no navegador (172.16.0.42), na máquina servidor ou cliente, aparecendo assim a página HTML padrão do Apache.

# FTP
De forma inicial, para configurar o servidor FTP e utilizar do compartilhamento de arquivos, é necessário que a máquina virtual esteja com a placa de rede habilitada como "Rede Interna".

### Passo a passo: 
### Na máquina servidor:  
#### 1 - sudo apt update (Para atualização dos repositórios)
#### 2 - sudo apt upgrade (Para atualizar os pacotes já instalados no sistema operacional)
#### 3 - sudo apt install vsftpd -y (Para a instalação do VSFTPD)

Após a instalação, foi verificado se o serviço estava em execução corretamente, utilizando o comando 
#### 4 - sudo systemctl status vsftpd. O resultado esperado para funcionamento adequado é: **Active: running.**

Em seguida, com CTRL + W, localizar as configurações e inserir a diretiva: 
#### 7 - allow_writeable_chroot=YES. 

Também é necessário garantir que a linha **chroot_local_user** esteja como **YES**. Ao confirmar ambos, salvar com CTRL + O, pressionar ENTER e sair com CTRL + X. 
Para o erro "**500 OOPS: run two copies of vsftpd for IPv4 and IPv6"**, deve-se retornar ao arquivo de configuração 
#### 8 - sudo nano /etc/vsftpd.conf 
e ajustar as diretivas para:
#### 9 - listen=YES e listen_ipv6=NO. 

Após salvar, o serviço deve ser reiniciado com: 
#### 10 - sudo systemctl restart vsftpd. 

### Na máquina cliente
É feito o acesso utilizando o comando: 
#### 1 - ftp 172.16.0.42 
que corresponde ao IP do servidor previamente configurado. 
Após inserir o nome de usuário e senha definidos no servidor, o acesso é estabelecido. 
Para listar e verificar os arquivos disponíveis, utiliza-se o comando: 
#### 2 - ls

## códigos e testes usados no FTP:

 sudo apt install vsftpd -y -- instalação
 
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak --fazer backup arquivo de configuração

sudo nano /etc/vsftpd.conf -- editar a configuração do FTP

allow_writeable_chroot=YES -- libera o acesso das permissões dentro do conf para não haver restrição na hora da configuração.

 sudo systemctl restart vsftpd -- reseta o serviço FTP
 
sudo systemctl status vsftpd -- verificar o status FTP

sudo adduser ftpuser -- adicionar usuário FTP

 sudo chmod -R 755 /home/ftpuser -- dar permissão a pasta do usuário
 
sudo mkdir /home/ftpuser/ftp -- criar pasta expecifica para usuário

sudo chmod 755 /home/ftpuser/ftp -- conceder permissão 

 ftp 172.16.0.35 -- acessa a página de teste
 
 ftp://ftpuser@172.16.0.42 -- forçar para acessar na web

# NFS
Para configurar o servidor NFS, foi necessário fazer a instalação do pacote nfs-kernel-server na máquina Servidor, configurando-o corretamente e aplicando as configurações. Na máquina cliente, é necessário baixar o pacote nfs-common para utilizar o serviço do NFS e após, montar o diretério compartilhado em um diretório local. 

## códigos e testes usados no NFS:

sudo apt install nfs-kernel-server -y -- instalação do servidor NFS

 sudo mkdir -p /srv/nfs -- criar pasta que será compartilhada
 
 sudo chmod 777 /srv/nfs -- ajustar permissões
 
 sudo nano /etc/exports -- configurar o arquivos de exports
 
 /srv/nfs 172.16.0.0/24(rw,sync,no_subtree_check) -- configurara o arquivo de exports
 
 sudo exportfs -ra -- aplicar as configurações
 
 sudo systemctl restart nfs-kernel-server -- reiniciar o serviço NFS
 
 sudo exportfs -v -- lista todas as pastas exportadas
 

 sudo mkdir -p /mnt/nfs --criar ponto de montagem
 
 sudo mount 172.16.0.40:/srv/nfs /mnt/nfs -- montar compartilhamento
 
 ls /mnt/nfs -- teste
 
 sudo touch /mnt/nfs/arquivo_teste.txt -- teste


