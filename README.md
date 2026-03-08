
# LinguaPlay

LinguaPlay es un proyecto modular que utiliza **Git Submodules** para gestionar diferentes componentes del sistema como repositorios independientes.

## Estructura del Proyecto

```
LinguaPlay
│
├── apps
│   ├── LP-API            # Backend API
│   └── LP-MOB            # Aplicación móvil
│
├── linguaplay-core       # Librería compartida entre aplicaciones
│
├── .gitmodules
├── README.md
└── LICENSE
```

Cada uno de estos directorios es **un repositorio independiente gestionado como submódulo**.

---

# Clonar el Proyecto

Para clonar el proyecto correctamente junto con todos sus submódulos:



Ejemplo:

```bash
git clone --recurse-submodules https://github.com/smyleface18/LinguaPlay.git
```

Este comando descargará:

* el repositorio principal
* todos los submódulos configurados

---

# Si el repositorio ya fue clonado sin submódulos

Inicializar y descargar los submódulos manualmente:

```bash
git submodule update --init --recursive
```

---

# Actualizar el Proyecto

Cuando haya cambios en el repositorio principal o en los submódulos:

```bash
git pull
git submodule update --init --recursive
```

Esto sincroniza todos los submódulos con el commit registrado.

---

# Actualizar Submódulos a la Última Versión

Para traer el último commit disponible de cada submódulo:

```bash
git submodule update --remote --recursive
```

---

# Trabajar en un Submódulo

Cada submódulo es un repositorio independiente.

Ejemplo trabajando en la API:

```bash
cd apps/LP-API
```

Realizar cambios normalmente:

```bash
git add .
git commit -m "feat: nueva funcionalidad"
git push
```

---

# Actualizar la Referencia del Submódulo en el Proyecto Principal

Después de hacer `push` en el submódulo, debes actualizar la referencia en el repositorio principal.

Volver al repositorio principal:

```bash
cd ../..
```

Verificar cambios:

```bash
git status
```

Agregar el submódulo actualizado:

```bash
git add apps/LP-API
```

Crear commit:

```bash
git commit -m "update LP-API submodule"
```

Subir cambios:

```bash
git push
```

Esto actualiza el commit del submódulo que usa el proyecto.

---

# Ver el Estado de los Submódulos

Para ver qué commit tiene cada submódulo:

```bash
git submodule status
```

Ejemplo:

```
b7adbdf linguaplay-core
a23c991 apps/LP-API
98ab123 apps/LP-MOB
```

---

# Comandos Útiles

Inicializar submódulos:

```bash
git submodule init
```

Descargar submódulos:

```bash
git submodule update
```

Actualizar todos los submódulos:

```bash
git submodule update --remote
```

Ver estado de submódulos:

```bash
git submodule status
```

---

# Flujo Recomendado de Trabajo

### 1. Entrar al submódulo

```bash
cd apps/LP-API
```

### 2. Realizar cambios

```bash
git add .
git commit -m "feature: nueva funcionalidad"
git push
```

### 3. Volver al repositorio principal

```bash
cd ../..
```

### 4. Actualizar referencia del submódulo

```bash
git add apps/LP-API
git commit -m "update LP-API submodule"
git push
```

---

# Notas Importantes

* Los submódulos **no se actualizan automáticamente**.
* Siempre debes **hacer commit del submódulo en el repositorio principal** después de actualizarlo.
* Cada submódulo tiene **su propio historial de Git y control de versiones**.
