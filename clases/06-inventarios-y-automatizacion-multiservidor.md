# Sesión 6: Inventarios y Automatización Multi-Servidor

## Objetivo de la clase

Al finalizar, el estudiante podrá modelar una infraestructura con inventarios estáticos, ejecutar automatizaciones sobre varios grupos, instalar MariaDB en un nodo separado y utilizar handlers para reiniciar servicios solamente cuando una configuración cambie.

## Punto de partida

Un comando contra un solo servidor demuestra que Ansible funciona. Un inventario bien organizado permite administrar una infraestructura.

El inventario responde tres preguntas:

| Pregunta | Elemento |
|---|---|
| ¿Qué servidores existen? | Hosts |
| ¿Qué función cumple cada servidor? | Grupos |
| ¿Cómo se conecta Ansible? | Variables |

Arquitectura usada durante la clase:

```text
control
├── web01  -> Nginx
├── web02  -> Nginx
└── db01   -> MariaDB
```

## 1. Inventarios estáticos

Un inventario estático es un archivo mantenido manualmente. Es adecuado para laboratorios y entornos con hosts relativamente estables.

```ini
[web]
web01 ansible_host=10.10.10.21
web02 ansible_host=10.10.10.22

[db]
db01 ansible_host=10.10.10.31

[linux:children]
web
db

[linux:vars]
ansible_user=ansible
ansible_python_interpreter=/usr/bin/python3
```

`web01` es el alias que se utiliza en Ansible. `ansible_host` es la dirección real. Separarlos permite cambiar una IP sin cambiar los playbooks.

`[linux:children]` crea un grupo padre. Ejecutar contra `linux` alcanza los grupos `web` y `db`.

### Inspección del inventario

```bash
ansible-inventory -i inventory.ini --graph
ansible-inventory -i inventory.ini --list
ansible all -i inventory.ini --list-hosts
ansible linux -i inventory.ini -m ansible.builtin.ping
```

`--graph` confirma la pertenencia a grupos. `--list` muestra la estructura y las variables resueltas. `--list-hosts` confirma el alcance antes de ejecutar.

## 2. Patrones de selección

| Patrón | Resultado |
|---|---|
| `all` | Todos los hosts |
| `web` | Solo servidores web |
| `web01` | Un servidor |
| `web:db` | Unión de grupos |
| `linux:!db` | Linux excepto base de datos |

```bash
ansible 'web:db' -i inventory.ini --list-hosts
ansible 'linux:!db' -i inventory.ini --list-hosts
```

Las comillas evitan que el shell interprete caracteres especiales.

## 3. Automatización multi-servidor

Ansible ejecuta una tarea en todos los hosts seleccionados antes de avanzar a la siguiente tarea. Con dos nodos web, la instalación de Nginx se aplica a ambos sin duplicar código.

```yaml
---
- name: Preparar servidores web
  hosts: web
  become: true
  tasks:
    - name: Instalar Nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true
```

### Alcance controlado

```bash
ansible-playbook -i inventory.ini site.yml --limit web01
ansible-playbook -i inventory.ini site.yml --limit web
```

`--limit` restringe una ejecución sin modificar el playbook.

`serial: 1` procesa un host por lote. Es útil cuando no se debe interrumpir todo un servicio al mismo tiempo.

```yaml
- name: Actualizar servidores web uno por uno
  hosts: web
  serial: 1
  become: true
```

## 4. MariaDB en un servidor separado

La base de datos tiene requisitos y riesgos diferentes al servidor web. El grupo `db` recibe un play independiente.

```yaml
- name: Preparar servidor de base de datos
  hosts: db
  become: true
  tasks:
    - name: Instalar MariaDB y el cliente Python
      ansible.builtin.apt:
        name:
          - mariadb-server
          - python3-pymysql
        state: present
        update_cache: true

    - name: Iniciar MariaDB
      ansible.builtin.service:
        name: mariadb
        state: started
        enabled: true
```

`python3-pymysql` permite utilizar posteriormente los módulos de base de datos de la colección `community.mysql`.

## 5. Handlers

Un handler es una tarea que se ejecuta cuando otra tarea informa un cambio y la notifica.

```yaml
tasks:
  - name: Configurar dirección de escucha
    ansible.builtin.lineinfile:
      path: /etc/mysql/mariadb.conf.d/50-server.cnf
      regexp: '^bind-address'
      line: 'bind-address = 0.0.0.0'
      backup: true
    notify: Reiniciar MariaDB

handlers:
  - name: Reiniciar MariaDB
    ansible.builtin.service:
      name: mariadb
      state: restarted
```

Primera ejecución:

```text
Configurar dirección de escucha ... changed
RUNNING HANDLER [Reiniciar MariaDB] ... changed
```

Segunda ejecución:

```text
Configurar dirección de escucha ... ok
```

El handler no aparece porque no hubo cambio.

## Resumen de comandos

```bash
ansible-inventory -i inventory.ini --graph
ansible all -i inventory.ini --list-hosts
ansible linux -i inventory.ini -m ansible.builtin.ping
ansible-playbook -i inventory.ini site.yml --syntax-check
ansible-playbook -i inventory.ini site.yml --check
ansible-playbook -i inventory.ini site.yml
```

## Errores frecuentes

| Error | Causa probable | Solución |
|---|---|---|
| No hosts matched | Grupo mal escrito | Revisar `--graph` y `hosts:` |
| UNREACHABLE | IP, usuario o llave incorrectos | Probar SSH y variables del inventario |
| MariaDB se instala en web | Alcance incorrecto | Separar plays y usar `hosts: db` |
| Handler no se ejecuta | No hubo cambio o no coincide el nombre | Revisar `changed` y `notify` |
| Handler se ejecuta siempre | Tarea no idempotente | Usar módulos declarativos |

## Cierre

El inventario convierte una lista de equipos en una estructura operable. Los grupos separan responsabilidades. Los plays aplican estados por grupo. Los handlers conectan cambios reales con acciones necesarias.
