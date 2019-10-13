
真的是什么都没有？

<html>
<body>
内容:
{% for post in site.posts %}
        <a  href='{{ post.url }}' class="list-group-item pjaxlink clearfix">
          {{post.title}}
          <span class="badge">{{ post.date | date:"%Y年%m月%d日" }}</span>
        </a>
 {% endfor %}
</body>

</html>