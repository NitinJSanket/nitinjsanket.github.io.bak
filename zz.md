---
layout: post
title: zz
order: 6
---

<hr/>
<div>

{% assign sorted_teaching = site.teaching | sort:"order" %}
{% for teaching in sorted_teaching %}
    <div><br/>
    <h3>{{ teaching.title }}</h3><br/>
    {% endfor %}
    </div><br/><hr/>
{% endfor %}

</div>