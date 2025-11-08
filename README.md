# Mi primer servidor web con Linux y Nginx (VirtualBox)

Proyecto de aprendizaje personal en 1º de ASIR.

## Objetivo

- Instalar Ubuntu Server en una máquina virtual con VirtualBox.
- Configurar la red para que la VM actúe como servidor.
- Instalar y configurar Nginx como servidor web.
- Acceder al servidor desde el equipo anfitrión (Windows) usando el navegador.

## Entorno

- **Host**: Windows 10/11
- **Virtualización**: VirtualBox
- **VM**:
  - Ubuntu Server 24.04 LTS
  - 4 GB RAM
  - 25 GB disco
  - Red: NAT + reenvío de puertos (host 8080 -> guest 80)

## Pasos realizados

1. Creación de la máquina virtual en VirtualBox.
2. Instalación de Ubuntu Server (idioma inglés, teclado español).
3. Configuración de red con DHCP (interfaz `enp0s3`).
4. Actualización del sistema:
   ```bash
   sudo apt update
   sudo apt upgrade -y
