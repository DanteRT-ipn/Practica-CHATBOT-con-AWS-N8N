
# 🌐 Diseño y Administración de Redes
## Practica 2 Implementacion de un chtabot en WhatsApp en AWS con n8n
<p align="center">
  <img src="https://1000marcas.net/wp-content/uploads/2025/03/Amazon-Web-Services-Emblem.png" alt="AWS" width="100"/>
  <img src="https://netolink.com/wp-content/uploads/2025/01/n8n.png" alt="n8n" width="100"/>
</p>

##### 👨‍🎓Creado por Ruiz Torres Dante Vladimir 
##### 👨‍🏫Asesorado por  Juárez Hernández Miguel Ángel

# 🌐 Configuración de AWS para Docker, Nginx y Certificado SSL con Certbot usando n8n

Esto es una Guia  paso a paso para desplegar **n8n** para crear un chat bot para WhatsApp en una instancia EC2 de AWS, usando **Docker**, configurando **Nginx** como proxy inverso y asegurando tu dominio con **Certbot**.

---

## 🖥️ Paso 1: Conéctate a la instancia AWS EC2 por SSH

```bash
cd Downloads
ssh -i yourkey.pem ec2-user@publicip
```
###### sustituye el nombre de tus keys.pem y la ip de tu instancia. Recuerda estar en la carpeta donde descargaste tus keys.
---

## 🔄 Paso 2: Actualiza la instancia e instala Docker
> Desde este punto solo vamos a copiar y pegar paso por paso cada comando una vez que hayamos entrado a nuestra instancia por medio de ssh.
```bash
sudo yum update -y
sudo yum install -y docker
```

---

## 🚀 Paso 3: Inicia y habilita el servicio Docker
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

---

## 👤 Paso 4: Agrega tu usuario al grupo Docker

```bash
sudo usermod -aG docker ec2-user
```

> ⚠️ **Importante:** Cierra la sesión y vuelve a entrar para aplicar los cambios.

---

## 🔁 Paso 5: Reconéctate a tu instancia

```bash
ssh -i yourkey.pem ec2-user@publicip
```
###### Recuerda cambiar el nombre de tu key y tu ip.
---

## 🧰 Paso 6: Instala Docker Compose

```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

---
> ⚠️ **AVISO IMPORTANTE antes del Paso 7**

Antes de ejecutar el paso 7, realiza los siguientes pasos para configurar un nombre de dominio dinámico (DDNS) con **NO-IP**:

### 🌐 Configuración de NO-IP

1. Abre tu navegador y entra a 👉 [https://www.noip.com/es-MX](https://www.noip.com/es-MX)
2. 📝 **Crea una cuenta** gratuita con tu correo electrónico.
3. Dirígete al panel de control y entra a la sección **NO-IP Hostnames**.
4. Haz clic en **"Add a Hostname"**:
   - Escribe un nombre personalizado (ej. `tudominio.ddns.net`)
   - Asegúrate de que el campo **IP Address** tenga tu IP pública actual.
   - Guarda la configuración.

5. ✅ **Copia tu nuevo hostname** (por ejemplo: `tudominio.ddns.net`)  
   Luego, continúa con el **Paso 7**, usando este hostname donde se solicite el dominio.

## 🧪 Paso 7: Ejecuta el contenedor Docker de n8n

```bash
sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="your-domain-name" \
-e WEBHOOK_TUNNEL_URL="https://your-domain-name/" \
-e WEBHOOK_URL="https://your-domain-name/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n
```
###### Edita el hostname en la parte de your-domain-name
---

## 🌐 Paso 8: Instala y configura Nginx

```bash
sudo dnf install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

---

## 🔁 Paso 9: Crea configuración de proxy inverso en Nginx

```bash
sudo nano /etc/nginx/conf.d/n8n.conf
```
### ✍️ Paso 9 – Edición con **nano**

En este paso, estaremos utilizando el editor de texto **nano** en la terminal.

📄 Al ejecutar el comando del paso 9, se abrirá un archivo en modo de edición.  
Aquí debes **pegar el contenido del archivo que copiaste previamente** (por ejemplo, un archivo de configuración o certificado).

🔁 Luego, **busca donde aparezca el texto `your domain name`**  
y reemplázalo con el **hostname que creaste en NO-IP**, por ejemplo:  
`tudominio.ddns.net`

💾 Para guardar los cambios:
- Presiona `Ctrl + O` y luego `Enter`
- Para salir del editor, presiona `Ctrl + X`

### Contenido del archivo `n8n.conf`:

```nginx
server {
    listen 80;
    server_name your-domain-name;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;

        # Headers for WebSocket support
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Upgrade $http_upgrade;

        # Additional headers for forwarding client info
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

📥 Guarda y cierra con: `CTRL+O`, `ENTER`, `CTRL+X`

---

## 🔍 Paso 10: Prueba y reinicia Nginx

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

## 🔐 Paso 11: Configura SSL con Certbot


En este paso vamos a **instalar y configurar un certificado SSL** en nuestro servidor usando **Certbot** y **NGINX**.  
⚠️ Es importante ejecutar **uno por uno** los siguientes comandos para asegurarnos de que cada paso se realice correctamente.

---

```bash
sudo dnf install -y certbot python3-certbot-nginx

sudo certbot --nginx -d your-domain-name
"Recuerda sustituir el hostname en este paso"
"Te pedira un correo electronico pon cualquiera y despues **Yes** dos veces"

```

---
 ## 🚀 ¡n8n instalado exitosamente!

Ahora que hemos instalado **n8n** en nuestra instancia, podemos acceder a la sesión de n8n a través de nuestro navegador ingresando la URL configurada (por ejemplo: `https://your-domain-name`).

🛠️ A partir de este momento, ya podemos comenzar a **crear nuestros flujos y chatbots personalizados** utilizando la plataforma de **n8n**.

¡Todo listo para automatizar! 🤖


