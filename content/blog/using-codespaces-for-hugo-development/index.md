---
title: Using GitHub Codespaces for Hugo development
date: 2022-07-05T09:00:00Z
tags:
  - Hugo
  - GitHub
  - Codespaces
  - Developer Velocity
---

In this blog post, I try using [Github Codespaces](https://docs.github.com/en/codespaces) to write this blog post about using Codespaces to write blog posts in Hugo! Yes, it's meta 🤓

A GitHub Codespace is a cloud based developer environment attached to your GitHub repository. It contains the necessary dependencies to be able to work on your codebase in VS Code (in-browser or you can connect from your desktop!) and then commit your code. Having a Codespace means never having to get all those pesky project dependencies set up on your computer AND it means you're always working from a clean environment so "Works on my machine" should be a thing of the past! Awesome, right?!

So now that I have the ability to create a Codespace I figured I'd give it a go rather than installing Hugo on my new work machine, git cloning, and so on. I also opted to not read the docs which not my usual approach but I thought this stuff was intended to be easy so I figured I'd give it a go! 

First of all there was a button that said simply `Create codespace on main` and I thought "Nah, it can't be that simple right?". I figured I'd need to take the `Configure and create codespace` option instead to get into to telling it to install Hugo etc. Well this took me to a wizard that gave me only a few options (branch, region, and machine type) and all of it was set appropriately so I pressed `Create codespace` and away I went. It took less than a minute and then I was looking at VS Code in the browser!

{{<
    amp-image src="default_codespace.png"
    height="90"
    width="160"
    layout="relative"
    alt="Screenshot of VS Code in browser thanks to GitHub Codespace"
>}}

This was an editable instance of my codebase and I'd noticed it had created a `.hugo_build.lock` filte and few directories like `resources` but I figured that it doesn't have all my Hugo dependencies so I'd better learn how to create a custom codespace so I started clicking on the links in the terminal about custom containers.

I had just created a branch via the Codespace where I intended to both write this blog post and build my custom container, when I decided "What the heck, see if Hugo works in the terminal" and with a quick type of `hugo` that container took 166ms to generate my entire blog site. Wow!

Knowing that I may not have to build a custom container at all, I've progressed the blog post to this point. My next step is to see what will happen if I run `hugo serve` -- will I be able to access the live preview of the site?

Drumroll please...

I can! 👏👏

There is now a Ports tab available where I can see it active and it gave me a popup with a big green button that opened up my site.

{{<
    amp-image src="running_hugo.png"
    height="90"
    width="160"
    layout="relative"
    alt="Screenshot of VS Code in browser with a popup offering to open the Hugo site"
>}}

Brilliant, it worked. Browsing to my site I can see it's generated the blog post and so on. The problem I now have is that the image isn't rendering because it's looking for `localhost:1313` in the image URL and I get a similar issue when I click through to the post itself.

{{<
    amp-image src="localhost_issue.png"
    height="90"
    width="160"
    layout="relative"
    alt="Screenshot of browser showing Edge's DevTools Inspect results on the left hand side and the blog on the right hand side with an empty space where an image should be"
>}}

This is a bit of a hiccup but this is to do with my love of generating absolute URLs versus relative URLs and Hugo starting with the default base URL of `http://localhost:1313`. Let's see if I can change that to the URL of the preview page Codespace generated (`stephlocke-stephlocke-github-io-wrggw7rjh6g9-1313.githubpreview.dev`).

I've just run `hugo serve -b https://stephlocke-stephlocke-github-io-wrggw7rjh6g9-1313.githubpreview.dev` and it worked! Well sort of. 🤔 Unfortunately, Hugo is still injecting the port into some URLs and it isn't something I can just use the terminal to get around. It's down to how I've coded the shortcodes and partials in Hugo. I'm using a Hugo modules to bring in the theme, I'm not able to easily edit the theme like I would if it was a git submodule or something I kept in the repo. I guess tidying the theme up whilst in Codepaces is a topic for a future blog post.

For now though, I'm able to get started with Hugo in-browser in less than a minute making it so that I can author new content much more easily. Hopefully, that'll mean I blog more! It was incredibly simple to get up and running with the only real change I had to make to accomodate my specific blog setup was to change the base URL when serving the Hugo site. It's really quite amazing how easy it all was. 

Thank you for reading this text-based live stream of me trying out this snazzy new tool. 🙏