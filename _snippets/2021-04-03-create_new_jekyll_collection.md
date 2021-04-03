---
layout: snippet
title: "Create new Jekyll collection"
author: "Daniel Biro"
categories: snippet
---

Creating a new Jekyll collection for "multiple-feeds" (not added to RSS feed):
1. Add a new collection to the `_config.yml` file: 
``` python
collections:
        collection_name:
            output: true  # new pages for each posts
```
2. Create a new layout `_layouts/collection_name.html`
3. If you have a menu, add this item:
``` python
- {name: 'Collection_Name', path: 'menu', url: 'collection_name.html'}
```
4. Create a `collection_name.md` markdown file with the collection_name layout and loop through the collection and display the posts however you want eg.:
<!-- I had to put everything liquid related to "raw" tags -->
{% highlight html %}
---
layout: collection_name
title: Collection List
---
<ul class="posts">
{% raw %} {% for post in site.collection_name %} {% endraw %}
    <li itemscope>
    <a href="{% raw %}{{site.github.url}}{{post.url}}{% endraw %}">{% raw %}{{post.title}}{% endraw %}</a>
    </li>
{% raw %} {% endfor %} {% endraw %}
</ul>

{% endhighlight %}

{:start="5"}
<!-- highlighting breaks the enumerated list so I start it again from 5 -->
5. Create a new folder `_collection_name` and add new posts to it

**Links**:
- [Multiply post types in Jekyll](https://www.csrhymes.com/development/2017/10/27/multiple-post-types-in-jekyll.html)
- [Introduction to collection in Jekyll](https://www.digitalocean.com/community/tutorials/jekyll-collections)