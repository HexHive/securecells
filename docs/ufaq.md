---
title: FAQ
---

# (Un)Frequently Asked Questions

We've compiled a list of the most frequently-asked questions:

{% capture template %}

<ul class="collapsible questions">
{% for item in site.data.faq %}
  <li>
    <div class="collapsible-header">
      <i class="material-icons drop-down"></i>
{{ item.question }}
    </div>
    <div class="collapsible-body">
{{ item.answer }}
    </div>
  </li>
{% endfor %}
</ul>

{% endcapture %}
{{ template | replace: '    ', ''}}
