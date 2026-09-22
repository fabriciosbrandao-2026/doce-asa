🛠️ Tutorial Passo a Passo: Configuração de Serviços no Debian 13 (Trixie)

Este tutorial documenta todo o procedimento para preparar o ambiente da máquina virtual acra-gana, desde o acesso remoto via SSH, passando pelo servidor Web Apache2 até o acesso à interface gráfica via RDP.

🧰 1. Especificações do Ambiente

Host Físico: PC 19

Máquina Virtual (VM): acra-gana

Endereço IP Local: 192.168.56.19

Sistema Operacional: Debian Trixie (13)

FQDN do Host: acra-gana.gana.lab

Domínios Gerenciados: www.gana.lab e docs.gana.lab (Alias: docs.lab)

🔑 ETAPA 1: Instalar e Configurar o SSH (Acesso Sem Senha)

O SSH permite gerenciar a máquina virtual Debian remotamente a partir do terminal no Windows (Git Bash ou PowerShell).

1.1. Instalar e iniciar o serviço SSH no Debian

No terminal do Debian (como root ou com sudo):

# Atualizar a lista de pacotes
sudo apt update

# Instalar o servidor SSH
sudo apt install openssh-server -y

# Ativar e iniciar o serviço
sudo systemctl enable ssh
sudo systemctl start ssh


1.2. Gerar o Par de Chaves no Cliente (Windows)

No terminal da sua máquina local (Git Bash ou PowerShell):

ssh-keygen -t ed25519 -C "admin@acra-gana"


(Pressione Enter em todas as confirmações para aceitar os locais padrão sem senha adicional).

1.3. Enviar a Chave Pública para o Debian

Envie a chave pública para que o Debian reconheça seu computador:

ssh-copy-id root@192.168.56.19


1.4. Testar o Acesso Direto sem Senha

ssh root@192.168.56.19


🌐 ETAPA 2: Instalar o Apache2 e Configurar os Serviços HTTP

Com o acesso SSH estabelecido, instalamos o servidor web Apache2 para hospedar os sites locais.

2.1. Instalar o Apache2 no Debian

sudo apt update
sudo apt install apache2 -y

# Ativar e iniciar o serviço
sudo systemctl enable apache2
sudo systemctl start apache2


2.2. Criar a Estrutura de Diretórios dos Portais Web

Crie as pastas que armazenarão os arquivos dos dois domínios:

sudo mkdir -p /srv/http/www.gana.lab
sudo mkdir -p /srv/http/docs.gana.lab


2.3. Criar os Arquivos Web (index.html)

1. Site Principal (/srv/http/www.gana.lab/index.html):

sudo nano /srv/http/www.gana.lab/index.html


(Cole o código HTML do portal principal contendo o botão redirecionando para http://docs.gana.lab).

2. Site de Documentação (/srv/http/docs.gana.lab/index.html):

sudo nano /srv/http/docs.gana.lab/index.html


(Cole o código HTML contendo os dados do servidor e status do sistema).

2.4. Permissões de Leitura do Apache

Ajuste as permissões do diretório para o usuário do Apache (www-data):

sudo chown -R www-data:www-data /srv/http/
sudo chmod -R 755 /srv/http/


2.5. Configurar os VirtualHosts do Apache

1. VirtualHost www.gana.lab:

sudo nano /etc/apache2/sites-available/www.gana.lab.conf


Conteúdo do arquivo:

<VirtualHost *:80>
    ServerName www.gana.lab
    DocumentRoot /srv/http/www.gana.lab

    <Directory /srv/http/www.gana.lab>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>


2. VirtualHost docs.gana.lab:

sudo nano /etc/apache2/sites-available/docs.gana.lab.conf


Conteúdo do arquivo:

<VirtualHost *:80>
    ServerName docs.gana.lab
    ServerAlias docs.lab
    DocumentRoot /srv/http/docs.gana.lab

    <Directory /srv/http/docs.gana.lab>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>


2.6. Ativar os VirtualHosts

# Ativar as configurações criadas
sudo a2ensite www.gana.lab.conf
sudo a2ensite docs.gana.lab.conf

# Validar se a sintaxe dos arquivos está correta
sudo apache2ctl configtest

# Recarregar as configurações no Apache
sudo systemctl reload apache2


2.7. Configuração de DNS no Cliente (Windows)

Para abrir os domínios .lab no navegador do Windows:

Abra o Bloco de Notas como Administrador.

Abra o arquivo: C:\Windows\System32\drivers\etc\hosts.

Adicione esta linha no final:

192.168.56.19   www.gana.lab docs.gana.lab docs.lab


Salve e limpe o cache DNS no Prompt de Comando (CMD):

ipconfig /flushdns


🖥️ ETAPA 3: Instalar e Configurar o Acesso Gráfico RDP (XRDP)

Por fim, configuramos o servidor RDP para acessar a área de trabalho do Debian remotamente pelo Windows (mstsc).

3.1. Instalar e Ativar o Serviço XRDP no Debian

# Instalar o servidor de área de trabalho remota
`sudo apt update`
`sudo apt install xrdp -y`

# Ativar no boot e iniciar o serviço
`sudo systemctl enable xrdp`
`sudo systemctl start xrdp`


3.2. Conectar pelo Windows

Pressione Win + R, digite mstsc e pressione Enter.

No campo Computador, informe o IP: 192.168.56.19.

Clique em Conectar e entre com seu usuário e senha do Debian.

🐙 ETAPA 4: Publicação da Documentação no GitHub (Codespaces)

Com os serviços ativos e testados, suba o arquivo index.md para o seu repositório GitHub via terminal ou VS Code / Codespaces:

# Verificar o estado do repositório
`git status`

# Adicionar a documentação
`git add index.md`

# Criar o commit com mensagem descritiva
`git commit -m "docs: adiciona tutorial de instalacao ordenada (SSH, Apache2/HTTP, RDP)"`

# Enviar para a branch principal no GitHub
`git push origin main`


🔍 Comandos de Validação e Diagnóstico

# Verificar status de todos os serviços principais
```
sudo systemctl status ssh
sudo systemctl status apache2
sudo systemctl status xrdp
```

# Testar resposta local das páginas HTTP
`curl -I http://www.gana.lab`
`curl -I http://docs.gana.lab`
