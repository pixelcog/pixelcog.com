---
layout: post
title:  Jekyll From Scratch - Core Architecture
tags:   jekyll
image:  jekyll-pt2-core-architecture.gif
---

Now that we've covered the [basics of Jekyll as a blogging platform]({% post_url 2013-07-16-jekyll-from-scratch-introduction %}), it's time to mold a richly customized website out of the hunk of clay Jekyll has given you. In this second post of my series, I explore the depths of Jekyll's potential as a static website generator without using any plugins.

{{ more }}{% raw %}

### Table of Contents
{:.no_toc}

* Table of Contents Placeholder
{:toc}

## Planning Your Website Structure

One of the first things you'll want to do when putting together a website/blog is to plan out the url structure. This is especially true if you are [migrating](http://jekyllrb.com/docs/migrations/) from another platform and you want to preserve your links.

Having started from scratch, here is the way I decided to structure mine:

	Home:             /
	About Page:       /about/
	Portfolio Page:   /projects/
	Contact Page:     /contact/
	Blog Index:       /blog/
	Blog Pagination:  /blog/pageN/
	Blog Post:        /blog/YYYY/post-slug/
	Blog RSS Feed:    /blog/feed.xml

Once you have an outline ready, you can define these paths in your configuration and initialize your file structure.

### Configuring Permalinks

Both blog pagination and post url structures can be configured in `_config.yml` like so:

	permalink:     '/blog/:year/:title/'
	paginate_path: '/blog/page:num/'

The full list of variables you can use in the permalink structure can be found [here](http://jekyllrb.com/docs/permalinks/).

### Adding Pages

While you *can* utilize both Liquid and Markdown in the same file, I think its best to avoid mixing form and function when possible.  In my setup I have decided segregate the two by creating a new folder called `_pages` to complement the `_posts` folder in the root directory. This folder contains Markdown files for each of my main pages (about, projects, and contact). All pages utilizing Liquid blocks for looping over data (like the blog index, archives, and feeds) will exist outside of this folder.

Technically these files could be anywhere in the Jekyll folder as long as they contain the correct `permalink` front-matter, but I chose to organize them this way out of personal preference.

To accomplish this, I added the following lines to my `_config.yml`: 

	include: ['_pages']
	relative_permalinks: false

Then I created files for each of my pages in the `_pages` directory with front-matter like so:

**"\_pages/home.md"**:

	---
	permalink: /
	layout:    default
	title:     Home
	---

**"\_pages/about.md"**:

	---
	permalink: /about/
	layout:    default
	title:     About Us
	---

Once my site is fully built out, I'll be able to do the majority of my routine edits and updates by simply modifying markdown files in `_pages` and `_posts`, only accessing the others when I decide to make structural or cosmetic changes to the site.

### Simple URL Redirects

For completeness I didn't like the idea of `/blog/2013/hello-world/` going somewhere while `/blog/2013/` responds with a "404 Not Found" message. Unfortunately, there is no way to perform a proper [301 redirect](http://en.wikipedia.org/wiki/HTTP_301) on GitHub Pages, but you can still achieve the same effect with meta-refresh and javascript.

I've created the following file in `/_layouts/redirect.html` (partly borrowed from an answer on [stackoverflow](http://stackoverflow.com/a/5411601/1447303)):

	<!doctype html>
	<html>
		<head>
			<meta charset="utf-8" />
			<meta http-equiv="refresh" content="1;url={{ page.redirect }}" />
			<link rel="canonical" href="{{ page.redirect }}" />
			<script type="text/javascript">
				window.location.href = "{{ page.redirect }}"
			</script>
			<title>Page Redirection</title>
		</head>
		<body>
			If you are not redirected automatically, follow <a href='{{ page.redirect }}'>this link</a>.
		</body>
	</html>

Now `/blog/2013.html` contains the following:

	---
	permalink: /blog/2013/
	redirect:  /blog/
	layout:    redirect
	---

All you need is the front-matter – just five lines and you're done.

This may be especially useful for redirecting some legacy urls after migrating or restructuring your blog.  It isn't a true 301 redirect, but the use of canonical metadata and http-refresh will approximate it fairly well for most search engine spiders and browsers.

### Custom 404 Pages

When using GitHub Pages for hosting, you can create a document named `404.html` in your root directory and GitHub will serve it up any time a client tries to access a non-existant url.

Note that GitHub's [official documentation](https://help.github.com/articles/custom-404-pages) on this feature says _"Jekyll-generated pages will not work"_, but this is **not strictly true**. Jekyll generated pages \*do\* work, provided it gets compiled into `404.html`. I think what they intended to warn you about was that you cannot rely on front-matter variables to do things like display the user's location (for obvious reasons). You can however utilize templates and includes just like any other Jekyll page.

### Pitfalls With Pretty URLs

You'll note that for all of these "pretty" urls, a trailing `/` is needed.  This is to force Jekyll to render them at `/url/index.html` rather than simply `/url`. Jekyll *will* render documents with arbitrary names if you ask it to, but since GitHub Pages will not know how to infer the data type without a file extension; the file will be served with the content-type `"application/octet-stream"` rather than `"text/html"`. For all intents and purposes, **trailing slashes are mandatory** on pretty urls.

Also noteworthy, since urls like this can be accessed at either `/url/` or `/url/index.html` you will effectively have two urls for the same document on your blog. This can cause unexpected problems when integrating with third party comment systems and social widgets, and it's also bad for SEO. To fix this, I'd highly recommend putting **canonical url metadata** into your default template using the following line in your `<head>`:

	<link rel="canonical" href="{{ page.url }}" />

## Building Site Navigation

Now that we've built-out some pages, we'll want to build a site navigation menu where we can point to them.  While it would be pretty straight forward to manually create a styled list with our menu hardcoded, I like being able to highlight the user's current location within the menu as well.

You could do something like this:

	<nav><ul>
		<li><a {% if page.url == '/about/' %}class="active" {% endif %}href="/about/">About</a></li>
		<li><a {% if page.url == '/projects/' %}class="active" {% endif %}href="/projects/">Projects</a></li>
		<li><a {% if page.url == '/blog/' %}class="active" {% endif %}href="/blog/">Blog</a></li>
		<li><a {% if page.url == '/contact/' %}class="active" {% endif %}href="/contact/">Contact</a></li>
	</ul></nav>

But what if a user goes is at `/blog/2013/foo/`? Dealing with an ever expanding list of if/elsif statements to cover each post isn't a sustainable solution. Instead I did this:

	{% capture location %}{{ page.url | remove_first:'/' | split:'/' | first }}{% endcapture %}
	<nav><ul>
		<li><a {% if location == 'about' %}class="active" {% endif %}href="/about/">About</a></li>
		<li><a {% if location == 'projects' %}class="active" {% endif %}href="/projects/">Projects</a></li>
		<li><a {% if location == 'blog' %}class="active" {% endif %}href="/blog/">Blog</a></li>
		<li><a {% if location == 'contact' %}class="active" {% endif %}href="/contact/">Contact</a></li>
	</ul></nav>

With this first line, the first subdirectory of the user's current uri is captured in the `location` variable. This way, any uri starting with `/blog/*` will cause the corresponding menu item to be `.active`.

### Procedurally Generated Menus

The above method was sufficient for my use case, but you could take this even a step further. Say you wanted to generate this menu procedurally. Jekyll provides a `site.pages` variable which we can use to access all non-blogpost pages and their front-matter variables. For example:

	<nav><ul>
	{% for node in site.pages %}
		{% if node.title != nil and node.url != '/' %}
		{% capture nodediff %}{{ page.url | remove:node.url }}{% endcapture %}
		<li><a {% if nodediff != page.url %}class="active" {% endif %}href="{{ node.url }}">{{ node.title }}</a></li>
		{% endif %}
	{% endfor %}
	</ul></nav>

Note that any front-matter variable you set is accessible here. In the above example, I included pages with titles who's url is not '/'. You could just as easily set a front-matter variable `menuitem: true` in a few pages and use `{% if node.menuitem %}`.

One minor caveat to consider here is the **sort order** of the items within `site.pages`. Getting the menu items to appear in the order you want them can be tricky. I've found two semi-decent methods [here](http://stackoverflow.com/a/16625558/1447303) and [here](http://stackoverflow.com/a/9126294/1447303). I'll leave it as an exercise to the reader to figure out multi-level dropdown menus or any more complicated implementations.

### Sitemaps and Robots.txt

**[Sitemaps](http://en.wikipedia.org/wiki/Sitemaps)** are simple xml files used to let search spiders from Google, Bing, and others know where your content is without having to crawl your site and sniff out links. For most uncomplicated Jekyll blogs they probably aren't necessary, but all the cool kids seem to be including them so I guess I will too.

You can check out my sitemap.xml implementation [here](https://github.com/pixelcog/pixelcog.github.io/blob/master/sitemap.xml), based partly on an example from [davidensinger.com](http://davidensinger.com/2013/03/generating-a-sitemap-in-jekyll-without-a-plugin/).

The `{% comment %}` tags are there only to prevent the generated file from having a ton of newlines in it. Also note that I use `date: '%Y-%m-%d'` in place of `date_to_xmlschema` for my posts since I don't actually plan to track the time of day in my post front-matter. If you do, you may want to change it back.

**[Robots.txt](http://en.wikipedia.org/wiki/Robots_exclusion_standard)** as you probably are aware is a simple file used to blacklist urls for search engine spiders and other bots. In this sense it serves the opposite function of a sitemap. It also can be used to tell spiders where to find your sitemap. Here is my implementation:

	---
	---
	User-agent: * 
	{% for node in site.pages %}{% if node.noindex %}{% assign isset = true %}Disallow: {{ node.url }}
	{% endif %}{% endfor %}{% if isset != true %}Disallow:
	{% endif %}
	Sitemap: {{ site.url }}/sitemap.xml

Note that the `site.url` prefix is necessary here. For some reason although `Disallow` statements can be relative links, the `Sitemap` declaration must be an absolute url complete with `http://…`.

Both of these implementations require the `url` value to be defined within `_config.yml` making it accessible with `site.url`. This should be your site's full URL starting with *http://*. Please ensure it does *not* contain a trailing `/` or this will break a lot of links.

You may have noticed I've also introduced two new optional front-matter variables: `updated` and `noindex`. These can be used like so:

	---
	permalink: /contact/thankyou/
	layout:    default
	title:     Thank You
	noindex:   true
	---

Doing this will add `/contact/thankyou/` to my "Disallow" list in `robots.txt` and remove it from `sitemap.xml` entirely.

	---
	layout:  post
	title:   Hello World
	updated: 2013-06-22
	---

Doing this will change the `lastmod` timestamp of the post in `sitemap.xml` (and later we'll use this in our rss/atom feed as well).

## Building Blog Navigation

Next, we turn to our blog implementation — the core function of Jekyll. The default blog index generated by `jekyll new` works just fine, but it doesn't really demonstrate the power available to you, namely through pagination, excerpts, tags, categories, and other custom variables.

### Showing Excerpts

In the vanilla blog index, Jekyll simply loops over `site.posts`, spitting out the date, title, and url of each. This is fine for some blogs, but I like the idea of displaying a small post summary under each article, as is done in the vast majority of blogs out there.

Prior to _Jekyll 1.0_, post excerpts had to be specified in a post's front-matter or generated by a plugin if you wanted them to show up in the blog index. Now, we can simply print an auto-generated excerpt using `post.excerpt`. This variable by default will include everything in the post content up until it encounters two newlines (essentially the first paragraph of the post). You can modify this behavior to capture everything up to a custom token by adding a line to your `_config.yml` file like the following:

	excerpt_separator: "<!-- more -->"

Here's what a blog index might look like with excerpts included:

	{% for post in site.posts %}
		<h2 class="post-title"><a href="{{ post.url }}">{{ post.title }}</a></h2>
		<p class="post-meta">{{ post.date | date_to_string }}</p>
		<p class="post-excerpt">{{ post.excerpt | strip_html }}&hellip; (<a href="{{ post.url }}">Read More</a>)</p>
	{% endfor %}

Note the **`strip_html`** filter on `post.excerpt`. This is necessary to "flatten" your post excerpt in case some extra formatting happened to be at the start of your post markup. Say you started a blog post with a `<blockquote>` or a `<h1>` tag. You probably don't want that markup appearing in your excerpt paragraph.

### Pagination

After adding post excerpts to our blog index, it no longer seems practical to display a complete list of blog posts on a single page. This is where *pagination* comes in handy.  We already customized our pagination permalinks a [few sections back](#configuring-permalinks). Here are the settings needed in `_config.yml`:

	paginate:      10
	paginate_path: '/blog/page:num/'

To take advantage of this now we turn again to our blog index file (mine is located at `/blog/index.html` rather than `/index.html`).  Pagination works similarly to the default index, but rather than using `site.posts` we loop over `paginator.posts`:

	{% for post in paginator.posts %}
		<h2 class="post-title"><a href="{{ post.url }}">{{ post.title }}</a></h2>
		<p class="post-meta">{{ post.date | date_to_string }}</p>
		<p class="post-excerpt">{{ post.excerpt | strip_html }}&hellip; (<a href="{{ post.url }}">Read More</a>)</p>
	{% endfor %}

The magic of this loop is that it will stop when it reaches a number of posts designated by `paginate`, and when Jekyll is done rendering `/blog/index.html` it will re-render the same template over again starting where it left off in the post loop, this time placing it at `/blog/page:num/index.html` (replacing `:num` with the current iteration). Pretty simple, eh?

One oddity I ran into while implementing this is that you **cannot specify a permalink** in your blog index front-matter. If you do, the index file and each subsequent page will attempt to use the same permalink and overwrite each other. I think this behavior is silly, but whatever.

Once you've generated the pagination, you'll probably want to link to these additional pages. Here is what I ended up using, adapted from the [pagination section](http://jekyllrb.com/docs/pagination/) in Jekyll's docs:

	{% if paginator.total_pages > 1 %}
	<div class="pagination">
		<ul>
	{% if paginator.previous_page %}
			<li class="prev"><a href="/blog/{% if paginator.previous_page != 1 %}page{{ paginator.previous_page }}{% endif %}">Previous</a></li>
	{% endif %}
			<li><a {% if paginator.page == 1 %}class="active" {% endif %}href="/blog/">1</a></li>
	{% for count in (2..paginator.total_pages) %}
			<li><a {% if paginator.page == count %}class="active" {% endif %}href="/blog/page{{ count }}">{{ count }}</a></li>
	{% endfor %}
	{% if paginator.next_page %}
			<li class="next"><a href="/blog/page{{ paginator.next_page }}">Next</a></li>
	{% endif %}
		</ul>
	</div>
	{% endif %}

If you end up with a lot of pages, you may want to forgo the numbered list and simply display "next" and "previous" links.

### RSS and Atom Feeds

Having had no real experience generating RSS or Atom documents before, I decided to do a little bit of research on the subject. Most of the debate over whether to use RSS versus Atom is [pretty old](http://blog.webreakstuff.com/2005/07/rss-vs-atom-you-know-for-dummies/), and it appears to be a moot point today since nearly every feed reader out there supports both protocols. I went with Atom given that it is more formally standardized and anecdotally easier to work with. The only substantive reason I've found for preferring RSS 2.0 over Atom is when running a podcast feed.

Here's my implementation:

	---
	---
	<?xml version="1.0" encoding="UTF-8"?>
	<feed xmlns="http://www.w3.org/2005/Atom">
		<title>{{ site.name | xml_escape }}</title>
		<link href="{{ site.url }}{{ page.url }}" rel="self" />
		<link href="{{ site.url }}/" />
		<id>{{ site.url }}/</id>
		<author>
			<name>{{ site.author | xml_escape }}</name>
		</author>
		<updated>{{ site.time | date_to_xmlschema }}</updated>
	{% for post in site.posts limit:20 %}
		<entry>
			<title type="text">{{ post.title | strip_html }}</title>
			<link href="{{ site.url }}{{ post.url }}" />
			<id>{{ site.url }}{{ post.url }}</id>
			<published>{{ post.date | date_to_xmlschema }}</published>
			<updated>{% if post.updated == null %}{{ post.date | date_to_xmlschema }}{% else %}{{ post.updated | append: '@12am' | date_to_xmlschema }}{% endif %}</updated>
			<summary type="html">{{ post.excerpt | strip_html | xml_escape }}</summary>
			<content type="html">{{ post.content | xml_escape }}</content>
		</entry>
	{% endfor %}
	</feed>

Note this takes advantage of an optional `post.updated` front-matter variable which I also was using in [sitemap.xml](#sitemaps-and-robotstxt). I had to use `append: '@12am'` in the `post.updated` instance to ensure that it would spit out a full RFC-3339 date rather than a short `YYYY-MM-DD` one.

Also, in while doing some research I learned that GitHub Pages strangely serves up xml files as `text/xml` with `us-ascii` rather than `utf-8`. To get around this, I changed my feed's filename to `/blog/feed.atom`. Doing this forces GitHub to use the proper content-type and character-encoding. Credit goes to [Taylor Fausak's blog](http://taylor.fausak.me/2012/04/26/serving-atom-feeds-with-github-pages/) for this tip.

Once you've added your atom feed, don't forget to include a line in your blog template's `<head>` to allow autodiscovery:

	<link rel="alternate" type="application/atom+xml" title="{{ site.name }} — Feed" href="{{ site.url }}/blog/feed.atom" />

{% endraw %}

### Tags and Categories

Tags and categories are two special front-matter attributes available to blog posts, and they function practically the same. Any post can be associated with any number of tags or any number of categories through a simple YAML declaration:

	---
	layout: post
	title:  "My Awesome Blog Post"
	category: blog
	tags: blog blogging awesomeness
	---

Where these become useful is in their corresponding global `site` variables. You can loop over all of the categories used in your blog with `site.categories`, and loop over all of your tags with `site.tags`. In addition, you can loop over all of the posts in a given category with `site.categories[CATEGORY]` or all of the posts with a given tag using `site.tags[TAG]`. These can be used to construct category-specific indexes, or archive pages featuring every post with a given tag.

Unfortunately, there is no way _([yet](https://github.com/mojombo/jekyll/pull/870))_ to automatically create an index page for each tag or category. You'll need to create them individually every time you add a new tag or category. Although you can make this process fairly easy with templates. It's still something I'd like to see added as a feature.

Lastly, there isn't currently a way to implement pagination on tag or category indexes without resorting to manual page generation or plugins. I am not currently using tags on this blog as this is only my third posting and a tag archive wouldn't be especially useful yet, but I will explore this a bit more as my blog matures.

One [interesting solution](http://alexpearce.me/2012/04/simple-jekyll-searching/) I've seen is to list tags through a "search" landing page by sniffing out a query parameter, utilizing a Jekyll-generated `search.json` file and some javascript. More on search in the [next blog post]({% post_url 2013-07-19-jekyll-from-scratch-extending-jekyll %}#interactive-search).

### More Tips and Tricks

**Multi-Author Blogging**

Using the power and flexability of YAML front-matter, a few bloggers have come up with some creative ways to implement author attribution and multi-author support within blog posts. The best examples I've seen are from [Lost Decade Games](http://www.lostdecadegames.com/blog-author-attribution-using-jekyll/) and [Richa Avasthi](https://gist.github.com/ravasthi/1834570).

I'm just a one-man show at the moment, but if I ever take on contributers I'll definitely use one of these models.

**Creative Use Cases for Categories**

I have seen categories used as a way to allow for [multi-blog websites](http://www.garron.me/en/blog/multi-blog-site-jekyll.html). Others I have seen co-opt a specific category for non-blogging purposes like [portfolio items](https://github.com/alexcican/alexcican.github.com), [team member profiles](http://developmentseed.org/blog/2011/09/09/jekyll-github-pages/), [press releases](http://stackoverflow.com/a/9117390/1447303), or photos.

**Tag Clouds**

I'm not a big fan of tag clouds myself, but it is definitely possible to [create a tag cloud in Jekyll](http://vvv.tobiassjosten.net/jekyll/jekyll-tag-cloud/). It is interesting to see what is possible in Liquid with a little bit of creativity.

## Final Word

In this post I've identified virtually everything one might need to structure their static website with Jekyll. If you have any additional tips or examples to share, please comment or drop me a note and I'll try to include them here.

In my [next post]({% post_url 2013-07-19-jekyll-from-scratch-extending-jekyll %}), I'll identify ways in which you can extend Jekyll further with interactive elements, integration with third party tools, and a few other neat tricks.

