# Ansible desde Cero: Automatiza Linux y Servidores

Aprende Ansible desde sus fundamentos teóricos, pasando por el lenguaje YAML y los comandos ad-hoc, hasta desplegar tu primer playbook automatizado e idempotente sobre un laboratorio multi-servidor en Docker.

## Ruta De Aprendizaje

| Bloque | Tema | Ir |
|---|---|---|
| 01 | ¿Qué es Ansible? Conceptos y Arquitectura | [Abrir](./clases/01-introduccion-ansible.md) |
| 02 | Introducción al Lenguaje YAML | [Abrir](./clases/02-introduccion-yaml.md) |
| 03 | Preparación del Entorno y Laboratorio | [Abrir](./clases/03-preparacion-laboratorio.md) |
| 04 | Primeros Pasos con Comandos Ad-Hoc | [Abrir](./clases/04-comandos-ad-hoc.md) |
| 05 | Desarrollo de tu Primer Playbook | [Abrir](./clases/05-primer-playbook.md) |
| 07 | Inventarios y Automatización Multi-Servidor | [Abrir](./clases/07-inventarios-y-automatizacion-multiservidor.md) |
| 08 | Variables, Facts, Condicionales y Templates | [Abrir](./clases/08-variables-facts-condicionales-templates.md) |
| 09 | Roles, Monitoreo, Seguridad y Backups | [Abrir](./clases/09-roles-monitoreo-seguridad-backups.md) |

### Ruta Rápida Del Programa

| Etapa | Enfoque | Resultado esperado |
|:---:|---|---|
| **1** | ¿Qué es Ansible y por qué automatizar? | Entender el funcionamiento del modelo *Push* por SSH y las ventajas de la filosofía declarativa. |
| **2** | Introducción al lenguaje YAML | Escribir y estructurar variables, listas y mapas en formato YAML sin errores de indentación. |
| **3** | Preparación del entorno y laboratorio | Levantar una infraestructura multi-servidor interconectada con Docker Compose para realizar pruebas. |
| **4** | Primeros pasos con comandos Ad-Hoc | Gestionar e inspeccionar servidores remotos en tiempo real mediante comandos rápidos de una línea. |
| **5** | Desarrollo de tu primer Playbook | Escribir y ejecutar un playbook de despliegue para Nginx, asegurando que sea repetible e idempotente. |
| **6** | Inventarios y automatización multi-servidor | Modelar una infraestructura con inventarios estáticos, ejecutar automatizaciones sobre varios grupos, instalar servicios en nodos separados y utilizar handlers para reiniciar servicios. |
| **7** | Variables, Facts, Condicionales y Templates | Definir variables en el playbook, `group_vars/` y `host_vars/`, recopilar y utilizar facts, ejecutar tareas diferentes mediante condiciones `when`, repetir tareas con `loop` y generar archivos personalizados con templates Jinja2. |
| **8** | Roles, Monitoreo, Seguridad y Backups | Crear roles reutilizables con `ansible-galaxy role init`, instalar Node Exporter, cifrar variables con Ansible Vault, auditar SSH y firewall, crear backups verificables y ejecutar un laboratorio integrador. |

## Laboratorios

| Laboratorio | Descripción | Ir |
|---|---|---|
| 01. Autobiografía YAML | Práctica guiada de estructuración, mapas, listas y formato en YAML. | [Abrir](./laboratorios/01-autobiografia-yaml.md) |
| 02. Comandos Ad-Hoc | Pruebas de conectividad y gestión remota rápida de servidores. | [Abrir](./laboratorios/02-comandos-ad-hoc.md) |
| 03. Despliegue de Nginx | Creación, ejecución y verificación de un playbook automatizado e idempotente. | [Abrir](./laboratorios/03-primer-playbook-nginx.md) |
| 04. Handlers | Uso de handlers para ejecutar tareas condicionales solo cuando hay cambios. | [Abrir](./laboratorios/04-handlers.md) |
| 05. Inventarios y Multi-Servidor | Despliegue multi-servidor con inventarios estáticos, plays separados y handlers para Nginx y MariaDB. | [Abrir](./laboratorios/05-inventarios-y-multiservidor.md) |
| 06. Variables, Facts y Templates | Despliegue multiplataforma con variables de grupo y host, facts descubiertos, condicionales `when`, bucles `loop` y templates Jinja2. | [Abrir](./laboratorios/06-variables-facts-templates.md) |
| 07. Roles, Vault y Laboratorio Final | Creación de roles reutilizables, uso de Ansible Vault para secretos, instalación de Node Exporter, auditoría de seguridad y backup verificable. | [Abrir](./laboratorios/07-roles-vault-y-laboratorio-final.md) |

## Material de Apoyo

| Material | Descripción |
|---|---|
| [Programa Completo Ansible](./material/Programa%20Completo%20Ansible.pdf) | PDF completo con todas las sesiones del curso |
| [Sesion01](./material/Sesion01.pdf) | Slides de la primera sesión |
