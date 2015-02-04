# Show latest blog post in Shopify

If you maintain multiple blogs on your Shopify site it's expected that you'll want to show visitors your most recent blog posts.  While this walkthrough won't show how to have complete control over your blog feed, it will show you how to do two specfic things:

* Show your store's most recent blog post regardless of which blog it came from.
* Show each of your blogs' most recent blog post together in a group.

### Some background

There's no easy way to sort all your store's blog articles into one array and
simply choose, for example, the most recent 5 posts.  If there was the CollectionDrop **blogs.all**, like there is for collections.all, then all the hard work would already be done.

So until that becomes a reality, here's some code snippets you can use to do some interesting things with presenting blog articles.

If you are *really keen* on sorting your blogs, though, I invite you to check out the API documents on [blog](http://docs.shopify.com/api/blog) and/or [article](http://docs.shopify.com/api/article).

## Code Time!

Show either:

A) your store's most recent blog post regardless of which blog it came from, or

B) each of your blogs' most recent blog post together in a group.

### Case A: Your store's most recent blog post

#### Getting started

* Make a new Snippet called **latest-post.liquid** in our Snippet directory.
* Call that Snippet in whatever template or other Snippet you'd like using `{% include 'latest-post' %}`

#### latest-post.liquid

In this newly created Snippet, we need to grab the most recent blog post from each of our blogs so we can compare them and find out which one has the most recent `published_at` value.

##### Step 1: Gather information on each blog's most recent post

In the first couple lines, write the following code:

```
{% assign blog_handles = "handle-1;handle-2;handle-3;etc" %}
{% assign blog_list = blog_handles | split: ';'%}
{% assign article_list = '' %}

{% for blog_handle in blog_list %}
  {% for article in blogs[blog_handle].articles limit: 1 %}
    {% capture article_list %}{{ article_list }}%%%{{ blog_handle }}@%{{ article.published_at }}{% endcapture %}
  {% endfor %}
{% endfor %}
```

Replace the list of blog handles assigned to `blog_handles` with the handles of your own blogs.  It is important that each handle is separated with a semi-colon (and no spaces).

You can find each blog's handle from your store's backend.  From the dashboard: click "Blog Posts" (on the sidebar), then "Manage Blogs" (up top), then choose a Blog.  The **blog handle** will be listed in the 'Search Enginges' section.

![http://take.ms/SF685](http://take.ms/SF685)


##### Step 2: Determine which blog has the most recent published_at value

The next block of code to paste is this:

```
{% assign article_array = article_list | split: '%%%' %}

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

What this does is break up the long string of information gathered from Step 1 into the array `article_array`.  Each element in that array is in turn broken into another array of blog articles where each array item has two pieces of information. 1) the blog handle of the blog it came from and 2) the published_at vaule.

From each array, we can grab the **published_at** value (which comes from `{{ article_item[1] }}`) and find out if it is more recent than the published_at value found before it.  If it is more recent, then it becomes the new `{{ latest_date }}`

In the end, we end up knowing which blog has the most recent article and identify it by its handle - which is set to `{{ latest_blog }}`.

##### Step 3: Display the most recent article's information

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

This block of code is mostly HTML and the write up is largely a suggestion - change it to suit your needs.  What's important to know, is with this set up you can use any of [article's liquid objects](http://docs.shopify.com/themes/liquid-documentation/objects/article).

You can also use any of [blog's liquid objects](http://docs.shopify.com/themes/liquid-documentation/objects/blog) as long as you write `blogs[latest_blog]` instead of "blog".  i.e. TO display the blog title, write: `{{ blogs[latest_blog].title }}`

#### Demonstration

Let's say I want to show the most recent post in a sidebar on all of my store's internal Pages.

![http://take.ms/2SlfT](http://take.ms/2SlfT)

In my **pages.liquid** template, I'm going to call my latest-post Snippet with `{% include 'latest-post' %}`.  The template below is using [Shopify's Timber framework](http://shopify.github.io/Timber/), but you can adapt the HTML and CSS to what ever you'd like.

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

The most recent blog post is "Winter is coming", so let's see if that's what shows up.

![http://take.ms/QeoI3](http://take.ms/QeoI3)


**Nailed it.**

### Case B: Show each blog's most recent post together in a group

A bit simpler that Case A, this time we don't need to compare each article's `published_at` value.

#### Getting started

* Make a new Snippet called **latest-post.liquid**
* Call that Snippet in whatever template or other Snippet you'd like using `{% include 'latest-post-loop' %}`

#### latest-post-loop.liquid

In this newly created Snippet, we need to grab the most recent blog post from each of our blogs and then immediately display it on the page.

##### Step 1: Gather the handles of all your blogs

Start the Snippet with the following block of code:

```
{% assign blog_handles = "handle-1;handle-2;handle-3;etc" %}
{% assign blog_list = blog_handles | split: ';' %}
```

Replace the list of blog handles assigned to `blog_handles` with the handles of your own blogs.  It is important that each handle is separated with a semi-colon (and no spaces).  For instructions on where to find a blog handle, scroll on up to Case A.

**Something to note**: This Snippet will display the most recent article for each blog, but it won't sort the blogs in order of recency.  So the articles will show up in the same order that you write the blog handles in `{% assign blog_handles = "..." %}`.

##### Step 2: Display the first article of each blog

```
{% for blog_handle in blog_list %}

  {% for article in blogs[blog_handle].articles limit: 1%}

    <article>
      <h3><a href= "{{ article.url }}">{{ article.title }}</a></h3>
      <p><span class="date" style="display: block;">Posted: <em>{{ article.published_at | date: "%B %d %Y" }}</em> in <a href="{{ blogs[blog_handle].url }}">{{ blogs[blog_handle].title }}</a></span></p>
      <section>{{ article.content | strip_html | truncatewords: 10 }}</section>
    </article>

    <hr class="divider" />

  {% endfor %}

{% endfor %}
```

This block of code has nested for loops - which can be a bit of a trick to understand.  For each **blog_handle**, I need to look at that blog's articles.  However, I'm limiting my article check to just the first one I encounter and writing that information for the browser.

Since I this group of articles to be compact looking, I need to limit the amount of text shown on the screen.  To do this, I applied two [string filters](http://docs.shopify.com/themes/liquid-documentation/filters/string-filters) to the article's content by writing: `{{ article.content | strip_html | truncatewords: 10 }}`.  I want to strip the HTML out, otherwise HTML in my blog article will be counted against my character/word limit.

#### Demonstration

I want to show my newly made latest-post-loop Snippet in a sidebar on any page using the **page.liquid** template.

##### page.liquid

```
<div class="grid">
  <div class="grid__item large--one-third">
    {% include 'latest-post-loop' %}
  </div>

  <div class="grid__item large--two-thirds">
    <h1>{{ page.title }}</h1>

    <div class="rte">
      {{ page.content }}
    </div>
  </div>
</div>

```

##### Result
![http://take.ms/tcPmJ](http://take.ms/tcPmJ)

**Beautiful.**

