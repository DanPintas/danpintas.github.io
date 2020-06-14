# Artículos

## Herramientas

{% assign postsHerramientas = site.categories.herramientas | sort: 'order' %}
{% for post in postsHerramientas %}
[{{ post.title }}]({{ post.url }})
{% else %}
{% endfor %}


## APIs de Java

{% assign postsApisJava = site.categories.apis-java | sort: 'order' %}
{% for post in postsApisJava %}
[{{ post.title }}]({{ post.url }})
{% else %}
{% endfor %}


## Programación funcional

{% assign postsFuncional = site.categories.funcional | sort: 'order' %}
{% for post in postsFuncional %}
[{{ post.title }}]({{ post.url }})
{% else %}
{% endfor %}


## Inyección de dependencias

{% assign postsInyeccionDependencias = site.categories.inyeccion-dependencias | sort: 'order' %}
{% assign postsInyeccionDependenciasSpring = site.categories.inyeccion-dependencias-spring | sort: 'order' %}
{% assign numPostsInyeccionDependencias = postsInyeccionDependencias | size %}


{% for post in postsInyeccionDependenciasSpring %}
[{{ post.title }}]({{ post.url }})
{% else %}
{% endfor %}
