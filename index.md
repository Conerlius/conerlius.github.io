<html>
<body>
<div class="row">
  <div class="col-md-12">
    <div class="panel panel-primary">
      <div class="panel-heading">最新文章</div>
      {% for post in site.posts %}
        <a  href='{{ post.url }}' class="list-group-item pjaxlink clearfix">
          {{post.title}}
          <span class="badge">{{ post.date | date:"%Y年%m月%d日" }}</span>
        </a>
      {% endfor %}
    </div>
  </div>
</div>
</body>
</html>


