# Show latest blog article in Shopify

Show either:

* your store's latest blog post, or
* each of your store's blog's latest post.

## Before we start

There's no easy way to sort all your store's blog articles into one array and
simply choose, for example, the most recent 5 posts.  If there was the CollectionDrop **blogs.all**, like there is for collections.all, then all the hard work would already be done.

So until that becomes a reality, here's some code snippets you can use to do some interesting things with presenting blog articles.

## Getting started

* Make a new Snippet called **latest-article.liquid**
* Call that Snippet in whatever template or other Snippet you'd like using `{% include 'latest-article' %}`

## latest-article.liquid

Inside this new Snippet, write the following code:

```
{% assign blog_handles = "watercooler;fashion-report;news"%}
{% assign blog_list = blog_handles | split: ';'%}

{% for blog_handle in blog_list %}
  {% for article in blogs[blog_handle].articles limit: 1 %}
    {% capture article_list %}{{ article_list }}%%%{{ blog_handle }}@%{{ article.published_at }}{% endcapture %}
  {% endfor %}
{% endfor %}
```

Replace the list of blog handles assigned to `blog_handles` with the handles of your own blogs.  It is important that each handle is separated with a semi-colon.

To find the URL from your store's dashboard, click "Blog Posts", then "Manage Blogs", then choose a Blog.  The **blog handle** will be listed in the 'Search Enginges' section.

![http://take.ms/SF685](http://take.ms/SF685)


