# Writing a (bad) Static Site Generator

##### _December 2023_

<img src="https://github.com/hydrol0x/blog/assets/34951139/46c06e9c-92a6-4aeb-acc2-d920182371b6" alt="A robot builds up an html page." width=850>

## Hello World!

To create this blog I spent a few minutes looking up existing tools, but after getting lazy I decided to just write a quick script on my own. As always, it would definitely have been quicker to read up on some existing generator like [Jekyll](https://jekyllrb.com/) (ironically, Github pages already uses it when hosting this website) and it would also have been less janky— but then I wouldn't learn as much!

If you're already familiar with static sites, html, markdown, etc., you can skip to [the technical details of my generator](#technical)

## What Is a Static Site?

Static sites are basically what a 'normal' website is: pre-written code that doesn't change once it's loaded up on a server and which is read in, executed by the browser, and rendered as some page.

### Dynamic Websites

Nowadays, many websites are dynamically generated. Often, this means that the actual raw HTML/source for the website is sometimes barebones and the rest of the site content is injected in by lots of JavaScript. This is how frameworks like React go about things. The key benefit is that you, for example, no longer need a `blog.html`, `index.html`, `about.html`, etc.; instead, you can dynamically change the HTML of one page to pretend that there are many different views/pages. Similarly, you can change a lot based on user input, reuse code through components, and so on.

### Benefits of Static Websites

Although many of these features are nice, they can become bloat over a simple site that only has unchanging content. A blog, which is only occasionally updated with new content (a.k.a new HTML to be generated) doesn't really need a bunch of dynamic pages and JavaScript. Basic HTML and CSS can do what they shine at. In the end, the site loads faster, there's less JavaScript, and in theory less complexity overall.

## What Is a Static Site Generator?

To understand what a static site generator is, it is first important to note its use case, at least in the context of a blog.

### Raw HTML

In theory, since all of the code is pure HTML/CSS, it would be possible to just edit the source. That would mean that every time I wanted to write a blog post, I would have to sit down, open the website HTML in my code editor, and work inside a `<p>` tag with the occasional `<i>` or `<h1>`. This kind of sucks, since ideally, you want to separate the end user experience (well, the only end user is me... but still) from the coding part and you also want there to be a somewhat decent UI.

### GUI

On the other hand of the spectrum, there are websites like Medium and Substack that let you write in an appealing online GUI where you can see how your post will look while writing. Though nice, that goes beyond the territory of a static site generator or my skillset/time, so I decided on a middle ground.

### Markdown

Those nice GUIs often employ text entry using `Markdown`, a simple markup language that is easy to read, write, and parse into viewable content. The static site generator I built takes in these markdown files and converts them into HTML pages (with CSS styling) that follow the structure of the original. The benefit of writing markdown over raw HTML is that the base file is already human readable and so it's much easier to write. For example, a sample heading and paragraph might look like

    # Heading

    To create this blog I spent a few minutes looking up existing tools, but after getting lazy I decided to just write a quick script on my ow- wait this is just this post!

Which is much nicer on the eye than a soup of HTML tags.

### Markdown ➡ HTML

Now that we understand the starting point (a markdown document) and endpoint (an HTML page the browser understands) it is easy to see what the static site generator _should_ do. Namely, get us from point **A** to **B**, markdown to HTML.

Converting the markdown source file into HTML that the browser can understand isn't too hard since markdown is a simple specification. Doubly so when there are plenty of existing parser libraries that let you disregard the inner workings of conversion. The reason that writing this entire generator isn't so trivial is because, on top of having a page, I want to have a nice page that links other posts, is styled to be readable, and so on. To accomplish this, the static site generator

1. takes in a set of markdown files (in a folder like `posts/`)
2. converts the contents to HTML
3. Adds re-used HTML code like a navigation bar and footer
4. Creates a homepage with the new post links added
5. Generates a CSS file that contains the styling for the posts themselves and any other elements

## <a name="technical"></a>Technical Details and Implementation

I used Python for the generator since anything else seems overkill. My script roughly follows the steps outlined above.

### Parsing / HTML generation

First, it finds all posts in a directory that contains the raw markdown. At first, I had it check if files were modified or new, but after this got annoying, I removed the code (I should probably fix it up and add it back later).

Next, the markdown is fed to a python [markdown parsing library](https://pypi.org/project/marko/), `marko`, and then written to the respective `.html` file in a distribution directory. I am using `post/` so that the URL looks nicer, for example `blog.jacobryabinky.com/post/Hello World`.
Note: the reason that this URL works (no `.html`) is because of the way that Github pages deploys the files.

To add on other parts of the site, such as the navigation bar, footer, and `head` element, I decided to recreate components in a worse way! I think there is now some way of doing this better through something like web components, but I have never used or looked into them, so that could be part of an update to make my code less messy.

Right now, these elements are stored in a [directory](https://github.com/hydrol0x/static-site-generator/tree/680a0ea707606e44924739d40aa638d54b6cd858/elements), `elements/`, which contains subfolders for each element that contain the necessary HTML and CSS files. In the script, I [find](https://github.com/hydrol0x/static-site-generator/blob/680a0ea707606e44924739d40aa638d54b6cd858/static-site-generator.py#L12) these files, read in the text, and append or [insert](https://github.com/hydrol0x/static-site-generator/blob/680a0ea707606e44924739d40aa638d54b6cd858/static-site-generator.py#L67C11-L67C11) it somewhere in the generated page (for example, nav bar at the top, footer at the bottom, etc.). Similarly, for CSS, I just append all of the component elements CSS files into the main `index.css` style sheet and rely on class names for scoping styles.

The components are inserted into the [newly generated blog post](https://github.com/hydrol0x/static-site-generator/blob/680a0ea707606e44924739d40aa638d54b6cd858/static-site-generator.py#L63C24-L63C24) HTML files [as well as](https://github.com/hydrol0x/static-site-generator/blob/680a0ea707606e44924739d40aa638d54b6cd858/static-site-generator.py#L99C22-L99C22) the `index.html` file serving as the blog's homepage using the `BeautifulSoup4` library, which lets me parse the HTML tags and insert or append directly into the `body`/a list/etc.

### Bespoke all the way down

<img src="https://github.com/hydrol0x/blog/assets/34951139/b6a9deab-4551-4853-8d13-7a3fa834e4f2" alt="sphagetti code" width=500>

The main reason my generator kind of sucks right now is that almost everything is hardcoded in, which works since I'm using it for just my blog site, but isn't great if I want to make changes or use it for other projects. The next steps will probably be to look into components and to abstract away some of these steps.

#### A Short Todo List:

- Better component system
- Metadata in markdown files
- Better blog homepage styling
