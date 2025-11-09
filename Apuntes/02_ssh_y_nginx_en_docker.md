````markdown
# 03 – SSH, Nginx en Docker y puertos (laboratorio en VM)

En esta sesión he montado un entorno bastante completo:
- Ubuntu Server en una máquina virtual (VirtualBox)
- Acceso remoto por **SSH**
- Servidor web **Nginx**:
  - uno “normal” instalado en la VM (puerto 80 → 8080 en el host)
  - otro dentro de un contenedor **Docker** (puerto 8081)
- Una página HTML personalizada servida desde el contenedor
- Todo accesible desde mi PC Windows

---

## 1. Acceso por SSH a la VM

### 1.1 Instalar y activar el servidor SSH en Ubuntu

En la VM (Ubuntu Server):

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable ssh --now
sudo systemctl status ssh
````

Comprobación de que está escuchando en el puerto 22:

```bash
sudo ss -tulpn | grep ssh
# o
sudo ss -tulpn | grep ':22'
```

Debe aparecer `LISTEN` en el puerto `22` con el proceso `sshd`.

---

### 1.2 Reenvío de puertos SSH en VirtualBox

En VirtualBox, con la VM usando **Red: NAT**, he añadido una regla en
**Configuración → Red → Adaptador 1 → Avanzadas → Reenvío de puertos**:

* **Nombre:** `SSH`
* **Protocolo:** `TCP`
* **Puerto anfitrión (host):** `2222`
* **Puerto invitado (guest):** `22`

Resultado:
`Windows 127.0.0.1:2222 → VM 22 (SSH)`

---

### 1.3 Conexión desde Windows por SSH

Desde una terminal de Windows (PowerShell / CMD):

```bash
ssh chiburruso@127.0.0.1 -p 2222
```

* Usuario: el de la VM (`chiburruso`).
* Primera vez: responder `yes` a la huella (`Are you sure you want to continue connecting?`).
* Luego introducir la contraseña del usuario de la VM.

A partir de aquí puedo administrar la VM desde Windows usando SSH
(copiar/pegar mucho más cómodo que en la consola de VirtualBox).

---

## 2. Repaso rápido de Docker + Nginx

### 2.1 Comandos básicos de Docker

Ver contenedores activos:

```bash
sudo docker ps
```

Ver **todos** los contenedores (incluidos parados):

```bash
sudo docker ps -a
```

Parar un contenedor:

```bash
sudo docker stop NOMBRE_CONTENEDOR
```

Eliminar un contenedor:

```bash
sudo docker rm NOMBRE_CONTENEDOR
# o a lo bestia:
sudo docker rm -f NOMBRE_CONTENEDOR
```

---

### 2.2 Contenedor `docker-nginx` escuchando en el puerto 8081

He creado un contenedor de Nginx con estas características:

* Nombre: `docker-nginx`
* Publica el puerto **80 del contenedor** como **8081 en la VM**
* Se reinicia automáticamente al arrancar la VM (`--restart unless-stopped`)

Comando:

```bash
sudo docker run -d --name docker-nginx \
  --restart unless-stopped \
  -p 8081:80 nginx
```

Comprobar que está arriba:

```bash
sudo docker ps
```

Debe aparecer algo como:

```text
PORTS
0.0.0.0:8081->80/tcp
```

Desde la propia VM puedo probar:

```bash
curl http://127.0.0.1:8081
```

---

## 3. Página HTML personalizada servida por Nginx en Docker

Por defecto, Nginx muestra la página de “Welcome to nginx”.
He creado mi propia página y la he montado dentro del contenedor usando un volumen.

### 3.1 Crear el HTML en la VM

```bash
mkdir -p ~/miweb-nginx
nano ~/miweb-nginx/index.html
```

Contenido de ejemplo:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Mi primer servidor Docker</title>
  </head>
  <body>
    <h1>Hola, soy Chiburruso 😎</h1>
    <p>Este Nginx está corriendo en un contenedor Docker dentro de mi VM.</p>
  </body>
</html>
```

Guardar y salir de `nano`.

---

### 3.2 Recrear el contenedor montando el HTML como volumen

Primero me aseguro de borrar el contenedor anterior:

```bash
sudo docker rm -f docker-nginx
```

Luego lo vuelvo a crear, pero montando mi `index.html`:

```bash
sudo docker run -d \
  --name docker-nginx \
  --restart unless-stopped \
  -p 8081:80 \
  -v ~/miweb-nginx/index.html:/usr/share/nginx/html/index.html:ro \
  nginx
```

* `-v origen:destino:ro`

  * **origen:** archivo en la VM (`~/miweb-nginx/index.html`)
  * **destino:** archivo que usa Nginx dentro del contenedor
    (`/usr/share/nginx/html/index.html`)
  * `:ro` → solo lectura desde el contenedor

Comprobación desde la VM:

```bash
curl http://127.0.0.1:8081
```

Debería mostrar el HTML personalizado.

---

## 4. Resumen de puertos (host, VM y contenedor)

* **Nginx “normal” (instalado en la VM):**

  * VM: escucha en `80`
  * VirtualBox: `Host 8080 → Guest 80`
  * Navegador Windows: `http://127.0.0.1:8080`

* **Nginx en Docker (`docker-nginx`):**

  * Contenedor: escucha en `80`
  * Docker: `-p 8081:80` → expone `8081` en la VM
  * VirtualBox: `Host 8081 → Guest 8081`
  * Navegador Windows: `http://127.0.0.1:8081`

* **SSH hacia la VM:**

  * VM: escucha en `22`
  * VirtualBox: `Host 2222 → Guest 22`
  * Cliente SSH en Windows:

    ```bash
    ssh chiburruso@127.0.0.1 -p 2222
    ```