
# Vim - Comandos básicos

###Z Abrir un archivo

```bash
vim archivo.yaml
````

---

## Modos principales

### Modo normal

Es el modo por defecto al abrir Vim.

Sirve para navegar y ejecutar comandos.

### Modo inserción

Presionar:

```
i
```

Permite empezar a escribir o editar texto.

Para volver al modo normal:

```
Esc
```

---

## Guardar y salir

### Guardar

```
:w
```

### Salir

```
:q
```

### Guardar y salir

```
:wq
```

### Salir sin guardar

```
:q!
```

---

## Navegación

### Ir al inicio del archivo

```
gg
```

### Ir al final del archivo

```
G
```

### Moverse entre líneas

```
j
```

Bajar una línea.

```
k
```

Subir una línea.

---

## Selección

### Seleccionar líneas

```
V
```

Luego usar:

```
j
k
```

para ampliar la selección.

### Seleccionar todo el archivo

```
ggVG
```

Significa:

```
gg  -> inicio del archivo
V   -> selección por líneas
G   -> hasta el final
```

---

## Eliminar

### Eliminar la línea actual

```
dd
```

### Eliminar varias líneas

Ejemplo, eliminar 5 líneas:

```
5dd
```

### Eliminar una selección

Primero seleccionar con:

```
V
```

y luego:

```
d
```

### Eliminar todo el archivo

```
:%d
```

Alternativa:

```
ggVGd
```

---

## Copiar y pegar

### Copiar una línea

```
yy
```

### Copiar varias líneas

Ejemplo, copiar 10 líneas:

```
10yy
```

### Copiar una selección

Seleccionar con:

```
V
```

y luego:

```
y
```

### Pegar debajo

```
p
```

### Pegar arriba

```
P
```

---

## Deshacer y rehacer

### Deshacer

```
u
```

### Rehacer

```
Ctrl + r
```

---

## Buscar

### Buscar texto

```
/texto
```

Ejemplo:

```
/password
```

### Ir al siguiente resultado

```
n
```

### Ir al resultado anterior

```
N
```

---

## Reemplazar todo el contenido de un archivo

Una forma rápida:

```
:%d
```

Después entrar en modo inserción:

```
i
```

Pegar el nuevo contenido.

Luego:

```
Esc
:wq
```

---

## Flujo básico de edición

```
vim archivo.yaml
i
[editar contenido]
Esc
:wq
```