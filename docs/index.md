# Taller de Introducción a Django

El taller está pensado para iniciarse en Django en aproximadamente **una hora**.


## Requisitos

Requiere tener [Python 3 instalado](https://www.python.org/downloads/), y un conocimiento básico tanto de Python como de tecnologías web (HTTP y HTML).

## Instalación

### Django

Primero, **instalaremos el paquete `Django`** a través de `pip`, el instalador de paquetes de Python (viene por defecto incluido en su instalación):

``` bash tab="Linux/UNIX"
pip3 install Django
```

``` bash tab="Max OS X"
pip3 install Django
```

``` PowerShell tab="Windows"
pip install Django
```

### Proyecto

En Django, un **proyecto** o _project_ es un conjunto de carpetas y archivos (un _esqueleto_) donde desarrollaremos el sitio web.

Vamos a **crear un proyecto** de Django e **iniciar el [servidor de desarrollo](https://docs.djangoproject.com/es/stable/intro/tutorial01/#the-development-server)**:

``` bash tab="Linux/UNIX"
django-admin startproject mysite
cd mysite
python3 manage.py runserver
```

``` bash tab="Mac OS X"
django-admin startproject mysite
cd mysite
python3 manage.py runserver
```

``` PowerShell tab="Windows"
django-admin.exe startproject mysite
cd mysite
python manage.py runserver
```

Con ésto habremos creado un un nuevo proyecto Django llamado `mysite`, y el servidor de desarrollo estará corriendo en el puerto por defecto (8000).

En nuestro navegador **abrimos [127.0.0.1:8000](http://127.0.0.1:8000)** y confirmamos que estamos viendo la página inicial del proyecto, que tendrá el siguiente mensaje:

> The install worked successfully! Congratulations!


Abre la carpeta con tu editor de código favorito. Verás los siguientes archivos:

- `db.sqlite3`: Base de datos por defecto, en [SQLite 3](https://www.sqlite.org).
- `manage.py`: Ejecutable para administrar el proyecto. Con él hemos iniciado el servidor de desarrollo.
- `mysite`: Módulo que engloba diferentes funcionalidades a nivel de proyecto.
    - `settings.py`: Contiene la configuración. Entre otras cosas, define la base de datos.
    - `urls.py`: Contiene la lista de URLs mapeadas a la lógica.
    - `wsgi.py`: Define la [Web Server Gateway Interface](https://en.wikipedia.org/wiki/Web_Server_Gateway_Interface) que usaríamos en producción. Podemos ignorarlo para este taller.


## Vistas

Las **vistas** o _views_ en Django son funciones Python que procesan las peticiones HTTP. Serían el _controlador_ en el patrón de arquitectura [Modelo-Vista-Controlador](https://es.wikipedia.org/wiki/Modelo%E2%80%93vista%E2%80%93controlador) (en Django [la nomenclatura es diferente](https://docs.djangoproject.com/es/stable/faq/general/#django-appears-to-be-a-mvc-framework-but-you-call-the-controller-the-view-and-the-view-the-template-how-come-you-don-t-use-the-standard-names)).

Para crear una vista, primero necesitamos una **aplicación** en la que definirla.

### Aplicación

Las aplicaciones son un módulo de Python que reunirá un conjunto de funcionalidades (lógica, modelos, etc). Además, podremos [reutilizarlas en otros proyectos](https://djangopackages.org/categories/apps/).

Vamos a crear una aplicación de ejemplo para gestionar notas. Usando `manage.py startapp`, crearemos una aplicación llamada `notes`:


``` bash tab="Linux/UNIX"
python3 manage.py startapp notes
```

``` bash tab="Mac OS X"
python3 manage.py startapp notes
```

``` PowerShell tab="Windows"
python manage.py startapp notes
```

El comando creará un modulo con los siguientes archivos:

- `admin.py`: Integración con el panel administrativo de Django.
- `apps.py`: Propiedades de la aplicación (por ejemplo el nombre).
- `migrations/`: Carpeta que contendrá las futuras migraciones de la base de datos.
- `models.py`: Definición de los modelos del [ORM](https://es.wikipedia.org/wiki/Mapeo_objeto-relacional) de Django.
- `tests.py`: Definición de los tests.
- `views.py`: Definición de las **vistas**.


Para acabar, vamos a _instalar_ la aplicación en el proyecto. Ésto nos permitirá integrarla más adelante con muchas de las funcionalidades de Django, como el ORM o el motor de plantillas ([_templates_](https://docs.djangoproject.com/es/stable/ref/templates/language/)). Abre `mysite/settings.py` con tu [editor de código favorito](https://tutorial.djangogirls.org/es/code_editor/) y añade `notes` (el nombre de la aplicación) a la lista de `INSTALLED_APPS`.


``` python hl_lines="8"
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'notes',
]
```

### Vista

En Django, una vista es una función que [recibe un objeto `HttpRequest` y devuelve un objeto `HttpResponse`](https://docs.djangoproject.com/es/stable/ref/request-response/). Por ejemplo:

``` python
def hello_world(request: HttpRequest):
  return HttpResponse('Hello World!')
```

Ésta vista devolvería una respuesta en [texto plano](https://developer.mozilla.org/es/docs/Web/HTTP/Basics_of_HTTP/MIME_types#textplain). Pero nosotros vamos a definir una vista que devuelva [texto HTML](https://developer.mozilla.org/es/docs/Web/HTTP/Basics_of_HTTP/MIME_types#texthtml).

Abre `notes/views.py` y define la siguiente vista:


``` python hl_lines="5 6 7 8 9 10 11 12 13"
from django.shortcuts import render

def index(request):
    template = 'index.html'
    context = {
        'note_list': [
            {'title': 'Sin noticias de Gurb', 'content': 'Libro de Eduardo Mendoza'},
            {'title': '¿Qué hace un perro con un taladro?', 'content': 'Taladrando'},
        ]
    }
    response = render(request, template, context)
    return response
```

El método [`render`](https://docs.djangoproject.com/es/stable/topics/templates/#django.template.backends.base.Template.render) pinta una plantilla (_template_) HTML usando [el motor de plantillas de Django](https://docs.djangoproject.com/es/stable/ref/templates/language/) y devuelve el resultado como un objeto `HttpResponse`. Recuerda que en Django las vistas deben devolver una instancia de esta clase.

Veamos que parámetros pasamos a `render`:
- En `template` pasamos `notes/index.html`, que es el fichero con la plantilla HTML a pintar. Lo crearemos en la sección siguiente.
- En `context` pasamos un [mapa (`dict`)](https://docs.python.org/3.5/library/stdtypes.html#typesmapping) con una serie de valores a ser usados por la plantilla. En este caso hemos definido una lista arbitraria de notas, y cada una de ellas es a su vez otro mapa con un título y un contenido.

### Plantilla HTML

Ahora, vamos a crear la **plantilla HTML**. Por defecto, [Django busca plantillas en la carpeta `templates` de cada aplicación instalada](https://docs.djangoproject.com/es/stable/topics/templates/#django.template.backends.django.DjangoTemplates). Así que vamos a crear una carpeta llamada `templates` dentro de `mysite/notes/`:


``` bash tab="Linux/UNIX"
mkdir notes/templates
```

``` bash tab="Max OS X"
mkdir notes/templates
```

``` PowerShell tab="Windows"
mkdir notes/templates
```

Y ahora en esa carpeta crea un nuevo archivo `index.html` (la ruta completa sería `notes/templates/index.html`) con el siguiente contenido:

``` html
<!DOCTYPE html>
<html>
  <head>
    <title>Notas</title>
  </head>
  <body>
    <h1>Notas</h1>
    {% for note in note_list %}
      <h2>{{ note.title }}</h2>
      <p>{{ note.content }}</p>
    {% endfor %}
  </body>
</html>
```

La plantilla imprimirá un documento HTML con los títulos y el contenido de las notas que hemos pasado en el _contexto_, gracias al bucle `for`.

En el lenguaje de plantillas de Django, podemos imprimir las variables recibidas del _contexto_ envolviéndolas entre `{{` y `}}`.

Además, podemos usar controles de flujo básicos (como `for`, `if`, etc) usando lo que en Django se denominan [`tags`](https://docs.djangoproject.com/es/stable/ref/templates/language/#tags). La sintaxis puede variar, pero siempre estará envuelta entre `{%` y `%}`.

### URL

Por último, debemos **mapear una URL a la vista** que hemos definido. Para ello, añadiremos una ruta ([`path`](https://docs.djangoproject.com/en/stable/ref/urls/#path)) a la lista `urlpatterns` de `mysite/urls.py`:


``` python hl_lines="4 8"
from django.contrib import admin
from django.urls import path

from notes import views


urlpatterns = [
    path('', views.index),
    path('admin/', admin.site.urls),
]
```

Hemos mapeado la vista a la ruta raíz `''`, de forma que si ahora refrescamos la página [127.0.0.1:8000](http://127.0.0.1:8000), veremos el resultado. Si lo hubiésemos mapeado tal que `path('notes', views.index)`, la URL sería `127.0.0.1:8000/notas`.

Fíjate que el servidor de desarrollo ha detectado las modificaciones y se ha reiniciado automáticamente, aplicando los cambios sin necesidad de apagarlo y volverlo a encender manualmente.

## Modelos


En Django, los **modelos** son la única fuente de información de tus datos. Con ellos:
- Definimos su [esquema](https://es.wikipedia.org/wiki/Esquema_de_una_base_de_datos).
- Gestionamos los cambios del esquema ([_migraciones_](https://en.wikipedia.org/wiki/Schema_migration)).
- Consultamos y manipulamos los registros.

Los modelos implementan el patrón [ORM](https://es.wikipedia.org/wiki/Mapeo_objeto-relacional). Django trabaja con [bases de datos relacionales](https://es.wikipedia.org/wiki/Base_de_datos_relacional). Recuerda que por defecto utiliza [SQLite 3](https://www.sqlite.org). Simplificando, podemos pensar que las bases de datos relacionales almacenan la información en tablas (_esquemas_), donde las columnas son las propiedades y las filas cada uno de los registros.

En nuestro proyecto en concreto, podríamos almacenar las notas en una tabla con las columnas "título" y "contenido" (`title` y `content`), donde las filas serían cada una de nuestras notas. Vamos a definir un modelo para ello. Abre el archivo `notes/models.py` y añade las siguientes líneas:

``` python hl_lines="4 5 6 7 8 9 10 11"
from django.db import models


class Note(models.Model):
    title = models.CharField(max_length=50)
    content = models.TextField()

    def __str__(self):
        return self.title
```

En Django, los modelos deben extender `models.Model`. Las clases `CharField` y `TextField` son algunos de los campos que podemos definir en nuestro modelo, pero [hay muchos más](https://docs.djangoproject.com/es/stable/ref/models/fields).

Fíjate que hemos sobreescrito el método mágico de Python [`__str__`](https://docs.python.org/3/reference/datamodel.html?#object.__str__). Django utilizará éste método en [diversos sitios](https://docs.djangoproject.com/es/stable/ref/models/instances/#str), y además facilitará la depuración de la aplicación. Si no lo sobreescribimos, el modelo por defecto devolverá el `id` del registro.

### Migraciones

Tenemos el modelo pero, antes de ponernos a crear registros, tenemos que crear la tabla donde se van a guardar. Para los cambios de esquema usaremos [_migraciones_](https://docs.djangoproject.com/es/stable/topics/migrations/). Django tiene un comando para crear migraciones, detectando automáticamente los cambios realizados en los modelos. Ejecuta el siguiente comando, que buscará estos cambios en la aplicación `notes`:

``` bash tab="Linux/UNIX"
python3 manage.py makemigrations notes
```

``` bash tab="Mac OS X"
python3 manage.py makemigrations notes
```

``` PowerShell tab="Windows"
python manage.py makemigrations notes
```

Esto habrá creado un archivo de migración en `notes/migrations/0001_initial.py`. Si lo abres, podrás ver instrucciones para crear la tabla en la base de datos con los campos que hemos definido en el modelo. Fíjate que automáticamente ha añadido una clave primaria, llamada por defecto `id`, sin necesidad de declararla en el modelo. Todos los modelos de Django tienen implícitamente un campo `id`.

Para crear la tabla en la base de datos, ejecutamos el comando `migrate`:

``` bash tab="Linux/UNIX"
python3 manage.py migrate notes
```

``` bash tab="Mac OS X"
python3 manage.py migrate notes
```

``` PowerShell tab="Windows"
python manage.py migrate notes
```

Este comando aplicará las migraciones pendientes de la aplicación `notes`. Django lleva un registro de las migraciones aplicadas en otra tabla creada por defecto.

Podemos abrir una consola en la base de datos del proyecto con el comando `dbshell`, y listar las tablas creadas con [`.tables`](https://sqlite.org/cli.html#special_commands_to_sqlite3_dot_commands_):

``` bash tab="Linux/UNIX"
$ python3 manage.py dbshell
SQLite version 3.24.0 2018-06-04 14:10:15
Enter ".help" for usage hints.
sqlite> .tables
django_migrations  notes_note
```

``` bash tab="Mac OS X"
$ python3 manage.py dbshell
SQLite version 3.24.0 2018-06-04 14:10:15
Enter ".help" for usage hints.
sqlite> .tables
django_migrations  notes_note
```

``` PowerShell tab="Windows"
$ python manage.py dbshell
SQLite version 3.24.0 2018-06-04 14:10:15
Enter ".help" for usage hints.
sqlite> .tables
django_migrations  notes_note
```

Las tablas creadas por defecto en Django están formadas por el nombre de la aplicación, un guión bajo (`_`), y el nombre del modelo (en minúscula y sin espacios). Por eso la tabla se ha creado con el nombre `notes_note` (recuerda que la aplicación se llama `notes`, y el modelo `note`).


### Crear, actualizar y borrar

Para crear algunos registros de ejemplo vamos a abrir una consola interactiva ([REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)) de Django con el comando `shell`:

``` bash tab="Linux/UNIX"
python3 manage.py shell
```

``` bash tab="Mac OS X"
python3 manage.py shell
```

``` PowerShell tab="Windows"
python manage.py shell
```

Una vez dentro de la consola, importamos el modelo:

``` python
from notes.models import Note
```

Podemos crear registros instanciando la clase del modelo (en este caso `Note`) pasando sus valores al constructor, y posteriormente llamando al método `save`:

``` python
note = Note(title='El Guateque', content='Película sobre una animada fiesta')
note.save()
```

El registro no se insertará en la base de datos hasta que llamemos a `save`, que al ser un registro nuevo ejecutará un [`INSERT`](https://www.w3schools.com/sql/sql_insert.asp).

Podemos actualizar el registro cambiando el valor de sus atributos en la instancia y llamando de nuevo a `save`, el cual ésta vez realizará un [`UPDATE`](https://www.w3schools.com/sql/sql_update.asp):

``` python
note.title = "Pulp Fiction"
note.save()
```

Por último, podemos borrar el registro llamando a `delete`, el cual ejecutará un [`DELETE`](https://www.w3schools.com/sql/sql_delete.asp).

``` python
note.delete()
```

No cierres la consola de Django, la necesitaremos para la sección siguiente.

### _Manager_

En Django, los modelos tienen por defecto un _**manager**_ para interactuar con la base de datos. Todos los modelos tienen un _manager_ en el atributo de clase [`objects`](https://docs.djangoproject.com/es/stable/topics/db/models/#model-attributes).

En los ejemplos anteriores, cuando llamamos a `save`, por debajo se está llamando al _manager_ del modelo. Vamos a crear un par de registros de ejemplo usando directamente el _manager_ desde la consola de Django:

``` python
Note.objects.create(title='El Guateque', content='Película sobre una animada fiesta')
Note.objects.create(title='Hyperion', content='Libro de Dan Simmons')
```

Pero lo más importante es que `objects` nos va a permitir hacer consultas a la base de datos para leer los registros guardados. Ejecuta la siguiente línea para ver una lista con los registros que acabamos de crear:

``` python
Note.objects.all()
```

`objects` ofrece [muchos métodos para realizar consultas](https://docs.djangoproject.com/es/stable/ref/models/querysets/), pero de momento nos quedamos con `all`, la cual devuelve todos los registros.

Cierra la `shell` de Django escribiendo `exit()`:

``` python
exit()
```

Vamos a cambiar la vista `index` para que lea los registros de la base de datos. Abre `notes/views.py` y reemplaza el valor de `note_list` del contexto de la vista:


``` python hl_lines="3 9"
from django.shortcuts import render

from notes.models import Notes


def index(request):
    template = 'index.html'
    context = {
        'note_list': Notes.objects.all()
    }
    response = render(request, template, context)
    return response
```

Si ahora refrescamos la página [127.0.0.1:8000](http://127.0.0.1:8000), veremos que muestra los registros que hemos creado ("El Guateque" e "Hyperion").

## Administración

Django ofrece por defecto un panel de [administración](https://docs.djangoproject.com/es/stable/intro/tutorial02/#introducing-the-django-admin) web que nos permitirá gestionar los registros de la base de datos fácilmente. Éste, a su vez, depende de otras aplicaciones que instala Django por defecto para gestionar usuarios y sesiones. Para empezar a usar la administración, primero debemos aplicar las migraciones de estos modelos. Ejecutemos el comando `migrate`:

``` bash tab="Linux/UNIX"
python3 manage.py migrate
```

``` bash tab="Max OS X"
python3 manage.py migrate
```

``` PowerShell tab="Windows"
python manage.py migrate
```

Veremos que ejecuta migraciones de diferentes aplicaciones. Ahora vamos a crear un usuario administrador para poder iniciar sesión en la administración. Ejecuta el comando `createsuperuser` y responde sus preguntas:

``` bash tab="Linux/UNIX"
python3 manage.py createsuperuser
```

``` bash tab="Max OS X"
python3 manage.py createsuperuser
```

``` PowerShell tab="Windows"
python manage.py createsuperuser
```

Ahora abrimos la página de administración en [127.0.0.1:8000/admin](http://127.0.0.1:8000/admin) e iniciamos sesión con el nombre de usuario y password que acabamos de crear. Verás que, entre otras cosas, puedes gestionar los usuarios desde ahí.

### Integración con el modelo

Vamos a registrar el modelo `Note` en la administración para que podamos gestionar los registros desde allí. Para ello, abrimos `notes/admin.py`, importamos el modelo `Note`, y lo regisramos con `admin.site.register`:


``` python hl_lines="3 6"
from django.contrib import admin

from .models import Note


admin.site.register(Note)
```

Si refrescamos el panel de administración en [127.0.0.1:8000/admin](http://127.0.0.1:8000/admin), veremos que ahora podemos gestionar los registros del modelo desde allí. Prueba a añadir algunas notas de ejemplo usando la administración, vuelve a la [página raíz del proyecto](http://127.0.0.1:8000), y comprueba que se muestran correctamente.


## Tests

Para acabar, vamos a escribir algunos [tests](https://docs.djangoproject.com/es/stable/intro/tutorial05/) para comprobar de forma automática el funcionamiento de la aplicación.

Django ofrece una clase `TestCase`, que extiende [`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase) con algunas [funcionalidades específicas](https://docs.djangoproject.com/en/stable/topics/testing/tools/#testcase) para escribir tests en Django. Una de las más importantes es que incluye un cliente (`self.client`) y un servidor para realizar peticiones a las vistas de la aplicación. Además, cada test estará envuelto en una transacción, de forma que podemos usar diferentes conjuntos de datos fácilmente.

### Tests para la vista

Abrimos `notes/tests.py` y añadimos los siguiente test:


``` python hl_lines="3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19"
from django.test import TestCase

from .models import Note


class IndexViewTests(TestCase):

    def test_index_shows_all_notes(self):
        Note.objects.create(title="Test", content="I am a test note")
        response = self.client.get('/')
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, '<p>I am a test note</p>')
        self.assertQuerysetEqual(response.context['note_list'], ['<Note: Test>'])

    def test_index_shows_empty(self):
        response = self.client.get('/')
        self.assertEqual(response.status_code, 200)
        self.assertNotContains(response, '<p>I am a test note</p>')
        self.assertQuerysetEqual(response.context['note_list'], [])
```

El primer test crea una de ejemplo en la base de datos y llama a la URL raíz (`/`). Después comprueba que la página HTML recibida está imprimiendo el contenido de la nota correctamente y que el contexto de la vista es el esperado (` <Note: Test>` es el resultado de evaluarse el modelo como una _string_ a través del método `__str__` que sobreescribimos en el modelo).

El segundo test comprueba que cuando la base de datos está vacía no se imprime ninguna nota. Fíjate que el registro creado en el primer test no estará disponible en el segundo (`assertNotContains`) debido a que cada test está envuelto en una transacción que que no se llega a completar.


### Ejecución

Para ejecutar los tests, usaremos el comando `test`:

``` bash tab="Linux/UNIX"
python3 manage.py test
```

``` bash tab="Max OS X"
python3 manage.py test
```

``` PowerShell tab="Windows"
python manage.py test
```

Veremos una salida similar a ésta:

```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.009s

OK
Destroying test database for alias 'default'...
```

Fíjate que ha creado una base de datos de prueba al principio de la ejecución, y la ha destruido al terminar. Ésta base de datos es creada exclusivamente para correr los tests.


## Siguientes pasos

A continuación se dan algunas sugerencias para continuar aprendiendo con éste proyecto:

- Añadir una [plantilla base](https://tutorial.djangogirls.org/es/template_extending/).
- Añadir una [clave foránea](https://docs.djangoproject.com/es/stable/ref/models/fields/#foreignkey) en el modelo `Note` [al modelo `User`](https://docs.djangoproject.com/es/stable/topics/auth/customizing/#referencing-the-user-model).
- Implementar la vista `index` como una [vista basada en clases (_class based views_)](https://developer.mozilla.org/es/docs/Learn/Server-side/Django/Generic_views#Vista_(basada_en_clases)).


### Para leer más

Algunos tutoriales más detallados:

- [Django Tutorial](https://docs.djangoproject.com/es/stable/intro).
- [Django Girls Tutorial](https://tutorial.djangogirls.org/es/django_installation/).
- [Guías de Django, por Mozilla](https://developer.mozilla.org/es/docs/Learn/Server-side/Django).
