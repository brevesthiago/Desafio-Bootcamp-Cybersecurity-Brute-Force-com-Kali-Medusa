# Desafio Bootcamp Cibersegurança - Simulando um Ataque de Brute Force com Kali Linux e Medusa

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

### Preparação da Wordlist
Utilizamos a wordlist padrão `rockyou.txt`, nativa do Kali Linux. Caso o arquivo esteja compactado, execute o seguinte comando:

```bash
# Localiza e descompacta a wordlist
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```

## 2. Reconhecimento e Coleta de Informações

Antes de iniciar os ataques, foi necessário identificar o endereço IP do alvo na rede interna e verificar quais versões de serviços estavam rodando.

### Identificação do Host (Host Discovery)
Utilizamos o `netdiscover` para escanear a faixa de IP da rede interna e localizar a máquina alvo.

```bash
sudo netdiscover -r 192.168.56.0/24
