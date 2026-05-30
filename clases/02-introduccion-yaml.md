# Clase 02: Introducción al Lenguaje YAML

Ansible utiliza **YAML** (*YAML Ain't Markup Language*) para escribir sus automatizaciones. En esta clase, estudiaremos las reglas fundamentales de este formato de serialización de datos y cómo estructurar la información sin ruido visual.

---

## 1. XML, JSON y YAML: Comparación de Formatos

Para configurar y describir sistemas, existen diferentes formatos de datos. Analicemos cómo se vería el mismo servidor representado en XML, JSON y YAML:

### XML (eXtensible Markup Language)
Basado en etiquetas formales y muy explícito, pero verboso y propenso al desorden en configuraciones complejas.
```xml
<servidor>
  <nombre>web01</nombre>
  <puerto>80</puerto>
  <activo>true</activo>
</servidor>
```

### JSON (JavaScript Object Notation)
Ligero, común en APIs web. Sin embargo, el uso constante de llaves, corchetes, comillas y comas puede dificultar su lectura manual.
```json
{
  "nombre": "web01",
  "puerto": 80,
  "activo": true
}
```

### YAML (YAML Ain't Markup Language)
Diseñado específicamente para ser legible por humanos. Elimina etiquetas y delimitadores innecesarios, confiando en la indentación.
```yaml
nombre: web01
puerto: 80
activo: true
```

---

## 2. Reglas Básicas de YAML

Para escribir YAML válido que Ansible pueda interpretar, debes seguir estas reglas estrictas:

* **Comentarios:** Empiezan siempre con el carácter `#`. Todo lo que esté a la derecha de `#` es ignorado.
* **Indentación:** La jerarquía y estructura se definen mediante espacios en blanco.
  * **REGLA DE ORO:** Se deben usar **espacios** (generalmente 2 por nivel).
  * **PROHIBIDO:** No utilices la tecla `TAB` (tabulaciones), ya que romperá el formateador de YAML.
* **Pares Clave-Valor:** Se definen con dos puntos.
  * **REGLA DE ORO:** Siempre debe haber un espacio después de los dos puntos: `clave: valor` (no `clave:valor`).
* **Listas:** Los elementos de una lista se declaran usando un guion `-` seguido de un espacio.

---

## 3. Estructuras de Datos: Escalares, Mapas y Secuencias

En YAML, cualquier dato se clasifica en uno de estos tres tipos:

1. **Escalares:** Valores individuales simples como textos (strings), números, booleanos (`true`/`false`) o nulos.
2. **Mapas (Diccionarios):** Colecciones de pares `clave: valor`.
3. **Secuencias (Listas):** Colecciones ordenadas de elementos.

### Ejemplo Completo
En este ejemplo combinamos mapas, listas y escalares:

```yaml
usuario:
  nombre: "ansible"              # Escalar (Texto)
  sudo: true                     # Escalar (Booleano)
  edad: 25                       # Escalar (Número)
  grupos:                        # Secuencia (Lista)
    - admins
    - devops
  contacto:                      # Mapa declarado en línea (Flow Style)
    ciudad: "Lima"
    pais: "Peru"
```

---

## 4. Conectando YAML con Ansible (Estructura de un Playbook)

Un Playbook de Ansible no es más que un archivo YAML que sigue una estructura predefinida. Veamos un ejemplo mínimo de cómo se mapean estos conceptos:

```yaml
---
- name: Preparar servidor web             # Inicio de un play (lista de plays)
  hosts: web                              # Grupo de servidores
  become: true                            # Ejecutar con privilegios de administrador (sudo)
  tasks:                                  # Lista de tareas a ejecutar
    - name: Instalar nginx                # Nombre de la tarea
      apt:                                # Módulo de Ansible (administrador de paquetes)
        name: nginx                       # Parámetro del módulo (clave-valor)
        state: present                    # Declaración del estado deseado
```

### Elementos Clave
* `---` al inicio: Indica el inicio de un documento YAML (opcional, pero considerado buena práctica).
* El guion `-` en `- name`: Indica que estamos ante un elemento de una lista de "Plays" (cada play asocia un grupo de servidores con tareas).
* `hosts`: Especifica a qué grupo de servidores del inventario afectará.
* `tasks`: Contiene una lista ordenada de tareas (secuencia) que se aplicarán secuencialmente.

---

[Anterior: Clase 01 - ¿Qué es Ansible?](./01-introduccion-ansible.md) | [Siguiente: Clase 03 - Preparación del Entorno](./03-preparacion-laboratorio.md)
