{% for album in site.albums reversed %}
 <h2> 
    <a href="{{ album.url }}">
      {{ album.name }} - {{ album.date }}
    </a>
  </h2>
  
  <p>{{ album.content | markdownify }}</p>
{% endfor %}
