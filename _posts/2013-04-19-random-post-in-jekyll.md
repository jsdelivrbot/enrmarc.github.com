---
layout   : post
title    : Random post in Jekyll
summary  : How to get a random post using Jekyll
tags     : Jekyll proyectos JavaScript Html
category : projects
permalink: /blog/random-post-in-jekyll
scripts  :
  - src  : /assets/js/jquery.min.js
disqus   : true
---

<script>
  var posts = [];
  {% for post in site.posts %}
    posts.push("{{ post.url }}");
  {% endfor %}
  window.onload = function() {
    $('#random').click(function() {
      window.location = posts[Math.floor(Math.random() * posts.length)];
      return false;
    });
  }
</script>

I've been looking for an easy way to get a random post using [Jekyll], but I have found nothing.
So here is a code snippet that does the trick.
Try it: <a href="javascript:void(0)" onclick="goToRandom()" id="random">random post</a>

{% highlight html %}
{% raw %}
<!-- navbar.html -->
<script>
   var posts = [];
   {% for post in site.posts %}
      posts.push("{{ post.url }}");
   {% endfor %}

   $(function() {
      var goToRandom = function() {}
         window.location = posts[Math.floor(Math.random() * posts.length)];
      }
   });
</script>

<div class="container">
   <div class="row-fluid">
      <div class="navbar">
         <div>
            <a id="random" class="random-post pull-right">
               <i class="icon-random"></i>
            </a>
         </div>
      </div>
   </div>
</div>

{% endraw %}
{% endhighlight %}


[Jekyll]: https://github.com/mojombo/jekyll
[GitHub Pages]: http://pages.github.com/
