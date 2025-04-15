# 🌐 Load Balancer com NGINX na AWS – XPTO Project

Este repositório documenta o processo de criação de um ambiente com balanceamento de carga usando **NGINX** em instâncias **Ubuntu EC2** na **AWS**, com as instâncias:

- `XPTO-1` – Backend 1
- `XPTO-2` – Backend 2
- `XPTO-3` – Backend 3
- `Load-Balancer-XPTO` – Load Balancer com NGINX

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
- `Load-Balancer-XPTO`

> Certifique-se de que as instâncias possuem **endereços IP públicos**.

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
    server XPTO-1; # Altere pelo IP privado da sua instância XPTO-1
    server XPTO-2; # Altere pelo IP privado da sua instância XPTO-2
    server XPTO-3; # Altere pelo IP privado da sua instância XPTO-3
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

## 📦 Extras

- Você pode customizar o HTML de cada instância utlizando `sudo nano /var/www/html/index.html`.


---
