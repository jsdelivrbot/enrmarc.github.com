---
layout    : post
title     : Random post in Jekyll
summary   : How to get a random post using Jekyll
tags      : jekyll proyectos random javascript
---

I've been looking for an easy way to get a random post using [Jekyll], but I have found nothing.
So here is a code snippet that do the trick.
Click the <i class="icon-random"> </i> icon at the top-right corner to see how it works.

{% highlight html %}
{% raw %}
<!-- navbar.html -->
<script>
   var posts = [];
   {% for post in site.posts %}
      posts.push("{{ post.url }}");
   {% endfor %}  

   $(function() {
      $('#random').click(function() {
         window.location = posts[Math.floor(Math.random() * posts.length)]; 
      });
   });
</script>

<div class="container">
   <div class="row-fluid">
      <div class="navbar">
         <div>
            <a id="random" class="random-post pull-right"><i class="icon-random"> </i>
            </a>
         </div>
      </div>
   </div>
</div>

{% endraw %}
{% endhighlight %}


[Jekyll]: https://github.com/mojombo/jekyll 
[GitHub Pages]: http://pages.github.com/ 