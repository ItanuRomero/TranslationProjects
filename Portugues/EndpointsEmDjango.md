---
layout: post
title: "Como Fazer endpoints simples com Django utilizando Restless"
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

A maioría dos tutoriais de Django nos ensinam como retornar um HTML como resposta a uma requisição. De vez em quando, é útil fazer com que ele seja um pouco mais RESTful (siga todos os padrões REST).
Uma opção é usar o [Django REST Framework](http://www.django-rest-framework.org/) porém ás vezes você precisa de algo um pouco mais simples. Assim, temos o [Restless](http://restless.readthedocs.io/).
Restless é um miniframework criado por [Daniel Lindsley](https://github.com/toastdriven) baseado no que ele aprendeu fazendo o [Tastypie](https://django-tastypie.readthedocs.io/en/latest/) e algumas outras bibliotecas REST.
<!--more-->

Como Lindsley diz na documentação *“Enquanto outros frameworks se atentam a ser extremamente completos, incluindo funcionalidades especiais ou utilizando ORMs, Restless é uma viagem de volta ao básico”*. 
E aqui eu vou mostrar como você pode criar endpoints de forma simples utilizando Restless.

{{<figure src="https://cdn-images-1.medium.com/max/800/1*BWYEnAFaPtrWCnpLWJ_gZA.gif#center">}}

Primeiro, vamos instalar o Restless:

```
$ pip install restless
```

Agora, vamos vê-lo em ação! Usarei um modelo Django simples como exemplo:

```python
class Author(models.Model):
    name = models.CharField(max_length=50)

class Book(models.Model):
    author = models.ForeignKey(Author)
    title = models.CharField(max_length=250)
pages = models.IntegerField()
```

Para fazer um endpoint que liste todas as instâncias do **book** é necessário criar um arquivo para adicionar uma classe chamada Resource. Aqui, nós vamos assumir que esse arquivo se chamará **api.py**.

Resources são classes que possuem métodos (implementados por você) para todos os principais métodos HTTP. Em nosso primeiro exemplo, nós escreveremos o método chamado **list**, que representa o método GET.

```python
# On the api.py...
from restless.dj import DjangoResource
from restless.preparers import FieldsPreparer

from .models import Book

class BookResource(DjangoResource):
    preparer = FieldsPreparer(fields={
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

Então, o que temos aqui? Primeiro nós definimos um **preparer**.
O preparador é onde você definirá quais atributos e propriedades da sua instância vão ir para a resposta de nossa API,
As chaves no dicionário que você definiu dentro da sua variável **preparer** podem ser qualquer coisa que queiras que seja (aqui, eu fiz elas começarem com **book_** então você pode ver que elas podem ser qualquer coisa). Os valores do dicionário, entretanto, devem ser um atributo da instância **book**. Se você tem uma chave estrangeira (ForeignKey) (que é o caso do campo **author**), você pode acessar seus atributos e propriedades da mesma forma que tu farias no related manager.

Assim, em nosso preparador, nós adicionamos 3 atributos: um que está relacionado com o autor do livro (chamado dentro do preparador como **author.name**), e duas propriedades guardadas na instÂncia book: **title** e **pages**.

Depois que definimos nosso preparador, aplicamos um método chamado **list**. O qual simplesmente retorna a query no modelo **Book** onde nós conseguimos todas as instâncias em meu database.

Após isso, você deve incluir em seu **urls.py** o **BookResource.urls** como seria feito em qualquer outra view Django. Para cada método na classe  BookResource, o Restless tenta encontrar um certo tipo de url.
Para o método **list**, a url típica parecerá com **localhost:8000/books/**.
Assim, toda vez que você fizer um GET na url **localhost:8000/books/**, a sua requisição será roteada para o BookResource e depois procurará pelo método **list**.

