Esta web plantea introducciones a conceptos de programación en Java usados habitualmente en programación profesional en Java, pero que a veces se omiten o no se explican lo suficiente en formaciones regladas o cursos introductorios.

En cada apartado se incluyen ejercicios para poner en práctica los conceptos y enlaces a fuentes de interés con información más detallada.

## Herramientas

{% assign herramientasPosts = site.categories.herramientas | sort: 'order' %}
{% for post in herramientasPosts %}
[{{ post.title }}]({{ post.url }})
{% endfor %}

## Funcionalidades de Java

{% for post in site.categories.java %}
[{{ post.title }}]({{ post.url }})
{% endfor %}
