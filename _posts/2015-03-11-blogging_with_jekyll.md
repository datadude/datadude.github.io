---
layout: post
title:  "Blogging with Jekyll"
date:   2015-03-11 13:00
categories: Rave Hacksc
---

[Jekyll](http://jekyllrb.com/) is a simple "Blog Aware" static site generator.  You can quicky and easily write
blog posts using the [MarkDown](https://help.github.com/articles/markdown-basics/) language and [Liquid](http://liquidmarkup.org/)
templates.  See how!

### Advantages:

 * Site is completely static, so it gets served up FAST!
 * It still gives you powerfull CMS features such as re-usable code bits.
 * It still lets you use custom Javascript and CSS (this blog uses foundation for instance).
 * No Database needed!
 * Write your posts using [MarkDown](https://help.github.com/articles/markdown-basics/)
 * [Github Pages](https://pages.github.com/) will host it for free.
 * Manage your Blog with Git.

### Disadvantages
* No database functionality.
* Large numbers of posts could get unwieldy.


## How does it work?

### [Quick Start](http://jekyllrb.com/docs/quickstart/)

{% highlight console %}
~ $ gem install jekyll
~ $ jekyll new myblog
~ $ cd myblog
~/myblog $ jekyll serve
# => Now browse to http://localhost:4000
{% endhighlight %}

That will generate a new site, After that it is as simple as adding a new file with a clever name to the "_sites"
directory adding a little "Front Data" to it.

### Deeper Dive

Front Data: Page scoped data, always found at the top of the page.
Here is the front data for this page:

{% highlight text %}
 ---
   layout: post
   title:  "Blogging with Jekyll"
   date:   2015-03-11 13:00
   categories: Rave Hacksc
---
{% endhighlight %}

These values are all pretty self explanitory but lets see how they are used.  As you can see this page uses the "post"
 layout, dropping the header and footer boiler plate here it is:

 {% highlight html linenos%}
 {% raw %}
 <div class="row">
     <div class="twelve columns centered">
         {% include nav.html %}
         <!-- Main page content -->
         <div id="wrapper" class="eleven columns centered">
           <div class="post">
             <h1>{{page.title}}</h1>
             {{ content }}
           </div>   <!-- end post -->
         </div><!-- end wrapper -->
         {% include post_footer.html %}
         <!-- End Whole-Body Div -->
     </div>
 </div>
 {% endraw %}
 {% endhighlight %}

Jekyll uses the [Liquid](http://liquidmarkup.org/) template language.
{%raw%}
 * {{page.title}} prefix page "front matter" with "page."
 * {{ site.title }} prefix the global config vars with "site."
 * {% include nav.html %} include tags include content in the "_includes" directory.
 * {{ content }} is the actual page content.
{% endraw %}

If you look at the index.html page you can see that you can easily iterate through all of the posts:


 {% highlight html linenos %}
  {%raw%}
   <p>This site is where I get to document both for myself and anyone interested a few hints and tricks or even whole
       systems that I have worked with or would like to work with.  Hopefully you will find some of these posts useful.</p>
       {% for post in site.posts %}
         {% include blog_post_row.html %}
       {% endfor %}
     <!-- end row -->
   </div>
   {% endraw %}
 {% endhighlight %}

Code highlighting is easy too:
 {% highlight html linenos%}
 {% raw %}
 {% highlight html linenos%}
  <div class="row">
      <div class="twelve columns centered">
          Some fancy HTML
      </div>
  </div>
  {% endhighlight %}
  {% endraw %}
  {% endhighlight %}

## Host with Github pages

[Hosting on Github Pages](http://jekyllrb.com/docs/github-pages/) is free and easy.
All you need to do is create a specially named repo (&lt;github userid&gt;.github.io) in your github account and push it to
github.

### Learn by example!

Feel free to clone this repo:

```
git clone https://github.com/datadude/datadude.github.io.git
```

##References:
* [Github Pages](https://pages.github.com/)
* [Jekyll hosting at Github Pages](http://jekyllrb.com/docs/github-pages/)
* [Jekyll docs](http://jekyllrb.com/docs/home/)
* [Liquid Templates](http://liquidmarkup.org/)
* [Pygment Lexers (code  highlighting)](http://pygments.org/docs/lexers/)

