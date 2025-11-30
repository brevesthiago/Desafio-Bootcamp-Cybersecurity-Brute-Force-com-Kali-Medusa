# Desafio Bootcamp Santander Cibersegurança - Simulando um Ataque de Brute Force com Kali Linux e Medusa

Este projeto documenta a simulação de ataques de força bruta (brute-force) visando auditar a segurança de serviços de rede e demonstrar a importância de políticas de senhas robustas.

## 1. Configuração do Ambiente

O laboratório foi montado utilizando virtualização para garantir um ambiente isolado (Sandbox).

### Infraestrutura
* **Atacante:** Kali Linux
* **Alvo:** Metasploitable 2
* **Rede:** Ambas as máquinas configuradas em **Rede Interna (Host-only)** no VirtualBox para isolamento total da internet.

### Ferramentas Utilizadas
* **Reconhecimento:** Nmap, Netdiscover
* **Ataque:** Medusa


## 2. Reconhecimento e Coleta de Informações

Antes de iniciar os ataques, foi necessário identificar o endereço IP do alvo na rede interna e verificar quais versões de serviços estavam rodando.

### Identificação do Host (Host Discovery)
Utilizamos o `netdiscover` para escanear a faixa de IP da rede interna e localizar a máquina alvo.

```bash
sudo netdiscover -r 192.168.56.0/24
```

### Varredura de Portas e Serviços
Com o IP identificado (192.168.56.101), executamos o Nmap para listar as portas abertas e identificar as versões dos serviços, focando nos protocolos FTP e SMB.

```bash
nmap -sV -p 21,139,445 192.168.56.101
```
Evidência do Escaneamento: 


## 3. Execução do Ataque: FTP (Porta 21)

O objetivo desta etapa foi testar a robustez das credenciais de acesso ao serviço FTP. Utilizamos o **Medusa** para realizar um ataque de força bruta focado no usuário `msfadmin`, tentando encontrar a senha correspondente na wordlist.

### Comando Executado
```bash
medusa -h 192.168.56.101 -u msfadmin -P pass.txt -M ftp
```

### Detalhamento da Sintaxe
* `-h 192.168.56.101`: Define o IP do alvo.

* `-u msfadmin`: Define o usuário específico a ser testado.

* `-P pass.txt`: Indica o caminho da wordlist de senhas.

* `-M ftp`: Seleciona o módulo específico para o protocolo FTP.

### Resultados Obtidos
A ferramenta processou a lista e obteve êxito ao encontrar a credencial correta.

Evidência do Ataque:

Credencial Encontrada: `User: mfsadmin Password: msfadmin`


## 4. Execução do Ataque: SMB (Porta 445)

Para o serviço SMB (Samba), simulamos um cenário onde o atacante tenta validar credenciais testando uma lista de nomes de usuários comuns (`users.txt`) contra a wordlist de senhas. Isso verifica a segurança de múltiplas contas ao mesmo tempo.

### Preparação da Lista de Possíveis Usuários
Criamos um arquivo local chamado `users.txt` contendo possíveis usuários do sistema:

```bash
echo -e "admin\nroot\nmsfadmin\nuser\nguest" > users.txt
```

### Preparação da Lista de Possíveis senhas
Criamos um arquivo local chamado `users.txt` contendo possíveis usuários do sistema:

```bash
echo -e "admin\nroot\nmsfadmin\nuser\nguest" > pass.txt
```

### Comando Executado
Nesta execução, alteramos a flag de usuário único (-u) para lista de usuários (-U) e selecionamos o módulo smbnt.

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M smbnt
```

### Resultados Obtidos
A ferramenta testou as combinações e obteve êxito ao validar o acesso para o usuário msfadmin, demonstrando que a reutilização de senhas ou senhas fracas em serviços críticos (como compartilhamento de arquivos) compromete o sistema.

Evidência do Ataque:

Credencial Confirmada: `User: mfsadmin Password: msfadmin`

## 5. Relatório de Mitigação e Conclusão

Com base nos testes realizados, foi possível comprometer com facilidade tanto o serviço de transferência de arquivos (FTP) quanto o compartilhamento de rede (SMB). A causa raiz da vulnerabilidade não foi uma falha no software em si, mas sim a configuração insegura de credenciais (Senhas Fracas).

### Medidas de Mitigação Recomendadas

Para prevenir ataques de força bruta como os demonstrados com o Medusa, recomendamos a implementação das seguintes defesas:

1.  **Políticas de Senhas Robustas:**
    * Impor um comprimento mínimo (ex: 12 caracteres).
    * Exigir complexidade (mistura de maiúsculas, minúsculas, números e símbolos) para inviabilizar ataques baseados em dicionário (wordlists).

2.  **Limitação de Tentativas (Account Lockout):**
    * Configurar o sistema para bloquear temporariamente o IP de origem ou a conta de usuário após um número fixo de falhas consecutivas (ex: 3 a 5 tentativas).
    * Ferramentas como **Fail2Ban** são eficazes para automatizar esse bloqueio em servidores Linux.

3.  **Desativação de Serviços Desnecessários:**
    * Se o protocolo FTP não é estritamente necessário, ele deve ser desativado e substituído por protocolos mais seguros e criptografados, como o SFTP (SSH).

4.  **Monitoramento de Logs:**
    * Ferramentas de força bruta geram um grande volume de logs de erro. Monitorar arquivos como `/var/log/auth.log` permite detectar e responder a esses ataques em tempo real.

### Conclusão

Este projeto demonstrou na prática como ferramentas automatizadas exploram configurações padrão e senhas fracas. O sucesso dos ataques contra o FTP e SMB reforça a necessidade de endurecimento (hardening) dos servidores e a importância de nunca utilizar credenciais padrão em ambientes de produção.
