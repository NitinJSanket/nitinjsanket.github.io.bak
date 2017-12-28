---
layout: page
title: Projects
order: 4
---
<hr/>

<div>
{% assign sorted_projects = site.projects | sort:"order" %}

{% for project in sorted_projects %}

    <div>
    <br/>
    
    <h3> <a href="{{ project.link }}">{{ project.title }}</a></h3><br/>
    
    {% if project.img %}
        <img class="right" style="width: 40%; padding-left: 1em" src="{{ project.img }}">
    {% endif %}

    </div>
    <hr/>
{% endfor %}
</div>