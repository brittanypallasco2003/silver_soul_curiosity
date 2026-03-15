# Silver Soul Curisity

Repositorio principal del sistema.
Este proyecto organiza múltiples componentes mediante **Git Submodules**, permitiendo que cada módulo tenga su propio repositorio, historial y ciclo de desarrollo independiente.

---

## Arquitectura del repositorio

El repositorio root **no contiene todo el código directamente**, sino que referencia otros repositorios a través de submódulos.

Ventajas de este enfoque:

* separación clara de responsabilidades
* repositorios independientes por módulo
* control de versiones específico por componente
* posibilidad de reutilizar módulos en otros proyectos

Estructura general:

```
root-project/
│
├── .gitmodules
├── services/
│   
│   
│   
│
├── libs/
│   
│   
│
└── README.md
```

Cada carpeta dentro de `services` o `libs` puede corresponder a un repositorio independiente.

---

# Requisitos

Antes de trabajar con el proyecto asegúrate de tener instalado:

* Git

Verificar instalación:

```bash
git --version
```

---

# Clonar el repositorio

Para clonar el repositorio **junto con todos los submódulos**:

```bash
git clone --recurse-submodules https://github.com/brittanypallasco2003/silver_soul_curiosity
```

Esto descargará:

* el repositorio principal
* todos los submódulos definidos en `.gitmodules`

---

# Inicializar submódulos manualmente

Si el repositorio fue clonado sin `--recurse-submodules`, inicializa los submódulos ejecutando:

```bash
git submodule update --init --recursive
```

Este comando:

* inicializa los submódulos
* descarga el contenido de cada repositorio
* posiciona cada módulo en el commit definido por el root

---

# Actualizar submódulos

Para sincronizar todos los submódulos con la versión registrada en el repositorio principal:

```bash
git submodule update --recursive
```

Para traer la versión más reciente de los submódulos desde sus repositorios remotos:

```bash
git submodule update --remote --merge
```

---

# Agregar un nuevo módulo al proyecto

Para añadir un nuevo componente como submódulo:

```bash
git submodule add <repo-url> <ruta>
```

Ejemplo:

```bash
git submodule add https://github.com/org/payment-service.git services/payment
```

Luego registrar el cambio:

```bash
git add .gitmodules services/payment
git commit -m "Add payment service submodule"
git push
```

---

# Actualizar un submódulo específico

1. Entrar al directorio del submódulo:

```bash
cd services/api
```

2. Actualizar el módulo:

```bash
git pull origin main
```

3. Volver al repositorio root:

```bash
cd ../..
```

4. Registrar el nuevo commit del submódulo:

```bash
git add services/api
git commit -m "Update api submodule"
git push
```

---

# Ver estado de los submódulos

```bash
git submodule status
```

Este comando muestra el commit actual de cada submódulo referenciado por el proyecto.

---

# Sincronizar URLs de submódulos

Si cambia la URL de algún submódulo:

```bash
git submodule sync
```

Luego:

```bash
git submodule update --init --recursive
```

---

# Problemas comunes

### Submódulos aparecen vacíos

Ejecutar:

```bash
git submodule update --init --recursive
```

---

### Cambios en submódulos no reflejados en el repositorio root

Después de actualizar un submódulo es necesario registrar el nuevo commit en el repositorio principal:

```bash
git add <ruta-del-submodulo>
git commit -m "Update submodule reference"
git push
```

---

# Buenas prácticas

* cada submódulo debe tener su propio ciclo de desarrollo
* evitar modificar código directamente desde el repositorio root
* mantener actualizada la referencia de los submódulos
* documentar cada módulo de forma independiente

---

