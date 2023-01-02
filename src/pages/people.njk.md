---
layout: base
permalink: /people/
eleventyNavigation:
  key: People
  order: 2
---

## People

### Lab Administration
<table class="table">
{%- for person in LabPeople -%}

{% if person.labtitle %}

{% if "Director" in person.labtitle %}

<tr><td><strong>{{person.labtitle}}</strong></td>
	<td>{{person.name}}</td>
	<td>{{person.email}}</td>
</tr>
{% endif %}
{% endif %}
{% endfor %}

</table>


### Core Research Team
<table class="table">
{%- for person in LabPeople -%}

{% if person.labtitle %}

{% if "Core Research" in person.labtitle %}

<tr>
	<td>{{person.name}}</td>
	<td>{{person.email}}</td>
</tr>
{% endif %}
{% endif %}
{% endfor %}

</table>


### At Stanford
<table class="table">
{%- for person in LabPeople -%}

{% if person.status %}

{% if 'stanford' in person.status %}

<tr>
	<td>{{person.name}}</td>
	<td>{{person.email}}</td>
	<td>{% if person.website%}<a href="{{person.website}}">website</a>{% endif %}</td>
{% endif %}
</tr>
{% endif %}
{% endfor %}

</table>



### Elsewhere
<table class="table">
{%- for person in LabPeople -%}

{% if person.status %}

{% if 'elsewhere' in person.status %}

<tr>
	<td>{{person.name}}</td>
	<td>{{person.email}}</td>
	<td>{% if person.website %}<a href="{{person.website}}">website</a>{% endif %}</td>
<td>{% if person.university %}{{person.university}}{% endif %}</td>
</tr>
{% endif %}
{% endif %}
{% endfor %}

</table>