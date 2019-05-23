---
id: django-integration
title: Django integration
sidebar_label: Django
---


Ariadne ships with `ariadne.contrib.django` package that should be used as Django app and provides utilities for adding GraphQL server to Django projects.


## Adding GraphQL API to Django project

### Add app to `INSTALLED_APPS`

Add `ariadne.contrib.django` to your project's `INSTALLED_APPS` setting (usually located in `settings.py`):

```python
INSTALLED_APPS = [
    ...
    "ariadne.contrib.django",
]
```

Ariadne app provides Django template for GraphQL Playground. Make sure that your Django project is configured to load templates form application directories. This can be done by checking if `APP_DIRS` option located in `TEMPLATES` setting is set to `True`:

```
TEMPLATES = [
    {
        ...,
        'APP_DIRS': True,
        ...
    },
]
```


### Create executable schema

Create Python module somewhere in your project that will define the executable schema. It may be `schema` module living right next to your settings and urls:

```python
# schema.py
from ariadne import QueryType

type_defs = """
    type Query {
        hello: String!
    }
"""

query = QueryType()

@query.field("hello")
def resolve_hello(*_):
    return "Hello world!"

schema = make_executable_schema(type_defs, query)
```


### Add GraphQL view to your urls 

Add GraphQL view to your project's `urls.py`:

```
from ariadne.contrib.django.views import GraphQLView
from django.urls import path

from .schema import schema

urlpatterns = [
    ...
    path('graphql/', GraphQLView.as_view(schema=schema), name='graphql'),
]
```


### Configuration options

`GraphQLView.as_view()` takes mostly the same options that [`graphql`](ariadne-reference.md#configuration-options) does, but with two differences:

- `context_value` can be callable that will be called with single argument ([`HttpRequest`](https://docs.djangoproject.com/en/2.2/ref/request-response/#httprequest-objects) instance) and its return value will be used for rest of query execution as `context_value`.
- `debug` option is not available and it's set to the value of `settings.DEBUG`

Django GraphQL view has one option specific to it: `playground_options`, a dict of [GraphQL Playground options](https://github.com/prisma/graphql-playground#settings) that should be used.


## `Date` and `Datetime` scalars

For convenience Ariadne also provides `Date` and `DateTime` scalar implementations that can be used to represent Django dates and datetimes in form understood by JS date and time handling libraries like [Moment.js](https://momentjs.com/).

> Scalars have dependency on [dateutil library](https://github.com/dateutil/dateutil).

To use them in your project, update your schema to define `Date` and `Datetime` scalar and pass their Python implementations to `make_executable_schema`:

```python
from ariadne.contrib.django.scalars import date_scalar, datetime_scalar

type_defs = """
    scalar Date
    scalar DateTime

    type Query {
        hello: String
    }
"""

schema = make_executable_schema(type_defs, [date_scalar, datetime_scalar, ...])
```