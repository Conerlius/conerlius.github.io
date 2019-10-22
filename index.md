<html>
<body>
<div class="row">
  <div class="col-md-12">
    <div class="panel panel-primary">
      <div class="panel-heading">
        <center><h1>最新文章</h1></center>
      </div>
      <table>
        <tr>
          <th>标题</th>
          <th>日期</th>
        </tr>
      {% for post in site.posts limit: 20 %}
      <tr>
        <td>
          <a  href='{{ post.url }}' class="list-group-item pjaxlink clearfix">
          {{post.title}}</a>
        </td>
        <td>
          <span class="badge">{{ post.date | date:"%Y年%m月%d日" }}</span>
        </td>
        </tr>
      {% endfor %}
      </table>
    </div>
  </div>
</div>
</body>
</html>


