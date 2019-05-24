---
layout: post
title: "Jekyll and GitHub Pages"
comments: true
---
<style>
	.demotable {
		width: 100%;
	}
	.demotable td {
		width: 50%;
	}
</style>
>"Jekyll is a simple, extendable, static site generator. You give it text written in your favorite markup language and it churns through layouts to create a static website. Throughout that process you can tweak how you want the site URLs to look, what data gets displayed in the layout, and more." [Jekyll Docs: Quickstart](https://jekyllrb.com/docs/)

>"Websites for you and your projects. Hosted directly from your GitHub repository. Just edit, push, and your changes are live." [GitHub Pages](https://pages.github.com/)

# Introduction

##### Typical Content Management System

When the server receives a page request, the Content Management System (CMS) generates the page on the fly[^1] by combining the template with meta data and user generated content.  The resulting page is returned to the browser requesting it.

[![CMS Diagram](/assets/images/2019/05/21/jekyll-and-github-pages/cms-diagram.png)](/assets/images/2019/05/21/jekyll-and-github-pages/cms-diagram.png)

##### Jeykyll and Host

Jekyll generates a site offline, which is then uploaded to a server.  Effectively the page construction is the same as a CMS.

[![CMS Diagram](/assets/images/2019/05/21/jekyll-and-github-pages/jekyll-and-host-diagram.png)](/assets/images/2019/05/21/jekyll-and-github-pages/cms-diagram.png)

*	GitHub has its own Jekyll interpreter, so you push the files to the repository and GitHub will pass them through Jekyll and then serve them too.
*	Sass and CoffeeScript are optional.  If you use either, Jekyll will pass these through their respective interpreters; otherwise, any CSS or JavaScript files will be copied to the site folder.

---

# Jekyll

##### Jekyll Pros
*	Security (no database or PHP security issues)
*	Speed (HTML and image only)
*	GitHub integration (free hosting)
*	1337

##### Jekyll Cons
*	Third-party functionality
	*	Forms
	*	Comments
*	Hosting synchronization

---

### Prerequisites

*	Ruby >= 2.4.0[^2], including development headers: `ruby -v`

*	RubyGems: `gem -v`

*	GCC: `gcc -v`

*	Make: `make -v`

*	git (for version control and GitHub integration): `git --version`

---

This tutorial relies on the official Jekyll documentation as it's easy to follow.

### Quick Start

<https://jekyllrb.com/docs/>

### Creating Blog Posts

<https://jekyllrb.com/docs/posts/>, covering sections: [The Posts Folder](https://jekyllrb.com/docs/posts/#the-posts-folder) and [Creating Posts](https://jekyllrb.com/docs/posts/#creating-posts)

### Creating Standalone Pages

<https://jekyllrb.com/docs/pages/>

### Front Matter

<https://jekyllrb.com/docs/front-matter/>

For this presentation, we'll only cover custom variables and the predefined variables *layout* and *title*.

### Themes

Previous version of Jekyll stored the theme files in the same directory as the other files for the site.  The current version uses gem-based themes and these are located in your user gems folder.  The Gemfile, located in the root of your site, indicates the location of the theme and the theme used is specified in the _config.yml.

Both have the basic directory structure:

	assets
	_includes
	_layouts
	_sass

*	**assets** typcially contains template resources other than HTML and CSS, such as: images, videos, etc...  (You may put other post/page resources here too.)

*	**_layouts** contains the templates used by Jekyll to format your posts and pages.  For example, if you specify `layout: foo`, Jekyll expects to find an HTML file named *foo.html*.  The default layout is *default.html*.  Typically themes have layouts for posts and pages, named *post.html* and *page.html* respectively.

*	**_includes** contains snippets of code included in other files.  For example, most themes include a *header.html* and *footer.html*, which are included in the layout file(s).

*	**_sass** [optional] is the location of `*.scss` files used to generate CSS for your site.

If you want to change a gem-based theme, you create the directory or directories that correspond to the files you want to change.  You then add file(s) with your changes to these directories.  Think of this similar to function overloading or WordPress Child Themes.  To view the theme's contents, execute: `tree $(bundle show minima)`

Changing themes is beyond the scope of this presentation.

##### Further Reading:
*	<https://jekyllrb.com/docs/themes/>

---

## GitHub Pages

GitHub offers two types of web hosting: project (per-project) and user/organization (per-account).  Both also allow you to use your own domain name instead of the generic *github.io*.

The main differences between project and user/organization sites are:

*	All commits must be made to the *master* branch when working with a user/organization site.  Project commits may be made to the *master* branch; a */doc* folder; or, a different branch, such as *gh-pages*.
*	The URL for a user/organization site is *username.github.io*.  For a project site, the URL is *username.github.io/project*.
*	When using a custom domain name, the domain name replaces the username.github.io.

These steps will walk through creating a user/organization site.

1.	Create a new repository named, `username.github.io`.  Ensure that this repository is **Public**.

1.	Navigate to your new repository and click the **Settings** menu option.

1.	Scroll to the **GitHub Pages** section.

1.  [Optional] Click **Change Theme** to configure your theme

You can then develop your site directly on the GitHub website or clone your site and push your changes to GitHub.

##### Further Reading:
<https://pages.github.com/>
<https://help.github.com/en/articles/user-organization-and-project-pages>

---

### Custom Domain

1.	On the Settings page of your *username.github.io* repository, scroll to the **GitHub Pages** section.

1.	Type the domain name in the **Custom domain** field and click **Save**.  (Do this first so another user can't hijack your domain name.)

1.	Visit your domain name DNS provider to set up an apex domain with a 'www' subdomain.

	1.	For an apex domain, create four A records with the following IP addresses:

		*	`185.199.108.153`
		*	`185.199.109.153`
		*	`185.199.110.153`
		*	`185.199.111.153`

	1.	[Optional] Add a CNAME record `www` that points to the hostname, `username.github.io.`.  (The period after the the GitHub address is mandatory.)

	The end result should look similar to this screenshot:

	[![DNS Records example](/assets/images/2019/05/21/jekyll-and-github-pages/dns-records.png)](/assets/images/2019/05/21/jekyll-and-github-pages/dns-records.png)

##### Further Reading:
*	<https://help.github.com/en/articles/quick-start-setting-up-a-custom-domain>
*	<https://help.github.com/en/articles/setting-up-an-apex-domain>
*	<https://help.github.com/en/articles/setting-up-a-www-subdomain>

---

## Demo Sites

| Website | Source |
|:-|:-|
| <https://gordon.celesta.me> (hosted by GitHub) | <https://github.com/va7sdf/va7sdf.github.io> |
| <https://artbergmann.com> (self-hosted) | <https://github.com/va7sdf/artbergmann> |
{: .demotable}

---

## References

*	Jekyll: <https://jekyllrb.com/>
*	Ruby: <https://www.ruby-lang.org/en/>
*	Ruby Gems: <https://rubygems.org/>
*	YAML: <https://yaml.org/>
*	Markdown: <https://daringfireball.net/projects/markdown>
*	Liquid: <https://github.com/Shopify/liquid/wiki>
*	Sass: <https://sass-lang.com/>
*	CoffeeScript: <https://coffeescript.org/>
*	GitHub Pages: <https://pages.github.com/>

---

## Notes

[^1]:	I that acknowledge most CMS cache pages once they are created but, for this simplistic diagram, we'll assume this isn't the case.
[^2]:	Ruby is not installed in Raspbian by default and the version in the Raspbian repositories is 2.3.3.  (You can use `apt-cache show ruby | grep Version` to verify the repository version.)

	1.	Issue these commands to install the most current version using Ruby Version Manager (RVM):

			sudo apt install build-essential curl nodejs
			gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
			curl -sSL https://get.rvm.io | bash -s stable --ruby

	1.	Add the following to your `.bashrc` file:

			[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm"

	1.	Restart your current terminal session.

	1.	Verify the current version of Ruby:

			ruby -v
