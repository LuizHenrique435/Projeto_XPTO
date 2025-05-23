# 🔒 VPN com OpenVPN na AWS – XPTO Project

Este repositório documenta o processo de criação de uma rede VPN ponto-a-ponto utilizando **OpenVPN** em instâncias **Ubuntu EC2** na **AWS**, com as instâncias:

- `XPTO-VPN-Server` – Servidor VPN
- `XPTO-VPN-Client` – Cliente VPN

---

## 🛠️ Passo a Passo

### 1. Criar 2 instâncias EC2 Ubuntu

No console da AWS:

- Tipo de instância: `t2.micro`
- AMI: `Ubuntu Server 22.04 LTS`
- Grupo de Segurança:
  - Porta `22` (SSH)
  - Porta `5000/UDP` (OpenVPN)

Nomeie as instâncias como:
- `XPTO-VPN-Server`
- `XPTO-VPN-Client`

> ⚠️ **Garanta que ambas as instâncias possuem IP público ou Elastic IP.**

---

### 2. Instalar OpenVPN e Easy-RSA no servidor

Acesse o servidor:

```bash
ssh -i "sua-chave.pem" ubuntu@IP_DO_SERVIDOR
````

Instale OpenVPN e Easy-RSA:

```bash
sudo apt update
sudo apt install -y openvpn easy-rsa
```

Crie a pasta para os certificados:

```bash
sudo mkdir -p /etc/openvpn/easy-rsa
sudo cp -r /usr/share/easy-rsa/* /etc/openvpn/easy-rsa/
cd /etc/openvpn/easy-rsa
```

Inicialize e gere os certificados:

```bash
sudo ./easyrsa init-pki
sudo ./easyrsa build-ca nopass
sudo ./easyrsa gen-dh
sudo ./easyrsa gen-req server nopass
sudo ./easyrsa sign-req server server
sudo ./easyrsa gen-req client1 nopass
sudo ./easyrsa sign-req client client1
```

---

### 3. Arquivo de configuração do servidor

Crie o arquivo `/etc/openvpn/server.conf`:

```bash
sudo nano /etc/openvpn/server.conf
```

Conteúdo:

```ini
ifconfig 192.168.0.100 192.168.0.200
tls-server
ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key
dh /etc/openvpn/easy-rsa/pki/dh.pem
port 5000
proto udp
keepalive 10 120
persist-key
persist-tun
verb 4
float
comp-lzo
cipher AES-256-CBC
```

---

### 4. Transferência dos certificados para o cliente

No servidor, exiba o conteúdo dos certificados:

```bash
cat /etc/openvpn/easy-rsa/pki/ca.crt
cat /etc/openvpn/easy-rsa/pki/issued/client1.crt
cat /etc/openvpn/easy-rsa/pki/private/client1.key
```

No cliente (`XPTO-VPN-Client`), acesse via SSH:

```bash
ssh -i "sua-chave.pem" ubuntu@IP_DO_CLIENTE
```

Crie os arquivos no cliente:

```bash
sudo nano /etc/openvpn/ca.crt
# Cole o conteúdo do ca.crt

sudo nano /etc/openvpn/client1.crt
# Cole o conteúdo do client1.crt

sudo nano /etc/openvpn/client1.key
# Cole o conteúdo do client1.key
```

---

### 5. Arquivo de configuração do cliente

Crie `/etc/openvpn/client.conf`:

```bash
sudo nano /etc/openvpn/client.conf
```

Conteúdo:

```ini
dev tun
ifconfig 192.168.0.200 192.168.0.100
remote IP_DO_SERVIDOR
tls-client
ca /etc/openvpn/ca.crt
cert /etc/openvpn/client1.crt
key /etc/openvpn/client1.key
port 5000
proto udp
keepalive 10 120
persist-key
persist-tun
verb 4
float
comp-lzo
cipher AES-256-CBC
```

Substitua `IP_DO_SERVIDOR` pelo IP público do servidor VPN.

---

### 6. Inicializar a VPN

No servidor:

```bash
sudo openvpn --config /etc/openvpn/server.conf
```

No cliente:

```bash
sudo openvpn --config /etc/openvpn/client.conf
```

---

## 🧪 Testando a VPN

* No cliente, execute:

```bash
ip a
```

Veja se aparece uma interface `tun0` com IP `192.168.0.200`.

* No servidor:

```bash
ip a
```

Veja se a interface `tun0` tem o IP `192.168.0.100`.

* Teste a conexão:

```bash
ping 192.168.0.100 # Do cliente para o servidor
ping 192.168.0.200 # Do servidor para o cliente
```

Se receber respostas, a VPN está funcionando corretamente.

---
