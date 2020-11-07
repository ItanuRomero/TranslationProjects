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