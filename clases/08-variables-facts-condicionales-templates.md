# Sesión 7: Variables, Facts, Condicionales y Templates

## Objetivo de la clase

Al finalizar, el estudiante podrá:

1. Explicar por qué un handler depende del estado `changed`.
2. Diferenciar una contraseña SSH de una contraseña de escalamiento con `sudo`.
3. Conectarse primero con usuario y contraseña, y después mediante una llave SSH.
4. Crear desde cero `ansible.cfg`, un inventario y la estructura de variables.
5. Definir variables en el playbook, `group_vars/` y `host_vars/`.
6. Recopilar y utilizar facts de Ubuntu y CentOS.
7. Ejecutar tareas diferentes mediante condiciones `when`.
8. Repetir una misma tarea mediante `loop`.
9. Generar archivos personalizados con templates Jinja2.
10. Notificar un handler cuando una configuración generada cambie.
11. Validar funcionamiento e idempotencia en dos distribuciones.

## Flujo de la sesión

```text
Inventario
    |
    v
Variables del grupo y del host
    |
    v
Facts descubiertos por Ansible
    |
    v
Condiciones y loops
    |
    v
Template Jinja2
    |
    v
Archivo generado
    |
    v
Handler si hubo cambio
    |
    v
Validación
```

Una automatización reutilizable separa tres elementos:

```text
Lógica  -> tareas, condiciones y handlers
Datos   -> variables y facts
Salida  -> archivos generados desde templates
```

## 1. Preparar el laboratorio

### Arquitectura

```text
ubuntu-c   -> nodo de control con Ansible

ubuntu1    -> nodo Ubuntu administrado
centos1    -> nodo CentOS Stream administrado
```

### Iniciar el laboratorio

```bash
cd ~/diveintoansible-lab
docker compose up -d
docker compose ps
```

Ingresar al nodo de control:

```bash
docker exec -it -u ansible ubuntu-c bash
```

Validar:

```bash
whoami
hostname
ansible --version
```

## 2. Repaso de handlers

### Problema que resuelve un handler

Una configuración puede cambiar o puede permanecer igual. Reiniciar un servicio en cada ejecución produce interrupciones innecesarias.

```text
Tarea normal:
se procesa cada vez que Ansible llega a ella

Handler:
se procesa solamente cuando una tarea cambia y lo notifica
```

### Sin handler

```yaml
- name: Copiar configuración
  ansible.builtin.copy:
    src: aplicacion.conf
    dest: /etc/aplicacion.conf

- name: Reiniciar aplicación
  ansible.builtin.service:
    name: aplicacion
    state: restarted
```

El reinicio se solicita siempre, incluso cuando el archivo ya era correcto.

### Con handler

```yaml
- name: Copiar configuración
  ansible.builtin.copy:
    src: aplicacion.conf
    dest: /etc/aplicacion.conf
  notify: Reiniciar aplicación

handlers:
  - name: Reiniciar aplicación
    ansible.builtin.service:
      name: aplicacion
      state: restarted
```

### Flujo del handler

```text
copy compara origen y destino
             |
       +-----+-----+
       |           |
      ok        changed
       |           |
  no notifica     notify
                   |
                   v
             handler pendiente
                   |
                   v
           handler al final del play
```

### Tres piezas de un handler

| Pieza | Función |
|---|---|
| `notify` | Notifica que un handler debe ejecutarse |
| `handlers` | Contiene la lista de handlers |
| `name` | Identificador que debe coincidir con `notify` |

### Momento de ejecución

Por defecto, Ansible ejecuta los handlers pendientes al final del play.

Para adelantar los handlers:

```yaml
- name: Ejecutar handlers pendientes
  ansible.builtin.meta: flush_handlers
```

## 3. Crear el proyecto desde cero

Estructura del proyecto:

```text
~/curso-ansible/sesion04/
|-- ansible.cfg
|-- inventory.ini
|-- group_vars/
|   |-- all.yml
|   `-- web.yml
|-- host_vars/
|   |-- ubuntu1.yml
|   `-- centos1.yml
|-- templates/
|   |-- index.html.j2
|   `-- pit-site.conf.j2
|-- 01-explorar-facts.yml
|-- 02-desplegar-sitio.yml
`-- 99-limpieza.yml
```

## 4. El inventario

```ini
[web]
ubuntu1
centos1

[ubuntu]
ubuntu1

[redhat]
centos1

[linux:children]
ubuntu
redhat

[linux:vars]
ansible_user=ansible
```

`[linux:children]` crea un grupo padre cuyos miembros son los hosts de `ubuntu` y `redhat`.

## 5. Diferencia entre `-k` y `-K`

| Parámetro | Nombre largo | Solicita |
|---|---|---|
| `-k` | `--ask-pass` | Contraseña de conexión SSH |
| `-K` | `--ask-become-pass` | Contraseña para escalar con `sudo` |
| `-u` | `--user` | Usuario remoto |
| `-b` | `--become` | Activa escalamiento de privilegios |

Dos etapas:

```text
Etapa 1: SSH
ubuntu-c
   |
   | usuario ansible + contraseña o llave
   v
cuenta ansible en ubuntu1

Etapa 2: become
cuenta ansible
   |
   | sudo + contraseña de become
   v
root
```

## 6. Llaves SSH

### Crear una llave dedicada

```bash
ssh-keygen \
  -t ed25519 \
  -f ~/.ssh/id_ed25519_pit \
  -C "curso-pit" \
  -N ""
```

| Opción | Función |
|---|---|
| `-t ed25519` | Utiliza el algoritmo Ed25519 |
| `-f` | Define la ruta y nombre del par |
| `-C` | Agrega un comentario identificador |
| `-N ""` | Crea la llave sin passphrase para el laboratorio |

### Copiar la llave pública

```bash
ssh-copy-id \
  -o PubkeyAuthentication=no \
  -o PreferredAuthentications=password \
  -i ~/.ssh/id_ed25519_pit.pub \
  ansible@ubuntu1
```

### Validar la llave

```bash
ssh -i ~/.ssh/id_ed25519_pit ansible@ubuntu1 hostname
ssh -i ~/.ssh/id_ed25519_pit ansible@centos1 hostname
```

## 7. Variables en Ansible

### Definición

Una variable es un nombre que representa un valor.

```yaml
http_port: 8080
```

Se referencia con Jinja2:

```yaml
"{{ http_port }}"
```

### Tipos de datos

```yaml
nombre: "Portal PIT"
puerto: 8080
habilitado: true

paquetes:
  - nginx
  - curl

aplicacion:
  nombre: "Portal PIT"
  puerto: 8080
```

| Tipo | Ejemplo |
|---|---|
| Texto | `"Portal PIT"` |
| Número | `8080` |
| Booleano | `true` |
| Lista | `- nginx` |
| Diccionario | `nombre: ...`, `puerto: ...` |

### Ubicaciones principales

| Ubicación | Alcance |
|---|---|
| `vars:` | Play actual |
| `group_vars/all.yml` | Todos los hosts |
| `group_vars/web.yml` | Hosts del grupo `web` |
| `host_vars/ubuntu1.yml` | Solo `ubuntu1` |
| `-e` | Valor enviado al ejecutar |
| Facts | Datos descubiertos en cada host |

## 8. Facts

### Definición

Los facts son variables que Ansible descubre al conectarse al host.

```text
Distribución
Versión
Familia del sistema
Hostname
Arquitectura
CPU
Memoria
Interfaces
Direcciones IP
```

### Recopilación automática

Un play recopila facts de forma predeterminada:

```yaml
- name: Ejemplo
  hosts: all
  gather_facts: true
```

Si no se necesitan:

```yaml
gather_facts: false
```

### Explorar con `setup`

```bash
ansible ubuntu1 -m ansible.builtin.setup
ansible linux -m ansible.builtin.setup -a "filter=ansible_distribution*"
ansible linux -m ansible.builtin.setup -a "filter=ansible_os_family"
ansible linux -m ansible.builtin.setup -a "filter=ansible_memtotal_mb"
```

### Facts utilizados

| Fact | Contenido |
|---|---|
| `ansible_hostname` | Hostname real |
| `ansible_distribution` | Ubuntu, CentOS, etc. |
| `ansible_distribution_version` | Versión |
| `ansible_os_family` | Debian o RedHat |
| `ansible_architecture` | Arquitectura |
| `ansible_processor_vcpus` | CPU virtuales |
| `ansible_memtotal_mb` | Memoria total |

## 9. Condicionales con `when`

`when` indica que una tarea se ejecuta solamente si la expresión es verdadera.

```yaml
- name: Tarea para Debian
  ...
  when: ansible_os_family == "Debian"
```

**No usar llaves dentro de `when`**:

Correcto:

```yaml
when: ansible_os_family == "Debian"
```

Evitar:

```yaml
when: "{{ ansible_os_family }}" == "Debian"
```

`when` ya interpreta expresiones Jinja2.

### Condiciones frecuentes

```yaml
when: ansible_os_family == "Debian"
when: environment_name == "produccion"
when: ansible_memtotal_mb >= 1024
```

Varias condiciones (todas deben cumplirse):

```yaml
when:
  - ansible_os_family == "Debian"
  - environment_name == "produccion"
```

Condición con `or`:

```yaml
when: ansible_distribution == "Ubuntu" or ansible_distribution == "Debian"
```

## 10. Repetición con `loop`

### Sin loop

```yaml
- name: Crear /opt/pit
  ansible.builtin.file:
    path: /opt/pit
    state: directory

- name: Crear /opt/pit/logs
  ansible.builtin.file:
    path: /opt/pit/logs
    state: directory
```

### Con loop

```yaml
- name: Crear directorios
  ansible.builtin.file:
    path: "{{ item }}"
    state: directory
  loop:
    - /opt/pit
    - /opt/pit/logs
```

`item` contiene el elemento actual.

## 11. Templates Jinja2

### Definición

Un template es un archivo de texto que contiene marcadores. Ansible reemplaza los marcadores antes de copiar el resultado al host.

### Diferencia entre `copy` y `template`

| Módulo | Comportamiento |
|---|---|
| `copy` | Copia contenido sin procesar Jinja2 desde un archivo |
| `template` | Renderiza variables, condiciones y loops antes de copiar |

### Sintaxis esencial

Variable:

```jinja2
{{ app_name }}
```

Condicional:

```jinja2
{% if environment_name == "produccion" %}
Contenido de produccion
{% else %}
Contenido de desarrollo
{% endif %}
```

Loop:

```jinja2
{% for directorio in common_directories %}
{{ directorio }}
{% endfor %}
```

Comentario:

```jinja2
{# Este texto no aparece en el resultado #}
```

## 12. Resumen de comandos

```bash
# Laboratorio
cd ~/diveintoansible-lab
docker compose up -d
docker exec -it -u ansible ubuntu-c bash

# Configuración e inventario
ansible --version
ansible-config dump --only-changed
ansible-inventory --graph
ansible-inventory --host ubuntu1

# Conexión y become
ansible linux -m ansible.builtin.ping
ansible linux -m ansible.builtin.command -a "whoami"
ansible linux -b -m ansible.builtin.command -a "whoami" --ask-become-pass

# Facts
ansible linux -m ansible.builtin.setup -a "filter=ansible_distribution*"
ansible linux -m ansible.builtin.setup -a "filter=ansible_os_family"

# Playbook
ansible-playbook 02-desplegar-sitio.yml --syntax-check
ansible-playbook 02-desplegar-sitio.yml --list-hosts
ansible-playbook 02-desplegar-sitio.yml --list-tasks
ansible-playbook 02-desplegar-sitio.yml --check --ask-become-pass
ansible-playbook 02-desplegar-sitio.yml --ask-become-pass
ansible-playbook 02-desplegar-sitio.yml --ask-become-pass
```
