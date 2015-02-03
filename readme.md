# Show latest blog post in Shopify

Show either:

* A) your store's most recent blog post, or
* B) each of your store's blog's most recent blog post.

## Some background

There's no easy way to sort all your store's blog articles into one array and
simply choose, for example, the most recent 5 posts.  If there was the CollectionDrop **blogs.all**, like there is for collections.all, then all the hard work would already be done.

So until that becomes a reality, here's some code snippets you can use to do some interesting things with presenting blog articles.

## Case A: Your store's most recent blog post

### Getting started

* Make a new Snippet called **latest-post.liquid**
* Call that Snippet in whatever template or other Snippet you'd like using `{% include 'latest-post' %}`

### latest-post.liquid

We need to grab the most recent blog post from each of our blogs so we can compare them and find out which one has the most recent post.

#### Step 1: Gather information on each blog's most recent post

Inside this new Snippet, first write the following code:

```
{% assign blog_handles = "news;fashion-report;dopeness"%}
{% assign blog_list = blog_handles | split: ';'%}
{% assign article_list = '' %}

{% for blog_handle in blog_list %}
  {% for article in blogs[blog_handle].articles limit: 1 %}
    {% capture article_list %}{{ article_list }}%%%{{ blog_handle }}@%{{ article.published_at }}{% endcapture %}
  {% endfor %}
{% endfor %}
```

Replace the list of blog handles assigned to `blog_handles` with the handles of your own blogs.  It is important that each handle is separated with a semi-colon.

To find the URL from your store's dashboard, click "Blog Posts", then "Manage Blogs", then choose a Blog.  The **blog handle** will be listed in the 'Search Enginges' section.

![http://take.ms/SF685](http://take.ms/SF685)


#### Step 2: Determine which blog has the most recent published_at value

The next block of code to paste is this:

```
{% assign article_array = article_list | split: '%%%'%}

{% assign latest_date = '' %}
{% assign latest_blog = '' %}


{% for article_set in article_array %}
  {% unless forloop.first %}

    {% assign article_item = article_set | split: '@%' %}

    {% if article_item[1] > latest_date %}
      {% assign latest_blog = article_item[0] %}
      {% assign latest_date = article_item[1] %}
    {% endif %}

  {% endunless %}
{% endfor %}
```

What this does is break up the long string of information gathered from Step 1 into useful arrays.  From each array, we can grab the **published_at** value (which comes from `{{ article_item[1] }}`) and find out if it is more recent than the value found before it.  If it is more recent, then it becomes the new `{{ latest_date }}`

In the end, we end up knowing which blog has the most recent article and identify it by its handle - which is set to `{{ latest_blog }}`.

#### Step 3: Display the most recent article's information

The next block of code to paste is this:

```
<article>
{% for article in blogs[latest_blog].articles limit: 1 %}
  <h3><a href= "{{ article.url }}">{{ article.title }}</a></h3>

  <p><span class="date" style="display: block;">Posted: <em>{{ article.published_at | date: "%B %d %Y" }}</em> in <a href="{{ blogs[latest_blog].url }}">{{ blogs[latest_blog].title }}</a></span></p>

  <section>{{ article.excerpt_or_content }}</section>
{% endfor %}
</article>
```

This block of code is mostly HTML and the write up is largely a suggestion - change it to suit your needs.  What's important to know, is with this set up you can use an of [article's liquid objects](http://docs.shopify.com/themes/liquid-documentation/objects/article).

You can also use any of [blog's liquid objects](http://docs.shopify.com/themes/liquid-documentation/objects/blog) as long as you write `blogs[latest_blog]` instead.  i.e. `{{ blogs[latest_blog].title }}`

### Demonstration

Let's say I want to show the most recent post in a sidebar on my store's Pages.

![http://take.ms/2SlfT](http://take.ms/2SlfT)

In my **pages.liquid** template, I'm going to call my new Snippet with `{% include 'latest-post' %}`.  The template below is using [Shopify's Timber framework](http://shopify.github.io/Timber/), but you can adapt it to what ever you'd like.

```
<div class="grid">
  <div class="grid__item large--one-third">
    {% include 'latest-post' %}
  </div>

  <div class="grid__item large--two-thirds">
    <h1>{{ page.title }}</h1>

    <div class="rte">
      {{ page.content }}
    </div>
  </div>
</div>
```

Below you see that I have three blogs, all with some blog posts.

![http://take.ms/ix5on](http://take.ms/ix5on)

The most recent blog post is **Winter is coming**, so let's see if that's what shows up.

![http://take.ms/QeoI3](http://take.ms/QeoI3)


**Nailed it.**





