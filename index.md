 <html>
 {% for post in paginator.posts %}
        <a  href='{{ post.url }}' class="list-group-item pjaxlink clearfix">
          {{post.title}}
          <span class="badge">{{ post.date | date:"%Y年%m月%d日" }}</span>
        </a>
      {% endfor %}
	  
	  <html>