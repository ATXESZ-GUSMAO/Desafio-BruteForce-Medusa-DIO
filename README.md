Desafio DIO: Brute Force com Kali Linux e Medusa


Autor: Lucas Gabriel Gusmão
Formação: Análise e desenvolvimento de sistemas / Cybersecurity Specialist
Data da Simulação: Novembro/2025

Objetivo do Desafio

O objetivo principal deste projeto foi implementar e documentar cenários práticos de ataque de força bruta em ambientes controlados, utilizando o Kali Linux como plataforma de ataque e a ferramenta Medusa, explorando vulnerabilidades comuns em serviços como FTP, SMB e formulários Web. O foco foi exercitar técnicas ofensivas para melhor compreender e propor medidas de mitigação eficazes.


2. Configuração do Ambiente de Teste

Para garantir um teste seguro e isolado, o ambiente foi configurado com máquinas virtuais (VMs) no VirtualBox:

* Máquina Atacante: Kali Linux (2024.x)
    * Endereço IP: `192.168.56.101` 
* Máquina Alvo: Metasploitable 2
    * Endereço IP: `192.168.56.102` 
* Rede: As VMs foram configuradas com Rede Interna para assegurar o isolamento do teste.

3. Wordlists Utilizadas

Para a simulação, foram criadas wordlists simples focadas em credenciais comuns e default do ambiente Metasploitable 2.

| Nome do Arquivo | Conteúdo (Amostra) | Propósito |
| :--- | :--- | :--- |
| `usuarios.txt` | `msfadmin`, `user`, `admin`, `ftpuser`, `postgres` | Teste contra múltiplos usuários. |
| `senhas.txt` | `123456`, `password`, `msfadmin`, `teste`, `root` | Teste contra múltiplas senhas comuns. |

Obs.: Estes arquivos serão disponibilizados no repositório.

4. Ataques Simulados e Evidências

Todos os testes foram realizados com sucesso, identificando credenciais fracas nos serviços alvo.

4.1. Cenário: Força Bruta em FTP

O serviço FTP (`vsftpd 2.3.4`) do Metasploitable 2 foi o alvo, demonstrando a ausência de lockout de tentativas.

* Comando Medusa:
    ```bash
    medusa -H 192.168.56.102 -U usuarios.txt -P senhas.txt -M ftp -n 21
    ```
* Resultado e Evidência:
    * O Medusa rapidamente identificou a credencial padrão.
    * Credencial Encontrada: `msfadmin:msfadmin`
    * Medusa successful brute force on FTP service showing the command line and the 'SUCCESS' output with msfadmin:msfadmin
* Validação: O acesso foi confirmado via linha de comando no Kali: `ftp 192.168.56.102`.

4.2. Cenário: Password Spraying em SMB

O ataque explorou o serviço SMB (Samba) do Metasploitable 2, usando a técnica de *password spraying* (uma senha comum contra muitos usuários).

* Comando Medusa (Password Spraying):
    ```bash
    medusa -H 192.168.56.102 -U usuarios.txt -p msfadmin -M smb
    ```
* Resultado e Evidência:
    * A ferramenta reportou sucesso para diversos usuários que compartilham a mesma senha padrão.
    * Credencial Comum Encontrada: `msfadmin`
    * [Descrição da Evidência em Imagem:] Medusa successful brute force on SMB service showing the command line and the 'SUCCESS' output for multiple users with the password msfadmin
* Observação: A enumeração prévia de usuários com o Nmap (`nmap --script smb-enum-users 192.168.56.102`) foi fundamental para criar a lista de usuários alvo.

 4.3. Cenário: Força Bruta em Formulário Web (DVWA)

Este teste focou na seção "Brute Force" do Damn Vulnerable Web Application (DVWA), executando a segurança no nível Low para demonstrar a vulnerabilidade à ausência de rate limiting.

* Comando (Utilizando Hydra para HTTP POST):
    * Nota: O Medusa pode ser complexo para POST requests, por isso, a ferramenta complementar Hydra foi usada para garantir o sucesso do ataque Web.
    ```bash
    hydra -L usuarios.txt -P senhas.txt 192.168.56.102 http-post-form "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:F=Login failed" -V
    ```
* Resultado e Evidência:
    * O Hydra identificou rapidamente as credenciais de login do DVWA.
    * Credencial Encontrada: `admin:password`
    * Hydra output showing the successful login credential (admin:password) for the DVWA web form



5. Recomendações de Mitigação

A simulação demonstrou que a ausência de controles de segurança resulta em credenciais comprometidas em segundos. As seguintes recomendações são essenciais para mitigar ataques de força bruta:

1.  Limite de Tentativas (Rate Limiting): Implementar bloqueio temporário do IP ou do usuário após um número pequeno (ex: 5) de tentativas de login falhas.
2.  MFA e CAPTCHA: Exigir Autenticação Multifator (MFA) e utilizar CAPTCHA/reCAPTCHA, especialmente em formulários web, para dificultar a automação.
3.  Política de Senhas Fortes: Forçar o uso de senhas com alta entropia (longas, complexas, sem caracteres sequenciais ou comuns).
4.  Monitoramento e Alerta: Configurar sistemas de Monitoramento de Log (SIEM) para alertar sobre volumes anormais de tentativas de autenticação em um curto período.
5.  Desabilitar/Bloquear Serviços Legados: Desabilitar serviços antigos e inseguros como FTP/Telnet, ou limitar o acesso apenas a IPs internos e confiáveis (ACLs).



6. Conclusão

Este desafio reforçou a importância da configuração segura de serviços. Ferramentas como Medusa e Hydra são extremamente eficazes e rápidas contra credenciais fracas. O entendimento prático do ataque é o primeiro passo para construir defesas robustas.
