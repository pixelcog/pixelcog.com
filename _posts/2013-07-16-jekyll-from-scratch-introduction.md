---
layout:  post
title:   Jekyll From Scratch - Getting Started
tags:    jekyll
image:   jekyll-pt1-intro.gif
updated: 2014-08-21 @ 7:31pm
---

[Jekyll](http://jekyllrb.com/) + [GitHub Pages](http://pages.github.com/) is the web platform underlying the new and improved PixelCog.com. I've decided to document my experience with it as I go along, consolidating best practices, tips, and tricks into a helpful guide to use for my own reference and the benefit of anyone else out there who might learn from my experience.

{{ more }}{% raw %}

### Table of Contents
{:.no_toc}

* Table of Contents Placeholder
{:toc}

## Blogging for Hackers

Although this is my first serious foray into blogging, I've had some experience with a few platforms in the past including Blogger, Tumblr, and WordPress. I've also used the latter as a content management system for some clients in past projects. They have generally gotten the job done, but when it came time to implement PixelCog.com, my occasional frustrations with them have driven me to experiment with some new options.

Being a professional web developer, I decidedly don't want a solution which sacrifices flexability so that it can hold my hand through things I can handle myself. I wanted a completely hackable blog platform. As I set out to do some research, I had a few criteria in mind:

- **Automation** — I don't want to create everything from scratch. I need a platform that will allow me to provide the common trappings of a modern blog (rss, pagination, tagging, content management, etc.) without a ton of effort.
- **Control/Portability** — I want to have full control over the source material. I don't want to publish it just to have it exist in some content silo where I cannot easily export it. Version control is a huge plus too.
- **Staying Power** — I don't want to build a website on top of some new flashy platform with no established community, nor do I want to use a free venture-funded blog service with an unknown monitization strategy.
- **Text Formatting** — I have been using [Markdown](http://daringfireball.net/projects/markdown/) for years and it is far and away the best markup language to manage hypertext content in a natural, readable, portable way. I don't want to be forced into a choice between spaghetti code generating [WYSIWYG](http://en.wikipedia.org/wiki/WYSIWYG) editors, overly simplistic BBCode, or manual HTML.
- **Customization** — I want every bit of HTML, JavaScript, and CSS sent to the browser to be under my control, and I want a robust templating system to make managing it painless.
- **Simplicity/Scale** — If self-hosting I don't want unneccesary database overhead or legacy one-size-fits-all code impeding my ability to maintain a simple, scalable hosting solution.
- **Security** — If self-hosting I don't want to become a server maintenance expert to run a simple website. I am competent enough to do it, but I also know that to do it well can be a full time job, and I wear enough hats in my job already.

Given these criteria I set out to look at a few solutions. The first one I landed on was [Scriptogr.am](http://scriptogr.am/). At first glance it seemed like it would fit quite well. It takes Markdown files hosted in a Dropbox folder, compiles them using a simple customized template, and hosts them for you for free. My main reservations were the absense of a monitization strategy (there's no such thing as a free lunch), a few [unfavorable anecdotes from switchers](https://twitter.com/alexcican/status/342574618055438336), and an apparant lack of active development (as of this writing, the last development blog update was 10 months ago in Sept 2012). Scriptogr.am has some promise, but I'd rather give it a few years to incubate and see where it ends up.

A little later I heard about an open-source project called [Octopress](http://octopress.org/) from the folks on the [/dev/hell podcast](http://devhell.info/) and it appeared to represent nearly everything I was looking for; Markdown support, incredibly easy self-hosting, and extreme flexibility while still providing a solid framework with which to build my website. Perfect!

Deciding to give it a try I dug deeper and learned that Octopress is actually a bootstrapped layer on top of another project called Jekyll. Having heard this I decided to give vanilla Jekyll a try before dealing with any (potentially unnecessary) layer of abstraction provided by Octopress.

## Intro to Jekyll and GitHub Pages

Jekyll is a powerful static website generator. It takes raw markup files and templates and compiles them into a complete, static html website. Since it requires no server-side scripting components it can be hosted on bare bones file servers like [Amazon S3](http://aws.amazon.com/s3/) rather than elaborate server stacks with a scripting language like PHP tied to a SQL database where you need to worry about things like security and scalability.

Jekyll happens to be the engine behind "[GitHub Pages](http://pages.github.com/)" which is easily Jekyll's most attractive hosting solution. With it you can simply upload your Jekyll blog to a [GitHub](https://github.com/) repository and have it automatically compiled and deployed through a post-receive hook each time you commit. Through this setup, publishing your blog to GitHub becomes as simple as typing `git push`! The service is free, has unlimited-bandwidth (AFAIK), and allows custom domains, and since I already use [git](http://git-scm.com/) for all of my professional projects, this workflow felt right at home.

The one catch to using GitHub Pages is its lack of plugin support.  For obvious reasons, GitHub doesn't want people executing arbitrary ruby code on their servers, so Jekyll is run in _safe_ mode.  This can be [worked around](http://edhedges.com/blog/2012/07/30/jekyll-with-plugins-hosted-on-github-pages/) by adding an extra build-step prior to publishing or by automating deployment with rake files, commit hooks, or shell scripts, but I have chosen to forgo plugins and go the simple route for a few reasons:

1. Using a vanilla Jekyll deployment is more portable, simple, and elegant. I'm not a fan of unnecessary complexity.
2. GitHub Pages default deployment solution requires no additional setup. I could go to any computer, clone my repo, make some changes, and push without mucking around with commit hooks, installing dependencies, and so forth.
3. Most anything you'd want to do with your blog can be done without plugins.

I have decided to keep my blog **completely plugin free and GitHub Pages compatable**, utilizing only Liquid, Markdown, and core Jekyll functions. If you'd like to work with plugins I won't be covering any of that here, but there are [many](http://octopress.org/docs/plugins/) [other](http://www.jekyll-plugins.com/) [resources](http://jekyllrb.com/docs/deployment-methods/) out there to help you get past those logistical hurdles.

### Finding Good Information

Jekyll may be [rising in popularity](http://www.google.com/trends/explore?q=github%20pages%2C%20jekyll%20github%2C%20jekyll%20bootstrap), but its community is still quite small, and Google is not yet good at curating its more helpful online resources. Jekyll's own [official website](http://jekyllrb.com/) and docs rank below other search results on several Google queries. The project site could certainly use some good [link juice](http://en.wikipedia.org/wiki/PageRank) from blog posts like this and others.

The project recently reached maturity with its [1.0.0 release](https://github.com/blog/1502-jekyll-turns-1-0) (May 2013), bringing with it support for drafts, excerpts, and many other enhancements. Many Jekyll guides and resources, inlcluding popular child projects [Octopress](http://octopress.org/) and [Jekyll-Bootstrap](http://jekyllbootstrap.com/), still disseminate obsolete solutions aimed at patching-in some of these enhancements through plugins. This blog post is largely a reaction to the lack of post-1.0, non-plugin-dependent resources out there for new Jekyll users.

I suspect many of the remaining plugin-based workarounds will also be made obsolete as popular features continue to get rolled into core. [Tag and category pagination](https://github.com/mojombo/jekyll/pull/870), more robust [templating features](https://github.com/mojombo/jekyll/pull/1204), and Octopress-like [publishing workflow automation](https://github.com/mojombo/jekyll/pull/1148) aren't far down the pipeline.

## Getting Started

Getting your feet wet with Jekyll is very simple. I find it best to force installation of the same versions of Jekyll and Liquid [currently used on GitHub Pages](https://github.com/github/pages-gem/blob/master/lib/github-pages.rb#L9) to ensure feature parity (<strike><em>1.0.3</em> and <em>2.5.0</em> respectively as of July 2013</strike>)(*2.2.0* and *2.6.1* as of August 2014). Assuming you already have [ruby installed](http://www.ruby-lang.org/en/downloads/), following Jekyll's [Quick-Start Guide](http://jekyllrb.com/docs/quickstart/) is as simple as this:

	$ gem update --system
	$ gem install liquid -v 2.6.1
	$ gem install jekyll -v 2.2.0
	$ jekyll new blog
	$ cd blog
	$ jekyll serve -w

Now simply go to `localhost:4000` and marvel at your creation...  
(*`crtl+c` when you're done*)

> **Update — Nov 22nd, 2013**:  GitHub Pages now has its own gemfile to install the currently used versions of Jekyll and its dependencies. You can now run `gem install github-pages` in place of the above install commands. More information [here](https://help.github.com/articles/using-jekyll-with-pages#installing-jekyll).

> **Update — Aug 21st, 2014**:  Fixed Jekyll and Liquid versions.

### Writing Your First Blog Post

Looking at the shell directory structure that we just created you'll notice a `_posts` subdirectory.  This is where your published blog posts will live. These must all follow the naming convention `YYYY-MM-DD-name-of-post.ext`. The example post uses the extension `*.markdown` but I much prefer the sussinct `*.md` extension; it makes no difference either way. Jekyll will infer the date and permalink slug of your post from this name unless overridden in the [YAML front-matter](http://jekyllrb.com/docs/frontmatter/) (the content between the lines with three dashes at the start of the file):

	---
	layout: post
	title:  "Welcome to Jekyll!"
	date:   2013-05-24 18:34:38
	categories: jekyll update
	---
	
	You'll find this post [...]

Beginning with the recent release of Jekyll 1.0, we can now create and preview blog post [drafts](https://gist.github.com/benbalter/5555992) without mixing them in with published posts and conforming to this rigid file naming convention. Simply create a `_drafts` folder in the root directory of your blog and add a new file to it. Let's call it `hello-world.md`:

	---
	layout: post
	---
	
	# Hello World
	This is my first _ever_ Jekyll post!

Save the file and run Jekyll with the `--drafts` flag to preview your work.

	$ jekyll serve -w --drafts

Go back to `localhost:4000` and you should see your new post listed under today's date. Jekyll infers the post title from the filename if none is provided, and uses the file's last-modified date as the publication date.

Now when you've finished a draft and decide to publish it, the process is pretty straight forward. Move the file from `_drafts` to `_posts`, add the year formatted as `YYYY-MM-DD-` to the start of the filename. You may then wish to configure the front-matter to add meta-data like tags, categories, permalink, et cetera.

**Pro Tip** — I like to store my blog post drafts where I can view and edit them on the go. To do this, I made my `_drafts` folder a symbolic link to a folder in my Dropbox. Using this method I can access my drafts from [Byword](http://bywordapp.com/) on my iOS devices and update them from anywhere.

### Rendering Engine Basics

Files in `_posts` are rendered according to the default permalink rule (`/:categories/:year/:month/:day/:title.html`) which can be overridden in `_config.yml` or the post's own front-matter (like `permalink: /foo/bar/`).

Page templates are assigned with the `layout` front-matter variable and can be found in the `_layouts` directory. Jekyll provides two by default, `default.html` and `post.html`. Template include files are found in `_includes` and can be embedded with `{% include file.ext %}`.

All other files follow these four rules:

1. Files and directories which begin with `.` or `_` are ignored.  
   _(unless overridden in `_config.yml`)_
   
2. Files *without* YAML front-matter are passed through when rendered.  
   _(`/css/style.css` or `/foo.html` with no front-matter are copied verbatim)_  
   
3. Files *with* YAML front-matter are processed with Liquid and (if appropriate) rendered in Markdown with an appropriate file extension change.  
   _(`/file.md` with front-matter becomes `/file.html`)_
   
4. Files with YAML front-matter which specify a permalink are preprocessed and rendered at the given url. If the permalink is a directory, index.html is used.  
   _(`permalink: /fancy/link/` yields `/fancy/link/index.html`)_

Once you understand these simple rules, you'll have a pretty firm grasp of Jekyll's rendering engine.

### Liquid Template Engine

Going over the intricacies of the Liquid templating language is beyond the scope of this tutorial, but there are some good references [here](http://jekyllrb.com/docs/templates/) and [here](http://wiki.shopify.com/Liquid). Anyone who has used a basic templating language before will find all the usual trappings — for loops, if/else logic, variable assignment, etc. I'd encourage you to play around with it and explore the [official Jekyll documentation](http://jekyllrb.com/docs/home/) before proceeding with the more advanced stuff in my next posts.

## Publishing Your Site with GitHub Pages

Once you've edited your first post and played around with the templates, you'll want to publish your work. Provided you already have [git installed](http://git-scm.com/book/en/Getting-Started-Installing-Git), this is a pretty simple process.

Following the instructions on [help.github.com](https://help.github.com/articles/using-jekyll-with-pages), you'll need to create an empty repository named `{user or org name}.github.io` and then perform the following actions:

	$ git init
	$ git add -A && git commit -m "initial commit"
	$ git remote add origin git@github.com:username/username.github.io.git
	$ git push -u origin master

_* remember to replace username with your user or org name_

Within 10 minutes (but usualy much quicker) your Jekyll blog will be rendered and online!

### Enabling a Custom Domain

If you want to enable a custom domain name on GitHub Pages, you can perform the following steps (also from [help.github.com](https://help.github.com/articles/setting-up-a-custom-domain-with-pages)):

Go to wherever you're domain's DNS zone is hosted and edit your `A` record to point to `204.232.175.78`.

Add a `CNAME` file to the github repo to tell it to use our domain:

	$ echo "mydomain.com" >> CNAME
	$ git add -A && git commit -m "enable custom domain"
	$ git push

_* replace mydomain.com with your domain name_

Wait for the DNS zone changes to propogate (up to 48 hours, but usually much sooner), and enjoy!

{% endraw %}

### Other Deployment Options

GitHub Pages may be my deployment option of choice, but it certainly isn't the only option. As mentioned before, Amazon S3 is another good, low-cost solution. This and many other options and deployment automation techniques can be found on [Jekyll's official website](http://jekyllrb.com/docs/deployment-methods/).

## Migrating an Existing Blog

Jekyll has built support for importing blog data from a large number of blogging platforms including Tumblr, WordPress, Blogger, and many others. Visit [Jekyll's migration page](http://jekyllrb.com/docs/migrations/) for more info.

In my [next post]({% post_url 2013-07-18-jekyll-from-scratch-core-architecture %}), I'll cover ways to maintain urls you were using from your old blog so you don't break all of your inbound links.

## Helpful Resources

- [Jekyll Official Documentation](http://jekyllrb.com/docs/home/)
- [Jekyll Changelog](http://jekyllrb.com/docs/history/)
- [Jekyll Template Reference](http://jekyllrb.com/docs/templates/)
- [Liquid Engine Reference](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers)
- [Liquid Filters Reference](http://wiki.shopify.com/Filters)
- [Markdown Reference](http://daringfireball.net/projects/markdown/syntax)
- [GitHub Pages Help Section](https://help.github.com/categories/20/articles)
- [Development Seed's Jekyll Blog](http://www.developmentseed.org/blog/2011/09/09/jekyll-github-pages/) — really helped me understand the basics of Jekyll's rendering engine when I started out, and provides some really clever examples of what you can do with Jekyll.
- [Sebastian Teumert's](http://teumert.net/en/2012-08-27-release-jekyll-template-toolkit.html) [Jekyll Template Toolkit](https://github.com/NetzwergX/jekyll-template-toolkit/) — an indispensable plugin-free Jekyll resource.
- [Tobias Sjösten's Blog](http://vvv.tobiassjosten.net/jekyll/) — has some nice tips and tricks for Jekyll, also plugin-free.
- [StackOverflow's Jekyll Tag](http://stackoverflow.com/questions/tagged/jekyll) — the place to go to ask questions if you get hung up.

## Final Thoughts

One of the biggest boons for budding young web developers in the last few decades has been the ability to hit "view source" to instantly see what's behind the curtain of their favorite website. For a kid who begged his parents for [triangle shaped screwdrivers](https://en.wikipedia.org/wiki/List_of_screw_drives#TA) so that he could dissect his McDonalds toys and put them back together, I found this feature invaluable when I first cut my teeth on HTML in my early teens. It's always easier to learn by observing real-life examples.

This is one reason why Jekyll's pairing with GitHub Pages is so awesome. Many of the Jekyll websites out there are hosted on public GitHub repositories, free to anyone who wants to dissect them and see what makes them tick. You can view the source material for [this post](https://github.com/pixelcog/pixelcog.github.io/blob/master/{{page.path}}), and even [this entire website](https://github.com/pixelcog/pixelcog.github.io/) on GitHub. One of the best ways I've found to learn it is to find a Jekyll blog you like and take a look at how they do things.

Go forth and start [blogging like a hacker](http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html)!

In my [next post]({% post_url 2013-07-18-jekyll-from-scratch-core-architecture %}) I'll be covering some more advanced website structural configuration with Jekyll.
