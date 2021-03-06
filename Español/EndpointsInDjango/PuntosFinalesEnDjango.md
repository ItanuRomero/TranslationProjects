---
layout: post
title: "Cómo hacer puntos finales simples con Django usando Restless"
categories:
  - django
  - python
tags:
  - Django 
  - restless
  - open-source
  - python
  - mongo
  - code
featured-img: rest
slug: restless
aliases:
  - /english/2017/04/03/make-endpoints-using-restless.html
date: 2017-04-03T14:25:52-05:00
---

La mayoría de los tutoriales nos enseñan cómo volver HTML como respuesta a una requisición. Algunas veces, és útil para hacerlo un poquito más RESTful.
Una de las opciones és utilizar [Django REST Framework](http://www.django-rest-framework.org/) pero algunas veces tu necesitas algo más simples. En esa ocasión puedes usar [Restless](http://restless.readthedocs.io/). 
Restless és un miniframework creado por [Daniel Lindsley](https://github.com/toastdriven) basado en lo que el aprendió haciendo [Tastypie](https://django-tastypie.readthedocs.io/en/latest/) y otras librerías REST.
<!--more-->

Cómo dijo Lindsley en la documentación *“Encuanto otros frameworks intentan ser muy completos, incluso las features especiales o los tie deeply para ORMs, Restless és un viaje de vuelta al esencial”*. 
Aquí yo les voy a mostrar que tu puedes crear endpoints de una manera muy simples utilizando Restless.

{{<figure src="https://cdn-images-1.medium.com/max/800/1*BWYEnAFaPtrWCnpLWJ_gZA.gif#center">}}

Primero, instalemos Restless:

```
$ pip install restless
```

Ahora, vamos a verlo en acción! Usaré un simples modelo Django como ejemplo:

```python
class Author(models.Model):
    name = models.CharField(max_length=50)

class Book(models.Model):
    author = models.ForeignKey(Author)
    title = models.CharField(max_length=250)
pages = models.IntegerField()
```

Para hacer un endpoint que enumere todas las instancias de **Book** necesitas crear un archivo para añadir una clase llamada Resource. Aquí, asumiremos que este archivo se llamará **api.py**.

Resources son clases (categorias) que tienen métodos (implementados por ti) para todos los principales métodos HTTP. En nuestro primer ejemplo, nosotros escribiremos el método llamado **list**, que representa el método GET.

```python
# On the api.py...
from restless.dj import DjangoResource
from restless.preparers import FieldsPreparer

from .models import Book

class BookResource(DjangoResource):
    preparador = FieldsPreparer(fields={
        'book_author': 'author.name',
        'book_title': 'title',
        'book_pages': 'pages',
    })

    def list(self):
      return Book.objects.all()


# On the urls.py...
from django.conf.urls import include, url
from .api import BookResource

urlpatterns = [
    url(r'^books/', include(BookResource.urls())),
]
```

Bueno, que tenemos aquí? Primero definimos un **preparador**.
Ese preparador és donde vás a definir quales atributos y propriedades de su instancia iran a nuestra respuesta de la API.
Las llaves en el diccionario que definiste adentro de tu variable **preparador** puede ser qualquer cosa que quieras que sea (aquí, yo los hice empezar con un **book** para que veas que puede ser qualquier cosa). Los valores en el diccionario, sin embargo, necesitan ser un atributo de la instancia **book**. Si tienes una clave externa (que és el caso del campo **author**), puedes acceder a esos atributos y propriedades en la misma manera que lo harías en el _related manager_.

Entonces, en nuestro preparador, nosotros añadimos 3 atributos: uno que esta relacionado con el autor del **book** (llamado adentro del preparador como un **author.name**), y dos propriedades guardadas en la instancia del **book**: **title** y **pages**.

Después que definimos nuestro preparador, nosotros aplicamos un método nombrado **list**. El qual simplemente vuelve la consulta en el modelo **Book** donde conseguiremostodas las instancias en mi base de datos.

Después, tu debes incluir en tú **urls.py** el **bookResource.urls** como lo harías con qualquier otra visualización Django. Para cada método en el la clase BookResource, el Restless intenta encontrar un cierto tipo de url.
Para el método **list**, la url típica se vera como **localhost:8000/books/**.
Entonces, todas las veces que haces un GET en la url **localhost:8000/books/**, tu solicitud será encaminada al BookResource y entonces buscará por el método **list**.

Lo que hace Restless debajo del capó cuando llega al método, es obtener cada instancia que devolvió la consulta (el **.objects.all()**), pasando por el **preparador** para crear un diccionario de información “serializable” y entonces serializarlo (devolverlo como un JSON).

Así, en resumo, cuando accedes el **localhost:8000/books/**, la solicitud será encaminada para la clase BookResource, y desde que esteas haciendo un GET, sabrá que debe buscar por un método **list**. Hecho. Tu API de todos los libros esta ahí, serializada como (para 2 instancias book en tu modelo):

```
{
  "objects":
    [
      {
        "book_author": "First Author",
        "book_title": "First Title",
        "book_pages": 200
      },
      {
        "book_author": "Seoond Author",
        "book_title": "Second Title",
        "book_pages": 124
      },
    ]  
}
```

Facíl fácil, no?

Como hemos visto antes, métodos diferentes en Restless nos llevan a diferentes patrones de url. Si intentas hacer un *POST* en la url **localhost:8000/books/**, será encaminado para un método **create** (crear) en la clase BookResource. Para una solicitud *PUT*, buscará por un método **update** (actualizar). Contodo, si tu vas a **.../books/book_id/** no lo encaminará para la lista de métodos, porque estás especificando el vlaor de la instancia que quieres acceder.


```python
# On the api.py...
from restless.dj import DjangoResource
from restless.preparers import FieldsPreparer

from .models import Author, Book

class BookResource(DjangoResource):
    preparador = FieldsPreparer(fields={
        'book_author': 'author.name',
        'book_title': 'title',
        'book_pages': 'pages',
    })

    def list(self):
      return Book.objects.all()
    
    def detail(self, pk):
       return Book.objects.get(id=pk)
      
    def create(self):
        return Book.objects.create(
            author=Author.objects.get(name=self.data['author']),
            title=self.data['title'],
            pages=self.data['pages']
        )
    
    def update(self, pk):
        book = Book.objects.get(id=pk)
        book.author = Author.objects.get(name=self.data['author'])
        book.title = self.data['title']
        book.pages = self.data['pages']
        book.save
        return book
```

En ese caso, el cuerpo del *POST* debe tener una clave de identificación válida para una instancia **Author**, el título debe ser una string y las paginas necesitan ser un entero.
Para usos más detallados, es recomendado inserir una verificación para los datos que tu usuario te esta enviando :D

{{<figure src="https://cdn-images-1.medium.com/max/800/1*8PlVNsci0toMN3ZMbQ5exg.gif#center">}}

Así, acá te dejo una lista de trucos para nuestro ejemplo BookResource.

* método de clase Restless → método HTML y la url
* método **list** →  método *GET* en **localhost:8000/books/**
* método **create** →  método *POST* en **localhost:8000/books/**
* método **detail** →  método *GET* en **localhost:8000/books/book_id/**
* método **update** →  método *PUT* en **localhost:8000/books/book_id/**

Ok, eso está demás, pero ahora cualquiera puede acceder a sus datos, cierto?
Para añadir algunas restricción en sus métodos Resource, tu debes agregar un método llamado **is_autenticated** en tu Resource, y eso debe volver un booleano. Si el método vuelve True, tu usuario podrá acceder a tus datos.
Esto se verá como:

```python
# On the api.py...
from restless.dj import DjangoResource
from restless.preparers import FieldsPreparer

from .models import Author, Book

class BookResource(DjangoResource):
  def is_authenticated(self):
    return self.request.user.is_authenticated()
```

Acá están algunos ejemplos simples de como puedes utilizar Restless. A pesar de que sea simples, puede ser muy poderoso. Espero que te ayude!