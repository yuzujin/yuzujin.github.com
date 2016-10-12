---
layout: page
title: Home
tagline: Supporting tagline
---
{% include JB/setup %}

To see a world in a grain of sand,    

And a heaven in a wild flower,   
Hold infinity in the palm of your hand,     
And eternity in an hour.  
                     -- William Blake 

    
### Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


