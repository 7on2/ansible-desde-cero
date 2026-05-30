# Clase 04: Primeros Pasos con Comandos Ad-Hoc

Los comandos **Ad-Hoc** son instrucciones rápidas y puntuales que se ejecutan directamente en la terminal de una sola línea, sin guardar la automatización en un archivo. Son ideales para tareas rápidas de mantenimiento, comprobación o consultas rápidas de estado en múltiples servidores simultáneamente.

---

## 1. El Archivo de Inventario (`inventory.ini`)

Antes de poder mandar comandos a tus servidores, debes decirle a Ansible cuáles son y cómo agruparlos. Esto se hace a través de un archivo de **Inventario**.

Crea un archivo llamado `inventory.ini` con la siguiente estructura:

```ini
[web]
ubuntu1

[db]
centos1

[all:vars]
ansible_user=ansible
```

### Componentes:
* `[web]` y `[db]`: Son grupos de servidores. Permiten apuntar a un conjunto de máquinas por su rol.
* `ubuntu1` y `centos1`: Son los nombres o IPs de las máquinas gestionadas.
* `[all:vars]`: Define variables aplicables a todas las máquinas de la lista. En este caso, le indica a Ansible que se conecte usando el usuario de SSH llamado `ansible`.

### Comandos de Verificación
Para comprobar que Ansible reconoce correctamente tu inventario, ejecuta:

```bash
# Listar todos los servidores gestionados
ansible all -i inventory.ini --list-hosts

# Listar solo los servidores del grupo web
ansible web -i inventory.ini --list-hosts
```

---

## 2. Sintaxis de los Comandos Ad-Hoc

Un comando ad-hoc sigue siempre esta estructura:

$$\text{ansible } \langle \text{hosts} \rangle \text{ -i } \langle \text{inventario} \rangle \text{ -m } \langle \text{módulo} \rangle \text{ -a } \text{"} \langle \text{argumentos} \rangle \text{"} \text{ [--become]}$$

* **`<hosts>`:** El grupo o IP de servidores (ej. `all`, `web`, `db`, `ubuntu1`).
* **`-i <inventario>`:** Ruta al archivo de inventario (ej. `-i inventory.ini`).
* **`-m <módulo>`:** El módulo de Ansible que ejecutará la acción (ej. `-m ping`, `-m apt`).
* **`-a "<argumentos>"`:** Los parámetros específicos requeridos por el módulo (opcional para algunos módulos como `ping`).
* **`--become`:** Ejecuta el comando usando privilegios de superusuario (`sudo`). Requerido para instalar paquetes o modificar archivos del sistema.

---

## 3. Ejemplos Prácticos de Comandos Ad-Hoc

### 1. Verificar Conectividad (Módulo `ping`)
No realiza un ping ICMP tradicional. En su lugar, verifica que Ansible pueda conectarse por SSH e interactuar con Python en el nodo destino:
```bash
ansible all -i inventory.ini -m ping
```

### 2. Ejecutar Comandos Remotos (Módulo `command`)
Ejecuta un comando del sistema y devuelve la salida:
```bash
ansible web -i inventory.ini -m command -a "uptime"
```

### 3. Instalar un Paquete (Módulo `apt` / `dnf`)
Instala la herramienta `htop` en los nodos Ubuntu utilizando privilegios `sudo`:
```bash
ansible web -i inventory.ini -m apt -a "name=htop state=present" --become
```

### 4. Copiar Archivos (Módulo `copy`)
Envía un archivo local a una ruta remota:
```bash
# Primero creamos un archivo simple local
echo "Hola desde el nodo de control" > hola.txt

# Lo copiamos a todos los servidores administrados
ansible all -i inventory.ini -m copy -a "src=./hola.txt dest=/tmp/hola.txt"
```

### 5. Recopilar Información del Sistema (Módulo `setup` o *Facts*)
Obtiene una lista masiva en formato JSON con todas las características del hardware, red y sistema operativo del servidor:
```bash
ansible web -i inventory.ini -m setup
```

### 6. Reiniciar Servicios (Módulo `service`)
Asegura el reinicio de servicios del sistema:
```bash
ansible web -i inventory.ini -m service -a "name=nginx state=restarted" --become
```

---

## 4. Tabla de Módulos Más Comunes

| Módulo | Acción Principal | Ejemplo de Argumento |
|---|---|---|
| **`ping`** | Comprueba conectividad SSH y Python. | *No requiere* |
| **`command`** | Ejecuta comandos de sistema simples (no soporta pipes ni variables de shell). | `-a "whoami"` |
| **`shell`** | Ejecuta comandos en el shell del host (soporta `<`, `>`, `\|`, `$VAR`). | `-a "cat /etc/hosts \| grep local"` |
| **`copy`** | Copia un archivo local al destino remoto. | `-a "src=origen dest=destino"` |
| **`apt`** | Gestiona paquetes en Debian/Ubuntu. | `-a "name=nginx state=present"` |
| **`service`** | Gestiona servicios del sistema (systemd). | `-a "name=cron state=started"` |
| **`setup`** | Recopila los facts (métricas e información del sistema). | *No requiere* |

---

[Anterior: Clase 03 - Preparación del Entorno](./03-preparacion-laboratorio.md) | [Siguiente: Clase 05 - Primer Playbook](./05-primer-playbook.md)
