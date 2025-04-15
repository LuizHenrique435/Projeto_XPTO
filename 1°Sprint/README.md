# 🌐 Load Balancer com NGINX na AWS – XPTO Project

Este repositório documenta o processo de criação de um ambiente com balanceamento de carga usando **NGINX** em instâncias **Ubuntu EC2** na **AWS**, com as instâncias:

- `XPTO-1` – Backend 1
- `XPTO-2` – Backend 2
- `XPTO-3` – Backend 3
- `XPTO-Balancer` – Load Balancer com NGINX

---

## 🛠️ Passo a Passo

### 1. Criar 4 instâncias EC2 Ubuntu

No console da AWS:

- Tipo de instância: `t2.micro`
- AMI: `Ubuntu Server 22.04 LTS`
- Grupo de Segurança:
  - Porta `22` (SSH)
  - Porta `80` (HTTP)

Nomeie-as como:
- `XPTO-1`
- `XPTO-2`
- `XPTO-3`
- `XPTO-Balancer`

> Certifique-se de que as instâncias possuem **endereços IP públicos ou Elastic IPs**.

---

### 2. Instalar e configurar NGINX nas instâncias XPTO-1, 2 e 3

Conecte-se a cada instância:

```bash
ssh -i "sua-chave.pem" ubuntu@IP_DA_INSTANCIA
```

Instale o NGINX:

```bash
sudo apt update
sudo apt install -y nginx
```

Crie um HTML identificador simples:

```bash
echo "Instância XPTO-1" | sudo tee /var/www/html/index.html
# Repita para XPTO-2 e XPTO-3 mudando o nome
```

Reinicie o serviço:

```bash
sudo systemctl restart nginx
```

---

### 3. Configurar o Load Balancer (XPTO-Balancer)

Acesse a instância XPTO-Balancer via SSH e instale o NGINX:

```bash
sudo apt update
sudo apt install -y nginx
```

Edite o arquivo de configuração:

```bash
sudo nano /etc/nginx/sites-available/default
```

Substitua o conteúdo pelo seguinte:

```nginx
upstream backends {
    server 54.82.70.202; # Altere pelos IPS privados das suas instâncias
    server 3.82.199.128;
    server 54.226.6.101;
}

server {
    listen 80;

    location / {
        proxy_pass http://backends;
    }
}
```

Reinicie o NGINX:

```bash
sudo systemctl restart nginx
```

---

### 4. (Opcional) Configurar o arquivo hosts localmente

Para facilitar o acesso via nomes personalizados, edite o arquivo `hosts` no seu computador:

```bash
sudo nano /etc/hosts
```

Adicione as linhas:

```plaintext
54.82.70.202      xpto-1 # Altere pelos IPS públicos das suas instâncias
3.82.199.128      xpto-2
54.226.6.101      xpto-3
54.167.12.188     xpto-balancer
```

---

## 🧪 Testando

Abra o navegador e acesse o IP público do `XPTO-Balancer`.  
Atualize a página várias vezes e observe que o conteúdo muda entre `XPTO-1`, `XPTO-2` e `XPTO-3`, confirmando o balanceamento de carga.

---

## 📦 Extras

- Você pode customizar o HTML de cada instância utlizando `sudo nano /var/www/html/index.html`.


---
