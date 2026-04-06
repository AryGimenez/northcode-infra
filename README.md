# NorthCode Infrastructure 🏗️ 🇺🇾

Este repositorio contiene la configuración de **Infraestructura como Código (IaC)** para los servicios de **NorthCode**. Nuestra arquitectura está diseñada para ser escalable, segura y optimizada para el mercado uruguayo, garantizando alta disponibilidad para sistemas críticos y servicios locales.

## 🌐 Arquitectura de Red
Utilizamos un nodo de cómputo en la nube (**DigitalOcean**) protegido por un firewall perimetral y gestionado mediante una red privada segura (VPN).

* **Servidor:** Ubuntu 24.04 LTS (DigitalOcean Droplet).
* **Orquestación:** Docker & Docker Compose V2.
* **VPN:** Wireguard (vía WG-Easy) para acceso administrativo exclusivo.
* **Proxy:** Nginx Proxy Manager para gestión de dominios y certificados SSL (Let's Encrypt).
* **Monitoreo:** Uptime Kuma (Disponibilidad) y Beszel (Rendimiento de Hardware).

## 📂 Estructura de Directorios
Para mantener el orden profesional, el servidor sigue esta jerarquía:
<!> NO se si estabien correjir 
```text 
/home/northcode/
├── infra/                  # Este repositorio (Gestión de Stacks)
│   ├── stacks/
│   │   ├── production/     # VPN, Proxy, Portainer, Monitoreo
│   │   └── staging/        # Demos y entornos de prueba
│   └── docs/               # Diagramas de red y políticas
└── apps/                   # Repositorios de código (clonados aparte)
    ├── snig-app/           # Sistema SNIG (Flutter Web)
    └── vet-core/           # Veterinaria Beltramelli (React/Vite)
```
## 🛠️ Requisitos Previos (Setup del Servidor) <!> Logito de alarma 
Antes de realizar el primer despliegue, es necesario preparar el entorno:

### 1. Crear Red Virtual de Docker <!> Logito de red 
Esta red permite que el Proxy se comunique con las aplicaciones de forma aislada.

```bash
docker network create northcode-net
```
### 2. Configurar Memoria de Intercambio (SWAP) <!> Opicional si el servidor tien poca ram <!> Logito de memoria cansado o algo graciosos 
Vital para asegurar que los procesos de compilación (build) no agoten la RAM física de 2GB.
<!> Esto no lo probe aun en produccion 
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## 🚀 Guía de Despliegue
### Desplegar Infraestructura Base (Producción)
1. Navegar a la carpeta
```bash
cd stacks/production
```
2. Configurar variables: 
```bash
cp .env.example .env && nano .env
```

3. Iniciar servicios: 
```bash
docker compose  up -d
```

### Desplegar Aplicaciones
1. Navegar a la carpeta de la app: 
```bash
cd stacks/staging (o la correspondiente).
```

2. Construir y desplegar: 
```bash 
docker compose up -d --build
```
<!> Implementarlo 

## 🔒 Políticas de Seguridad y Puertos

| Servicio | Puerto | Acceso |
| :--- | :--- | :--- |
| **HTTP/HTTPS** | 80, 443 | Público (Mundo) |
| **VPN Wireguard** | 51820 (UDP) | Público (Mundo) |
| **NPM Admin** | 81 | Privado (Solo VPN) |
| **Portainer** | 9443 | Privado (Solo VPN) |
| **Uptime Kuma** | 3001 | Privado (Solo VPN) |

## 📊 Monitoreo
Contamos con un sistema de alertas proactivo:

**Uptime Kuma:** Monitorea la disponibilidad de las URLs de los clientes y envía notificaciones.

<!> Agregar como link sitio oficial 

https://uptimerobot.com/alternative-to-uptimekuma/?qwerty=Cj0KCQjws83OBhD4ARIsACblj19Htp-vQifpnh_HOXXvruwBe6Kx9gWgw1G-n7rbmui3Y9D4H8FC3R8aAoUREALw_wcB&gad_source=1&gad_campaignid=21727835248&gbraid=0AAAAACQq4QiBv0FVuFZxBj4Qli5QO21YF&gclid=Cj0KCQjws83OBhD4ARIsACblj19Htp-vQifpnh_HOXXvruwBe6Kx9gWgw1G-n7rbmui3Y9D4H8FC3R8aAoUREALw_wcB


**Beszel:** Supervisa el uso de CPU, RAM y Disco para prevenir saturaciones en el Droplet.

## 🔐 Panel de Control Administrativo (Vía VPN)

| Servicio                  | URL de Acceso              | Función                                      |
|---------------------------|----------------------------|----------------------------------------------|
| VPN Admin (WG-Easy)       | http://10.13.13.1:51821    | Crear/Eliminar accesos para el equipo.       |
| Nginx Proxy Manager       | http://10.13.13.1:81       | Gestionar dominios y certificados SSL.       |
| Portainer                 | https://10.13.13.1:9443    | Gestionar contenedores y ver logs.           |
| Uptime Kuma               | http://10.13.13.1:3001     | Estado de webs y alertas.                    |

---

## 🛠️ Paso 1: Configurar tu conexión inicial (Primera vez)

Como todavía no tenés la VPN conectada en el droplet nuevo, tenés que entrar una sola vez por la IP pública para generar tu perfil:

1. Entrá a:  
   `http://192.241.148.226:51821`

2. Poné la contraseña definida en el `.env` (`WG_PASSWORD`)

3. Hacé clic en **"+ New Client"**  
   Nombre: `Ary-Notebook`

4. Descargá el archivo `.conf` o escaneá el QR

5. Conectate desde Linux:
```bash
sudo wg-quick up northcode
```

## 🛡️ Paso 2: El Blindaje (Cerrar la muralla)

Ir a: DigitalOcean → Networking → Firewalls

Configurar reglas de entrada (Inbound Rules):

SSH (22) → Solo tu IP (o abierto si usás SSH keys)
HTTP (80) → All IPv4 / All IPv6
HTTPS (443) → All IPv4 / All IPv6
WireGuard (51820 UDP) → All IPv4

## ❌ Eliminar TODO lo demás

Resultado:

Nadie puede acceder a:

```
http://192.241.148.226:81 
```

Solo vos vía VPN.

## 📊 Monitoreo interno (Uptime Kuma)

Para monitorear contenedores sin salir a internet, usar el nombre del contenedor:

**SNIG:**
http://snig-web-demo:80

**Veterinaria:**
http://vet-core-app:80
💡 ¿Por qué esto es pro?

Si falla el firewall externo, igual recibís alertas porque el monitoreo es interno (red Docker).

## 📱 Tip para celular
Instalar app WireGuard
Escanear QR desde WG-Easy
Conectarte desde cualquier lugar

### 👉 Flujo:

Llega alerta (Telegram)
Activás VPN
Entrás a Portainer
Reiniciás contenedor

Todo desde el celular 🚀
