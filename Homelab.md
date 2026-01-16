# Homelab: TL‑WA850RE V7 – Firmware, SSH e Segurança

## 1️⃣ Contexto Inicial

- **Dispositivo:** TL‑WA850RE V7
- **Problema inicial:** conexão recusada ao tentar acessar a interface web pelo IP conhecido
- **Situação de rede:** repetidor conectado à LAN doméstica via Wi-Fi; PC com internet funcionando normalmente

---

## 2️⃣ Diagnóstico Inicial

### 2.1 Testes básicos

| Teste                     | Resultado                          |
|----------------------------|-----------------------------------|
| Ping no repetidor          | ✅ Respondendo                     |
| Acesso web via navegador   | ❌ Conexão recusada                |
| Identificação do MAC       | ✅ 20:23:51:73:1A:8A → TP-Link    |

### 2.2 Tabela ARP no PC

- Comando usado: `arp -a`
- IP correspondente ao MAC do repetidor: **192.168.1.4**
- Outros IPs da LAN listados para referência
- Confirmação do fabricante via API: **TP-Link Systems Inc**

### 2.3 Scan de portas com Nmap

Comando: `nmap 192.168.1.4`

| Porta   | Estado | Serviço |
|---------|--------|---------|
| 22/tcp  | open   | ssh     |
| 80/tcp  | open   | http    |
| 443/tcp | open   | https   |

**Observação:** Porta SSH aberta indicava servidor OpenSSH antigo (legado).

---

## 3️⃣ Teste SSH (diagnóstico técnico)

- Comando inicial: `ssh admin@192.168.1.4`  
  Resultado: erro de KEX → `diffie-hellman-group1-sha1`

- Teste avançado com algoritmos forçados:  
```powershell
ssh -o KexAlgorithms=+diffie-hellman-group1-sha1 -o HostKeyAlgorithms=+ssh-dss admin@192.168.1.4
```
**Resultado:** erro ssh-dss → servidor usa chave DSA antiga


**Conclusão:**

**SSH** funcional, mas obsoleto

Não suportado oficialmente para gestão

Potencial risco de **segurança** se exposto

Scan avançado com Nmap:

`nmap -p 22 -sV 192.168.1.4`
**Resultado:** OpenSSH 6.6.0, confirmando versão antiga

## 4️⃣ Diagnóstico da interface web
Inicialmente: Erro 401 ao tentar upgrade de firmware

Causas: sessão expirada, login inválido, cache do navegador

Resolução: login novamente, navegador limpo, aba privada

Conclusão: interface web é o método oficial e seguro de gestão

## 5️⃣ Upgrade de Firmware
Firmware anterior: **1.0.12** Build 221109 Rel.31477n (6985)

Firmware atualizado: **1.1.0** Build 250808 Rel.34045n (6985)

**Procedimento seguro adotado**
  - PC conectado via cabo Ethernet ao repetidor
  - Internet alternativa via hotspot do smartphone
  - Backup de configuração
  - Upload do arquivo correto para hardware V7
  - Monitoramento durante o upgrade
  - Verificação pós-upgrade da versão e estabilidade

## 6️⃣ Estado pós-upgrade
Teste	Resultado
Ping no repetidor	✅ OK
Acesso web	✅ OK
Scan de portas	22/tcp filtered, 80/443 abertas
SSH	inativo ou filtrado da LAN → risco praticamente zero

Observação: porta SSH agora filtrada, reduzindo riscos de ataque interno

## 7️⃣ Medidas de segurança adotadas
Senha do repetidor alterada para descartável

Testes de SSH realizados apenas localmente

Bloqueio/restrição de SSH após testes (via firewall ou configuração interna, se disponível)

Backup das configurações antes do upgrade

## 8️⃣ Conclusão e aprendizado
Identificação correta do IP real do repetidor via ARP + MAC + Nmap

Diagnóstico de portas e serviços (SSH legado e web)

Reconhecimento de riscos de segurança associados ao SSH antigo

Procedimento seguro de atualização de firmware

Validação pós-upgrade: porta SSH filtrada, web UI funcional, firmware atualizado

Status atual: repetidor seguro e funcional, pronto para operação normal e registro no homelab
