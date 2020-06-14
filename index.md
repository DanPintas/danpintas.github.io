# Artículos

## Herramientas

{% assign postsHerramientas = site.categories.herramientas | sort: 'order' %}
{% for post in postsHerramientas %}
* [{{ post.title }}]({{ post.url }})
{% endfor %}


## APIs de Java

{% assign postsApisJava = site.categories.apis-java | sort: 'order' %}
{% for post in postsApisJava %}
* [{{ post.title }}]({{ post.url }})
{% endfor %}


## Programación funcional

{% assign postsFuncional = site.categories.funcional | sort: 'order' %}
{% for post in postsFuncional %}
* [{{ post.title }}]({{ post.url }})
{% endfor %}


## Inyección de dependencias

{% assign postsInyeccionDependencias = site.categories.inyeccion-dependencias | sort: 'order' %}
{% for post in postsInyeccionDependencias %}
* [{{ post.title }}]({{ post.url }})
{% endfor %}
