# Guía paso a paso (Primera parte) — Proyecto **Cinépolis** (Django, Python, VS Code)

Voy a darte **todo** lo necesario: comandos, estructura de carpetas, archivos con código listo para copiar/pegar (models, views, urls, admin, templates) y las instrucciones para crear, activar y ejecutar el proyecto en el puerto **8036**. Trabajo con la petición de **no usar forms.py** y de enfocarnos por ahora en **CATEGORÍA** (las otras dos entidades quedan definidas pero su CRUD queda pendiente).

---

# 1 — Crear la carpeta del proyecto

Abre tu terminal (PowerShell / CMD / Terminal) y ejecuta:

```bash
# desde la carpeta donde quieras crear el proyecto
mkdir UIII_Cinepolis_0777
cd UIII_Cinepolis_0777
```

---

# 2 — Abrir VS Code sobre la carpeta

Si tienes `code` en PATH:

```bash
code .
```

(esto abre VS Code en `UIII_Cinepolis_0777`).

También puedes abrir VS Code y usar **File → Open Folder...** y seleccionar `UIII_Cinepolis_0777`.

---

# 3 — Abrir terminal en VS Code

En VS Code: menú **Terminal → New Terminal** (o `Ctrl+` ` ` ` ``) — te abre la terminal en la raíz del proyecto.

---

# 4 — Crear carpeta/entorno virtual `.venv` desde la terminal de VS Code

(Comando multiplataforma — usa la versión `python` correcta)

```bash
# Windows / macOS / Linux (asegúrate que 'python' apunta a Python 3.x)
python -m venv .venv
```

Esto crea la carpeta `.venv` dentro de `UIII_Cinepolis_0777`.

---

# 5 — Activar el entorno virtual

* **Windows PowerShell**

  ```powershell
  .\.venv\Scripts\Activate.ps1
  ```

  Si te bloquea por política (ExecutionPolicy), puedes ejecutar en PowerShell como administrador:

  ```powershell
  Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
  ```

  y luego activar.

* **Windows CMD**

  ```cmd
  .\.venv\Scripts\activate
  ```

* **macOS / Linux**

  ```bash
  source .venv/bin/activate
  ```

Cuando esté activo verás `(.venv)` al inicio de la línea de tu terminal.

---

# 6 — Activar intérprete de Python en VS Code

1. En VS Code: `Ctrl+Shift+P` → escribe **Python: Select Interpreter**.
2. Selecciona la ruta al intérprete dentro de tu venv, p. ej. `UIII_Cinepolis_0777\.venv\Scripts\python` (Windows) o `.venv/bin/python` (mac/linux).
3. Opcional: reinicia la ventana si pide recargar.

---

# 7 — Instalar Django

Con venv activado:

```bash
pip install --upgrade pip
pip install django
```

Verifica:

```bash
python -m django --version
```

---

# 8 — Crear proyecto **backend_Cinepolis** SIN duplicar carpeta

Para evitar que Django cree una carpeta extra `backend_Cinepolis/backend_Cinepolis`, utiliza el punto `.` al crear el proyecto **dentro** de la carpeta actual.

Desde `UIII_Cinepolis_0777`:

```bash
django-admin startproject backend_Cinepolis .
```

> El `.` indica "crear el proyecto aquí" (no crear subcarpeta adicional). Ahora verás `manage.py` y el paquete `backend_Cinepolis/`.

---

# 9 — Ejecutar servidor en el puerto **8036**

Antes de ejecutar, crear app y migraciones. Pero para probar:

```bash
python manage.py runserver 8036
```

Abre en el navegador: `http://127.0.0.1:8036/` o `http://localhost:8036/`.

(Si VS Code te pide permisos de firewall acepta para localhost).

---

# 10 — Copiar y pegar el link en el navegador

Abre tu navegador y pega:
`http://127.0.0.1:8036/` (o `http://localhost:8036/`)

---

# 11 — Crear aplicación `app_Cinepolis`

Con venv activado y en la raíz del proyecto:

```bash
python manage.py startapp app_Cinepolis
```

Esto crea la carpeta `app_Cinepolis` con sus archivos.

---

# 12 — `models.py`

Copia/pega el siguiente contenido en `app_Cinepolis/models.py`. Incluye los tres modelos (Categoria, Sala, Pelicula) tal como los diste — trabajaremos CRUD sólo para `Categoria` por ahora.

```python
from django.db import models

# ==========================================
# MODELO: CATEGORÍA
# ==========================================
class Categoria(models.Model):
    nombre = models.CharField(max_length=50, unique=True)
    descripcion = models.TextField(blank=True, null=True)
    edad_recomendada = models.CharField(max_length=10)
    fecha_creacion = models.DateField(auto_now_add=True)
    activo = models.BooleanField(default=True)
    popularidad = models.PositiveIntegerField(default=0)
    icono = models.CharField(max_length=100, blank=True, null=True)

    def __str__(self):
        return self.nombre


# ==========================================
# MODELO: SALA
# ==========================================
class Sala(models.Model):
    nombre = models.CharField(max_length=50)
    numero_sala = models.PositiveIntegerField(unique=True)
    capacidad = models.PositiveIntegerField()
    tipo_pantalla = models.CharField(max_length=30, choices=[
        ('2D', '2D'),
        ('3D', '3D'),
        ('IMAX', 'IMAX')
    ])
    sonido = models.CharField(max_length=30, default='Dolby Atmos')
    disponible = models.BooleanField(default=True)
    ubicacion = models.CharField(max_length=100)

    def __str__(self):
        return f"Sala {self.numero_sala} - {self.tipo_pantalla}"


# ==========================================
# MODELO: PELÍCULA
# ==========================================
class Pelicula(models.Model):
    titulo = models.CharField(max_length=150)
    descripcion = models.TextField(blank=True, null=True)
    duracion = models.PositiveIntegerField(help_text="Duración en minutos")
    clasificacion = models.CharField(max_length=10)
    idioma = models.CharField(max_length=50)
    fecha_estreno = models.DateField()
    sala = models.ForeignKey(Sala, on_delete=models.CASCADE, related_name="peliculas")
    categorias = models.ManyToManyField(Categoria, related_name="peliculas")

    def __str__(self):
        return self.titulo
```

---

# 12.5 — Procedimiento para realizar migraciones (`makemigrations` y `migrate`)

Primero registra la app en settings (ver paso 25). Después:

```bash
# crear migraciones para app_Cinepolis
python manage.py makemigrations app_Cinepolis

# aplicar migraciones a la base de datos
python manage.py migrate
```

---

# 13 — Primero trabajamos con el MODELO: **CATEGORÍA**

Las vistas, urls y templates que te doy a continuación enfocan CRUD en `Categoria` únicamente. `Sala` y `Pelicula` quedan en models.py para el futuro.

---

# 14 — `views.py` de `app_Cinepolis`

Reemplaza el contenido de `app_Cinepolis/views.py` por este (funcional, sin `forms.py`, sin validaciones):

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Categoria

# Página de inicio del sistema
def inicio_cinepolis(request):
    total_categorias = Categoria.objects.count()
    categorias = Categoria.objects.all().order_by('-fecha_creacion')[:5]
    return render(request, 'inicio.html', {'total_categorias': total_categorias, 'categorias': categorias})

# Mostrar formulario para agregar categoría
def agregar_categoria(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre')
        descripcion = request.POST.get('descripcion')
        edad_recomendada = request.POST.get('edad_recomendada')
        activo = True if request.POST.get('activo') == 'on' else False
        popularidad = request.POST.get('popularidad') or 0
        icono = request.POST.get('icono')

        Categoria.objects.create(
            nombre=nombre,
            descripcion=descripcion,
            edad_recomendada=edad_recomendada,
            activo=activo,
            popularidad=popularidad,
            icono=icono
        )
        return redirect('ver_categorias')

    return render(request, 'categoria/agregar_categoria.html')

# Ver todas las categorias
def ver_categorias(request):
    categorias = Categoria.objects.all().order_by('nombre')
    return render(request, 'categoria/ver_categorias.html', {'categorias': categorias})

# Mostrar formulario con datos para actualizar
def actualizar_categoria(request, categoria_id):
    categoria = get_object_or_404(Categoria, id=categoria_id)
    return render(request, 'categoria/actualizar_categoria.html', {'categoria': categoria})

# Procesa la actualización (POST)
def realizar_actualizacion_categoria(request, categoria_id):
    categoria = get_object_or_404(Categoria, id=categoria_id)
    if request.method == 'POST':
        categoria.nombre = request.POST.get('nombre')
        categoria.descripcion = request.POST.get('descripcion')
        categoria.edad_recomendada = request.POST.get('edad_recomendada')
        categoria.activo = True if request.POST.get('activo') == 'on' else False
        categoria.popularidad = request.POST.get('popularidad') or 0
        categoria.icono = request.POST.get('icono')
        categoria.save()
        return redirect('ver_categorias')
    return redirect('ver_categorias')

# Confirmación y borrado
def borrar_categoria(request, categoria_id):
    categoria = get_object_or_404(Categoria, id=categoria_id)
    if request.method == 'POST':
        categoria.delete()
        return redirect('ver_categorias')
    return render(request, 'categoria/borrar_categoria.html', {'categoria': categoria})
```

---

# 15 — Crear carpeta `templates` dentro de `app_Cinepolis`

Ruta: `app_Cinepolis/templates/`

Dentro de ella habrá:

* `base.html`
* `header.html`
* `navbar.html`
* `footer.html`
* `inicio.html`
* subcarpeta `categoria/` con los html del CRUD.

---

# 16 — Archivos HTML principales

A continuación te doy plantillas simples y con **Bootstrap CDN**.

Crea `app_Cinepolis/templates/base.html`:

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}Cinépolis - Administración{% endblock %}</title>

    <!-- Bootstrap CSS (CDN) -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

    <!-- Bootstrap Icons -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.css" rel="stylesheet">

    <style>
      body { padding-top: 70px; background-color: #f8fafc; }
      .card-soft { border-radius: 12px; box-shadow: 0 6px 16px rgba(18,24,40,0.06); }
      footer.fixed-footer { position: fixed; left:0; bottom:0; width:100%; background:#ffffffcc; padding:10px 0; border-top:1px solid #eee; }
    </style>

    {% block extra_head %}{% endblock %}
  </head>
  <body>
    {% include "navbar.html" %}
    <div class="container my-4">
      {% block content %}{% endblock %}
    </div>

    {% include "footer.html" %}

    <!-- Bootstrap JS Bundle -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
    {% block extra_js %}{% endblock %}
  </body>
</html>
```

Crea `app_Cinepolis/templates/navbar.html`:

```html
<nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm fixed-top">
  <div class="container">
    <a class="navbar-brand d-flex align-items-center" href="{% url 'inicio' %}">
      <i class="bi bi-film" style="font-size:1.4rem; margin-right:8px;"></i>
      <strong>Sistema de Administración Cinepolis</strong>
    </a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMain">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navMain">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'inicio' %}">Inicio</a></li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Categoría</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'agregar_categoria' %}">Agregar Categoria</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_categorias' %}">Ver Categorias</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Salas</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar salas</a></li>
            <li><a class="dropdown-item" href="#">Ver salas</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Películas</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar peliculas</a></li>
            <li><a class="dropdown-item" href="#">Ver peliculas</a></li>
          </ul>
        </li>

      </ul>
    </div>
  </div>
</nav>
```

Crea `app_Cinepolis/templates/footer.html`:

```html
<footer class="fixed-footer text-center">
  <div class="container">
    <small>
      &copy; {{ now|date:"Y" }} Cinepolis — Todos los derechos reservados.
      &nbsp;|&nbsp; Creado por Ing. Eliseo Nava, Cbtis 128
    </small>
  </div>
</footer>
```

Crea `app_Cinepolis/templates/inicio.html`:

```html
{% extends "base.html" %}
{% block title %}Inicio - Cinepolis{% endblock %}
{% block content %}
  <div class="row">
    <div class="col-md-8">
      <div class="card card-soft p-3">
        <h4>Bienvenido al Sistema de Administración Cinepolis</h4>
        <p>Manage las categorías de películas, salas y funciones. Información del sistema:</p>
        <ul>
          <li>Total de categorías: <strong>{{ total_categorias }}</strong></li>
        </ul>
      </div>
      <div class="mt-3">
        <img src="https://images.unsplash.com/photo-1517604931442-53d4f1a66f6d?q=80&w=1200&auto=format&fit=crop&ixlib=rb-4.0.3&s=placeholder" alt="Cine" class="img-fluid rounded">
      </div>
    </div>
    <div class="col-md-4">
      <div class="card card-soft p-3">
        <h6>Últimas categorías</h6>
        <ul class="list-group">
          {% for c in categorias %}
            <li class="list-group-item">{{ c.nombre }} <small class="text-muted d-block">Creada: {{ c.fecha_creacion }}</small></li>
          {% empty %}
            <li class="list-group-item">No hay categorías aún.</li>
          {% endfor %}
        </ul>
      </div>
    </div>
  </div>
{% endblock %}
```

> La imagen en `inicio.html` es una URL pública de ejemplo (Unsplash). Puedes reemplazarla con otra URL de tu elección.

---

# 21 — Crear subcarpeta `categoria` dentro de templates

Ruta: `app_Cinepolis/templates/categoria/`

---

# 22 — Templates CRUD para `categoria`

Crea `app_Cinepolis/templates/categoria/agregar_categoria.html`:

```html
{% extends "base.html" %}
{% block title %}Agregar Categoria{% endblock %}
{% block content %}
<div class="card p-4 card-soft">
  <h4>Agregar Categoria</h4>
  <form method="post">
    {% csrf_token %}
    <div class="mb-3">
      <label class="form-label">Nombre</label>
      <input class="form-control" name="nombre" required>
    </div>
    <div class="mb-3">
      <label class="form-label">Descripción</label>
      <textarea class="form-control" name="descripcion"></textarea>
    </div>
    <div class="mb-3">
      <label class="form-label">Edad Recomendada</label>
      <input class="form-control" name="edad_recomendada">
    </div>
    <div class="mb-3 form-check">
      <input type="checkbox" class="form-check-input" name="activo" checked>
      <label class="form-check-label">Activo</label>
    </div>
    <div class="mb-3">
      <label class="form-label">Popularidad (número)</label>
      <input class="form-control" name="popularidad" type="number" min="0">
    </div>
    <div class="mb-3">
      <label class="form-label">Icono (texto)</label>
      <input class="form-control" name="icono">
    </div>
    <button class="btn btn-primary">Guardar</button>
    <a href="{% url 'ver_categorias' %}" class="btn btn-secondary">Cancelar</a>
  </form>
</div>
{% endblock %}
```

Crea `app_Cinepolis/templates/categoria/ver_categorias.html`:

```html
{% extends "base.html" %}
{% block title %}Ver Categorias{% endblock %}
{% block content %}
<div class="card card-soft p-3">
  <div class="d-flex justify-content-between align-items-center mb-3">
    <h4>Categorías</h4>
    <a class="btn btn-success" href="{% url 'agregar_categoria' %}"><i class="bi bi-plus"></i> Nueva Categoría</a>
  </div>

  <table class="table table-hover align-middle">
    <thead>
      <tr>
        <th>Nombre</th>
        <th>Edad</th>
        <th>Activo</th>
        <th>Popularidad</th>
        <th>Creada</th>
        <th>Acciones</th>
      </tr>
    </thead>
    <tbody>
      {% for c in categorias %}
      <tr>
        <td>{{ c.nombre }}</td>
        <td>{{ c.edad_recomendada }}</td>
        <td>{% if c.activo %}Sí{% else %}No{% endif %}</td>
        <td>{{ c.popularidad }}</td>
        <td>{{ c.fecha_creacion }}</td>
        <td>
          <a class="btn btn-sm btn-info" href="{% url 'actualizar_categoria' c.id %}">Editar</a>
          <a class="btn btn-sm btn-danger" href="{% url 'borrar_categoria' c.id %}">Borrar</a>
        </td>
      </tr>
      {% empty %}
      <tr><td colspan="6">No existen categorías.</td></tr>
      {% endfor %}
    </tbody>
  </table>
</div>
{% endblock %}
```

Crea `app_Cinepolis/templates/categoria/actualizar_categoria.html`:

```html
{% extends "base.html" %}
{% block title %}Actualizar Categoria{% endblock %}
{% block content %}
<div class="card p-4 card-soft">
  <h4>Actualizar Categoria</h4>
  <form method="post" action="{% url 'realizar_actualizacion_categoria' categoria.id %}">
    {% csrf_token %}
    <div class="mb-3">
      <label class="form-label">Nombre</label>
      <input class="form-control" name="nombre" value="{{ categoria.nombre }}" required>
    </div>
    <div class="mb-3">
      <label class="form-label">Descripción</label>
      <textarea class="form-control" name="descripcion">{{ categoria.descripcion }}</textarea>
    </div>
    <div class="mb-3">
      <label class="form-label">Edad Recomendada</label>
      <input class="form-control" name="edad_recomendada" value="{{ categoria.edad_recomendada }}">
    </div>
    <div class="mb-3 form-check">
      <input type="checkbox" class="form-check-input" name="activo" {% if categoria.activo %}checked{% endif %}>
      <label class="form-check-label">Activo</label>
    </div>
    <div class="mb-3">
      <label class="form-label">Popularidad</label>
      <input class="form-control" name="popularidad" type="number" min="0" value="{{ categoria.popularidad }}">
    </div>
    <div class="mb-3">
      <label class="form-label">Icono</label>
      <input class="form-control" name="icono" value="{{ categoria.icono }}">
    </div>
    <button class="btn btn-primary">Actualizar</button>
    <a href="{% url 'ver_categorias' %}" class="btn btn-secondary">Cancelar</a>
  </form>
</div>
{% endblock %}
```

Crea `app_Cinepolis/templates/categoria/borrar_categoria.html`:

```html
{% extends "base.html" %}
{% block title %}Borrar Categoria{% endblock %}
{% block content %}
<div class="card p-4 card-soft">
  <h4>Confirmar borrado</h4>
  <p>¿Deseas eliminar la categoría <strong>{{ categoria.nombre }}</strong>?</p>
  <form method="post">
    {% csrf_token %}
    <button class="btn btn-danger">Sí, eliminar</button>
    <a class="btn btn-secondary" href="{% url 'ver_categorias' %}">No, cancelar</a>
  </form>
</div>
{% endblock %}
```

---

# 23 — No utilizar `forms.py`

Como pediste, el CRUD usa `request.POST` directo.

---

# 24 — `urls.py` en `app_Cinepolis`

Crea `app_Cinepolis/urls.py` con este contenido:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_cinepolis, name='inicio'),
    path('categoria/agregar/', views.agregar_categoria, name='agregar_categoria'),
    path('categoria/ver/', views.ver_categorias, name='ver_categorias'),
    path('categoria/actualizar/<int:categoria_id>/', views.actualizar_categoria, name='actualizar_categoria'),
    path('categoria/actualizar/realizar/<int:categoria_id>/', views.realizar_actualizacion_categoria, name='realizar_actualizacion_categoria'),
    path('categoria/borrar/<int:categoria_id>/', views.borrar_categoria, name='borrar_categoria'),
]
```

---

# 25 — Agregar `app_Cinepolis` en `settings.py` de `backend_Cinepolis`

Edita `backend_Cinepolis/settings.py`, en `INSTALLED_APPS` añade `'app_Cinepolis',` por ejemplo:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # apps locales
    'app_Cinepolis',
]
```

Opcional: para que la etiqueta `now` funcione en templates (usada en footer) activa `django.template.context_processors.request` (viene por defecto) y tendrás acceso a `now` con `{% now "Y" %}` — en el footer uso `{{ now|date:"Y" }}` que requiere context processor `django.template.context_processors.request` y que envíes `now` en contexto; para simplicidad, reemplaza `{{ now|date:"Y" }}` por `{% now "Y" %}`. Si no, actualiza footer:

```html
&copy; {% now "Y" %} Cinepolis — ...
```

Asegúrate que en `TEMPLATES` `APP_DIRS: True` está activado (por defecto).

---

# 26 — Configurar `urls.py` de `backend_Cinepolis` para enlazar con `app_Cinepolis`

Edita `backend_Cinepolis/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Cinepolis.urls')),  # rutas principales de la app
]
```

---

# 27 — Registrar modelos en `admin.py` y volver a migrar/crear superuser

En `app_Cinepolis/admin.py`:

```python
from django.contrib import admin
from .models import Categoria, Sala, Pelicula

@admin.register(Categoria)
class CategoriaAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'edad_recomendada', 'activo', 'popularidad', 'fecha_creacion')
    search_fields = ('nombre',)
    list_filter = ('activo',)

@admin.register(Sala)
class SalaAdmin(admin.ModelAdmin):
    list_display = ('numero_sala', 'nombre', 'capacidad', 'tipo_pantalla', 'disponible')

@admin.register(Pelicula)
class PeliculaAdmin(admin.ModelAdmin):
    list_display = ('titulo', 'duracion', 'clasificacion', 'idioma', 'fecha_estreno')
    filter_horizontal = ('categorias',)
```

Luego:

```bash
# crear migraciones (si aún no lo hiciste después de añadir app y models)
python manage.py makemigrations
python manage.py migrate

# crear superusuario para admin
python manage.py createsuperuser
```

---

# 27 (segunda aparición) — Por ahora solo trabajar con “categoría”

Las vistas, urls y templates anteriores se centran sólo en administrar `Categoria`. `Sala` y `Pelicula` están en models.py pero **no** tienen vistas ni templates (pendiente).

---

# 28 — Estética: colores suaves y modernos

Las plantillas usan Bootstrap, sombras suaves (`card-soft`) y tipografía predeterminada de Bootstrap para mantener diseño limpio y moderno. Puedes personalizar `style` en `base.html`.

---

# 29 — Estructura completa inicial de carpetas y archivos

Ejemplo de árbol principal (después de aplicar lo anterior):

```
UIII_Cinepolis_0777/
├─ .venv/
├─ manage.py
├─ backend_Cinepolis/
│  ├─ __init__.py
│  ├─ settings.py
│  ├─ urls.py
│  └─ wsgi.py
├─ app_Cinepolis/
│  ├─ migrations/
│  ├─ templates/
│  │  ├─ base.html
│  │  ├─ navbar.html
│  │  ├─ footer.html
│  │  ├─ inicio.html
│  │  └─ categoria/
│  │     ├─ agregar_categoria.html
│  │     ├─ ver_categorias.html
│  │     ├─ actualizar_categoria.html
│  │     └─ borrar_categoria.html
│  ├─ __init__.py
│  ├─ admin.py
│  ├─ apps.py
│  ├─ models.py
│  ├─ tests.py
│  ├─ urls.py
│  └─ views.py
```

---

# 30 — Proyecto totalmente funcional (pasos finales)

1. Asegúrate que `app_Cinepolis` está en `INSTALLED_APPS`.
2. Ejecuta migraciones:

   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```
3. Crea superusuario (opcional para admin):

   ```bash
   python manage.py createsuperuser
   ```
4. Ejecuta servidor en puerto **8036**:

   ```bash
   python manage.py runserver 8036
   ```
5. Abre en navegador: `http://127.0.0.1:8036/` — deberías ver la página de inicio.
6. Accede a `Categorias` → Ver/Agregar/Editar/Borrar desde la barra de navegación.

---

# 31 — Finalmente ejecutar servidor en el puerto 8036

(Ya incluido arriba; repito el comando final)

```bash
python manage.py runserver 8036
```

---

## Notas rápidas, recomendaciones y consideraciones

* No validé entradas (según tu instrucción). Si en el futuro quieres validación, podemos añadir `forms.py` o validaciones en views.
* Si quieres que las plantillas usen una base de CSS personalizada (colores suaves específicos), dime paleta (ej.: #f6f8fb, #9fb8c8, etc.) y la integro.
* Para servir archivos estáticos (si quieres CSS local o imágenes), configura `STATIC_URL` y `STATICFILES_DIRS` en `settings.py` y crea `static/` en la raíz del proyecto.
* Si quieres, preparo el diagrama ERD o el archivo `.drawio` más adelante.

---

Si quieres, ahora puedo:

* (A) Generarte los archivos listos para pegar (yo ya incluí los contenidos clave arriba).
* (B) Crear las vistas/CRUD para `Sala` y `Pelicula` también.
* (C) Añadir validaciones, mensajes (Django messages) y estilo adicional CSS.

Indica cuál de esas opciones quieres que haga **ahora** y lo genero aquí mismo.
