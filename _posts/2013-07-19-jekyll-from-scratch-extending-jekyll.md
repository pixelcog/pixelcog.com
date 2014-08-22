---
layout: post
title:  Jekyll From Scratch - Extending Jekyll
tags:   jekyll
image:  jekyll-pt3-extending-jekyll.gif
updated: 2014-08-21 @ 7:42pm
---

In my previous posts I've covered [the basics of Jekyll]({% post_url 2013-07-16-jekyll-from-scratch-introduction %}), and [building a static website framework]({% post_url 2013-07-18-jekyll-from-scratch-core-architecture %}) from the ground up. In this post, I'll demonstrate ways in which you can enhance your otherwise-static Jekyll website or blog with interactive components like comment threads, site search, and contact forms, and I'll share a few additional tips and tricks I've run into as well.

{{ more }}

### Table of Contents
{:.no_toc}

* Table of Contents Placeholder
{:toc}

## Building Client-Side Interaction

Now that the framework is in place, we've got a fully functioning static website and blog. But what about the interactive component? What about the things that require server-side scripting and databases like comment threads or search? Well, many dynamic features of modern blogs can be incorporated through third party services nowadays. Here I'll go over a few of them.

### User Comments with Disqus

There are a few services out there offering embedded comment integration at the moment. I picked Disqus because it's been around a while, has good adoption, and generally seems to work well. Close contenders include [Livefyre](http://www.livefyre.com/comments/) and Facebook's [Comments Box](https://developers.facebook.com/docs/reference/plugins/comments/) plugin. Each has its pros and cons.

To implement Disqus comments, you simply [sign up for an account](https://disqus.com/admin/signup/) and add some javascript to your templates. You can follow the instructions they give you, or you can just use my consolidated script below.

Include the following lines somewhere in your main template, replacing `your_shortname` with the shortname you were given when you signed up. Ensure this is embedded on every page where you'd want to embed disqus comments or list comment counts.
{% raw %}
	<script type="text/javascript">
		var disqus_shortname  = 'your_shortname',
		    disqus_identifier = '{{ page.path | split:'/' | last | cgi_escape }}',
		    disqus_url        = '{{ site.url }}{{ page.url | uri_escape }}'
		;
		(function() {
			var load = function(src){
				var s = document.createElement('script'); s.type = 'text/javascript'; s.async = true; s.src = src;
				var e = document.getElementsByTagName('script')[0]; e.parentNode.insertBefore(s, e);
			};
			load('//' + disqus_shortname + '.disqus.com/count.js');
			if (document.getElementById('disqus_thread')) {
				load('//' + disqus_shortname + '.disqus.com/embed.js');
			}
		})();
	</script>
{% endraw %}
On any page where you'd want to display a comment thread, add the following line somewhere after `page.content`:

	<div id="disqus_thread"></div>

Also wherever you want to include a link displaying the number of comments at the target url, include the following line:
{% raw %}
	<a href="{{ post.url }}#disqus_thread" data-disqus-identifier="{{ post.path | split:'/' | last | cgi_escape }}">View Comments</a>
{% endraw %}
The "View Comments" text will be replaced with "# Comments" when the script is loaded.

_**Important:** if you aren't looping over pages using a `post` variable like I am in this example, change `post` to whatever variable you're using. If you're linking to the comments on the current page, use `page` instead of `post` on both the href and data attributes._

In these examples I've utilized the `disqus_url` variable to ensure that comment threads are linked to the canonical url, and the `disqus_identifier` variable to tie the thread to the post filename. This way if you ever restructure your blog, the comment threads will still work so long as the post filenames are the same.

### Social Widgets

Integrating social plugins into your website is pretty straight forward. Usually it's just a little bit of javascript to add to a template. If you use any of these, this is where the `rel="canonical"` declaration that I suggested [in the previous post]({% post_url 2013-07-18-jekyll-from-scratch-core-architecture %}#pitfalls-with-pretty-urls) can become important.

**Facebook** — Go to [Facebook's Developer Page](https://developers.facebook.com/docs/plugins/) to add "Like" buttons, "Share" widgets, and so on.

**Twitter** — Go to [Twitter's Widget Settings](https://twitter.com/settings/widgets) to create embedded tweet timelines, or favorites lists. Go to [Twitter's Resources Page](https://twitter.com/about/resources/buttons) to find "Tweet This", "Follow Me", or "@Mention Me" buttons.

**LinkedIn** — Go to [LinkedIn's Plugins Page](https://developer.linkedin.com/plugins) to find "Follow" and "Share" buttons.

There's also [Reddit](http://www.reddit.com/buttons/), [Delicious](https://delicious.com/tools) and a billion others. I personally think they're mostly clutter and fairly pointless unless you have a sizable audience, so I don't intend to use them at the moment.

### Contact Forms with jotForm

There are a lot of free contact form services out there. [Wufoo](http://www.wufoo.com/), [Formstack](http://www.formstack.com/), [123ContactForm](http://www.123contactform.com/) and [jotForm](http://www.jotform.com/) are some popular ones just to name a few. One guy I saw even [co-opted his GitHub Issue Tracker](http://erjjones.github.io/blog/How-I-built-my-blog-in-one-day/) as a feedback/contact form.

I ended up using jotForm because it was fairly simple and included [CAPTCHA](http://en.wikipedia.org/wiki/CAPTCHA) support. To incorporate this onto my own [contact page](/contact/) I created an account, made a simple form, and clicked the "Embed Form" button.

I opted for the "Source Code" option as opposed to "Embed" or "iFrame" because I wanted primary control over the CSS. The default stylesheet clashed with my fonts and wasn't compatable with my responsive design, so I threw it out and used my own.

### Interactive Search

I was surprised to find that there are actually a few clever options for embedded interactive searching on static websites like Jekyll.

One option is a service called [Tapir](http://tapirgo.com/). This service will take an RSS or Atom feed for your site's content, index it, and provide a keyword search api which you can interact with through a jQuery widget. I'm not exactly sure how they make money on this, but it's a great service.

To integrate it, you'd simply take the Atom feed we created earlier, remove the post limit, and add the non-blogpost pages to create a full site feed for Tapir to digest. I don't yet have enough content on my site to warrant a full keyword search, but I'll definitely come back to this if I decide I need it in the future.

Another option is a pretty ingenious idea from [Alex Pearce](http://alexpearce.me/2012/04/simple-jekyll-searching/) and [Development Seed](http://developmentseed.org/blog/2011/09/09/jekyll-github-pages/) which involves creating a `search.json` document with Jekyll, populated with your site's content including tags and other metadata which you can pull in through a custom javascript plugin.

### User Analytics

No website is complete without the ability to track page hits and user engagement. Since GitHub Pages doesn't offer access to server logs, using a third party tracker is your only option. I went with [Google Analytics](http://www.google.com/analytics/) because it's familiar, but any of [the alternatives](https://iwantmyname.com/blog/2013/03/prefer-to-own-your-data-here-are-some-alternatives-to-google-analytics.html) would do just fine as well. Enabling any of them is as simple as copying a few lines of javascript to the bottom of your site template.

## Other Jekyll Tips and Tricks

**Reading Time**

This is a [neat idea](http://andytaylor.me/2013/04/07/reading-time/) that I saw in a [couple](http://sicanstudios.com/blog/) of places. Since Jekyll provides a handy `number_of_words` Liquid filter, you can estimate the reading time of your posts based on a rate of 180 words-per-minute with `{% raw %}{{ page.content | number_of_words | divided_by:180 }}{% endraw %}`.

Since there's no way to round a number using Liquid, the filter ends up spitting out the "[floor](http://en.wikipedia.org/wiki/Floor_and_ceiling_functions)" of the number (essentially it ignores everything after the decimal point). The original author added `append: '.0'` to get a long floating point number and then used JavaScript to round it up to a whole number, but that's a really hacky solution and it's actually unnecessary. You could just use `plus:91 | divided_by:180` to trick it into rounding upward. I ended up using the following:
{% raw %}
	{% capture readtime %}{{ post.content | number_of_words | plus:91 | divided_by:180.0 | append:'.' | split:'.' | first }}{% endcapture %}
	{% if readtime == '0' %} &lt; 1{% else %}{{ readtime }}{% endif %} min. read
{% endraw %}

> **Update — Aug 21st, 2014**:  A recent update to Liquid broke my original read time trick.  It used to truncate everything after the decimal place, but now it will display uneven divisions as a fraction or a long floating point number.  You can get around this by adding `.0` to the divisor and then adding `| append:'.' | split:'.' | first`.  I have updated the example code above.  It's ugly but it still works.

**Similar Posts**

Jekyll provides a built-in mechanism to identify and display ["related" posts](http://jekyllrb.com/docs/variables/#site_variables) with the `site.related_posts` variable. This may be useful to display a handful of links for further reading below your current post. Alternatively, you could loop through similarly-tagged posts and list those using `site.tags[tagname]`.

**Site Last Updated Notice**

You can use `{% raw %}{{ site.time | date_to_string }}{% endraw %}` in your page footer to display a site-wide "last updated" time. This will be set to the most recent time Jekyll has compiled the blog (presumably on your last git push).

**Table of Contents /w <strike>Maruku</strike> Kramdown**

> **Update — May 8th, 2014**:  Earlier in 2014 Jekyll and GitHub Pages both [announced](https://help.github.com/articles/migrating-your-pages-site-from-maruku) that they are deprecating maruku as Jekyll's default markdown compiler. Luckily, "kramdown" now has pretty solid feature parity including aspects like [table of contents support](http://kramdown.gettalong.org/converter/html.html#toc).

<strike><p>
If you're using "Maruku" as your Markdown interpreter (which at the moment is Jekyll's default), you can take advantage of a few extra features which it defines as a <a href="http://maruku.rubyforge.org/maruku.html">"superset" of the Markdown syntax</a>. One feature which I've found handy for this post in particular is the ability to generate a linked <a href="http://maruku.rubyforge.org/maruku.html#toc-generation">table of contents</a> using all of the headers in your document.
</p></strike>

Adding a table of contents to your post is pretty simple; just include the following in your post somewhere:

	* Table of Contents Placeholder
	{:toc}

<strike><p>
Strangely for this to work, you'll need an `h1` tag <a href="http://webiva.lighthouseapp.com/projects/38599/tickets/5-maruku-table-of-contents-not-generating-without-extra-h1-tag">in your document somewhere</a>. I don't like this requirement because I prefer to have my post title be `h1` and all post content using `h2` or lower. I got around it with a somewhat hacky solution. I just made sure to include `# Table of Contents` right before the lines above, and replaced it manually in my post template. I also removed the inline styling on the list it generates because I prefer to use my own CSS.
</p></strike>

**Static Asset Combination**

As demonstrated by [Development Seed](http://developmentseed.org/blog/2011/09/09/jekyll-github-pages/) and others, you can utilize liquid `{% raw %}{% include file.ext %}{% endraw %}` tags to consolidate static assets for your site. Say you have a handful of javascript or css files which are included on each page. Simply move them to the `_includes` folder and create one consolidated file like so:
{% raw %}
	---
	---
	{% include file1.css %}
	{% include file2.css %}
	...
{% endraw %}
This will reduce the number of requests the browser has to make and speed up load times.

### Conclusion

With that, I think I'll close out my series of posts on Jekyll. I do intend to keep updating these posts as I run across new ideas and examples from other blogs. Please let me know if you run across any other handy tips or third-party services to add to this list in the comment thread.
