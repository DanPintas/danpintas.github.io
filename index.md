Esta web plantea introducciones a conceptos de programación en Java usados habitualmente en programación profesional en Java, pero que a veces se omiten o no se explican lo suficiente en formaciones regladas o cursos introductorios.

En cada apartado se incluyen ejercicios para poner en práctica los conceptos y enlaces a fuentes de interés con información más detallada.

## Herramientas

{% assign herramientasPosts = site.categories.herramientas | sort: 'order' %}
{% for post in herramientasPosts %}
[{{ post.title }}]({{ post.url }})
{% endfor %}

## APIs de Java

{% assign apisJavaPosts = site.categories.apisJava | sort: 'order' %}
{% for post in apisJavaPosts %}
[{{ post.title }}]({{ post.url }})
{% endfor %}

## Programación funcional

{% assign functionalPosts = site.categories.functional | sort: 'order' %}
{% for post in functionalPosts %}
[{{ post.title }}]({{ post.url }})
{% endfor %}
