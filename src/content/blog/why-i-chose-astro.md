---
title: 'Why I Chose Astro'
description: ''
pubDate: '2026-09-21'
tags: ["software", "astro", "javascript", "blog", "data sovereignty", "data ownership"]
---

If you look at the previous incarnations of my website, you'll notice that I did post updates in relation to things that I was working on. I both simultaneously over engineered and underengineered this approach. It relied on a Wordpress account which the website would then access via the Wordpress API and then I'd have full flexibility on how I wanted to display those posts. This let me post updates without having to redeploy the whole website, with the advantage of using Wordpress as a content management system (CMS), meaning that I didn't have to build my own.

The flaw with this approach is that a bit of downtime for a deployment is completely acceptable my use case, so integrating with a third party is overkill.

I've wanted to get into writing a blog for a while since a good way to develop your skills is to be able to break them down and explain them, and my website is a great place for me to do that. There were a couple of approaches I could have taken with this. I could have kept things how they were, using Wordpress as a CMS and pulling my posts via the API. I could have migrated everything to an existing CMS service, such as Wordpress, and either used that as my website or built a website around it. A third option would be to integrate the blog into my website, hosting all of the pages within itself and building a UI around that.

An episode of [Scott Hanselman's Hanselminutes Podcast](https://www.youtube.com/@shanselman) really helped cement my decision. Unfortunately it was a while back (I've been sitting on this idea for some time) so I can't remember which episode it is exactly. To badly paraphrase, Scott says that the reason he self hosts his own blog is so that he's in full control of his writing. He says that his issue with putting writing directly on social media is that you're at the whims of that company. They could go out of business, remove your posts, terminate your account etc. so hosting your own writing is the only way to retain full control over it. I completely agree with Scott on this which is why I decided to self host. The decision was also made easier since Wordpress is going through what I'll call here a "turbulent time", making Scott's point seem more likely if you're using their platform.

So why Astro instead of every other option?

1. Let's me use layouts and components like any React app 
2. Lets me write in markdown which I find cleaner than having HTML tags everywhere
3. It's not a SPA so easier to do SEO
4. It's built with blogs in mind with one of the tutorial projects being to build a blog, and a blog is one of the starter templates
5. It seemed fun!

Points 2 and 5 are the major ones here as they reduce the friction of starting to write, meaning that it's easier to just get writing instead of dealing with the technical side of things before even starting.

Of course, every choice has drawbacks so here's some of the downsides. The first is that I can't release a new post or update without some downtime, but as previously mentioned, this isn't a critical service and a deployment is fast. The second downside is that I'm in the Node ecosystem, and NPM is a big attack vector so I'll need to watch my dependencies.

There was a third downside but I've got a solution for that. With using something like Wordpress, I've got the flexibility of being able to log in on any browser and write a post. With using Astro, it'd mean making sure I've got Git installed, cloning the repo, downloading an editor or choosing to write a new blog post in Notepad. That's not actually the case though! I can access my repo directly in Github, add a new file and preview the markdown, so that turns this negative back into a positive!