# Sesión 8: Roles, Monitoreo, Seguridad y Backups

## Objetivo de la clase

Al finalizar, el estudiante podrá:

1. Explicar por qué un rol mejora la reutilización y el mantenimiento.
2. Diferenciar `tasks`, `handlers`, `templates`, `defaults` y `vars`.
3. Crear un rol local con `ansible-galaxy role init`.
4. Ejecutar el rol dos veces y comprobar idempotencia.
5. Explicar el modelo *pull* de Prometheus y la función de Node Exporter.
6. Instalar y validar Node Exporter en un nodo de laboratorio.
7. Cifrar una variable mediante Ansible Vault.
8. Auditar SSH y firewall sin bloquear el acceso al laboratorio.
9. Crear un backup mínimo y verificable mediante Ansible.
10. Ejecutar un laboratorio final que combine rol, Vault, auditoría y backup.

## Flujo de la clase

```text
Inventario
    |
    v
Roles reutilizables
    |
    +--> Web / Base de datos
    +--> Node Exporter
    +--> Seguridad / Backup
    |
    v
Vault protege secretos
    |
    v
Validación + segunda ejecución
```

## 1. Teoría de roles

### Problema que resuelve un rol

Un playbook monolítico termina mezclando:

- instalación de paquetes;
- creación de usuarios;
- templates;
- variables;
- handlers;
- validaciones específicas de cada servicio.

Un rol agrupa una responsabilidad:

```text
roles/pit_banner/
|-- defaults/main.yml
|-- handlers/main.yml
|-- tasks/main.yml
`-- templates/banner.j2
```

### Directorios de un rol

| Ruta | Responsabilidad |
|---|---|
| `tasks/main.yml` | Punto de entrada de las tareas del rol |
| `handlers/main.yml` | Acciones notificadas cuando una tarea cambia |
| `templates/` | Archivos Jinja2 procesados antes de copiarse al host |
| `defaults/main.yml` | Valores públicos y fáciles de sobrescribir |
| `vars/main.yml` | Variables internas con mayor precedencia |
| `files/` | Archivos estáticos que no requieren procesamiento |
| `meta/main.yml` | Metadatos y dependencias del rol |

## 2. Ansible Galaxy

Galaxy es una plataforma comunitaria para compartir roles y colecciones.

### Comandos básicos

```bash
ansible-galaxy role list
ansible-galaxy role init --help
ansible-galaxy role search nginx
```

### Instalar un rol desde requirements.yml

```yaml
---
roles:
  - name: nginx
    src: geerlingguy.nginx
    version: "3.2.0"
```

```bash
ansible-galaxy role install -r requirements.yml -p roles
ansible-galaxy role list -p roles
```

## 3. Monitoreo con Prometheus y Node Exporter

### Prometheus

- almacena series temporales;
- consulta endpoints de métricas periódicamente;
- evalúa reglas y alertas;
- utiliza un modelo *pull*.

### Node Exporter

- corre en cada servidor Linux;
- expone métricas en `http://host:9100/metrics`;
- informa CPU, memoria, disco, red y sistema de archivos;
- no almacena el histórico: Prometheus lo consulta.

```text
Prometheus
   | HTTP GET /metrics
   +------------------> ubuntu1:9100
   +------------------> centos1:9100
```

## 4. Seguridad

### Capas de protección

```text
Vault       -> protege secretos en el repositorio
SSH         -> controla identidad y acceso remoto
UFW         -> reduce puertos expuestos
Fail2Ban    -> bloquea abuso repetido
Backups     -> permite recuperar información
Validación  -> demuestra que el estado es correcto
```

### Orden seguro de hardening SSH

1. Crear o confirmar el usuario administrativo.
2. Instalar la llave pública.
3. Abrir una segunda conexión y comprobarla.
4. Validar `sshd_config` con `sshd -t`.
5. Permitir SSH en el firewall.
6. Recién entonces desactivar contraseña/root.
7. Recargar SSH; no cerrar la sesión original hasta validar.

## 5. Ansible Vault

Vault permite cifrar variables y archivos para que no estén visibles en texto plano.

### Comandos esenciales

```bash
ansible-vault encrypt_string 'valor-secreto' --name 'mi_secreto'
ansible-vault view archivo.yml
ansible-vault edit archivo.yml
ansible-vault decrypt archivo.yml
```

### Ejemplo de uso

```bash
ansible-vault encrypt_string 'token-demo-pit' \
  --name 'vault_demo_token' \
  > group_vars/all/vault.yml
```

Para ejecutar un playbook con vault:

```bash
ansible-playbook site.yml --ask-vault-pass
```

## 6. Comandos resumen

```bash
# Roles
ansible-galaxy role init roles/pit_banner
ansible-galaxy role list

# Vault
ansible-vault encrypt_string 'valor' --name 'var_name'
ansible-playbook site.yml --ask-vault-pass

# Node Exporter
ansible ubuntu1 -i inventory.ini -b -m ansible.builtin.apt \
  -a "name=prometheus-node-exporter state=present update_cache=true" \
  --ask-become-pass

ansible ubuntu1 -i inventory.ini -b -m ansible.builtin.service \
  -a "name=prometheus-node-exporter state=started enabled=true" \
  --ask-become-pass

# Validación
ansible ubuntu1 -i inventory.ini -m ansible.builtin.uri \
  -a "url=http://localhost:9100/metrics status_code=200 return_content=false"
```
