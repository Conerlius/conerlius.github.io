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
        {% for po in site.recent_num  %}
      <tr>
        <td>
          <a  href='' class="list-group-item pjaxlink clearfix">
          aaa</a>
        </td>
        <td>
          <span class="badge">bbb</span>
        </td>
        </tr>
      {% endfor %}
      {% for post in site.posts  %}
        {% if forloop.index  <= site.recent_num %}
      <tr>
        <td>
          <a  href='{{ post.url }}' class="list-group-item pjaxlink clearfix">
          {{post.title}}</a>
        </td>
        <td>
          <span class="badge">{{ post.date | date:"%Y年%m月%d日" }}</span>
        </td>
        </tr>
        {% endif %}
      {% endfor %}
      </table>
    </div>
  </div>
</div>
</body>
</html>


