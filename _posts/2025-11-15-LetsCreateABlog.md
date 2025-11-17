---
title: Let's Create a Blog
date: 2025-11-15 12:00:00 -400
categories:
  - blog
  - homelab
tags:
  - git
  - github
  - obsidian
  - blog
  - homelab
  - linux
  - omarchy
  - jekyll
  - chirpy
media_subpath: /assets
---

# How Did We Get Here?
I was watching one of the [Simply Cyber](https://www.youtube.com/@SimplyCyber) videos, specifically the Cybersecurity Mentors podcast episode featuring Charles Carmakal. He mentioned that something he actively looks for is demonstration of someone's desire to learn where an example was keeping a blog. I've been wanting to *start* a blog for a least a year now, but after hearing how influential it is for someone to really **see** what I'm doing, what I'm passionate about, and what I'm learning I wanted to act.

In my voracious consumption of homelab and free open-source software (FOSS) content on YouTube-- hopefully a lot more of that to come-- I found a couple of videos that stuck with me for creating and hosting a blog.

## NetworkChuck
NetworkChuck is an incredible resource for people at any skill level and age to get introduced to and implementing cybersecurity technology, software, and devices. His content is perfect for providing people with the process and overview for how to get something done. In [this](https://www.youtube.com/watch?v=dnE7c0ELEH8) video in particular he goes over his process for creating and hosting blog. Is it over-engineered to the point of insanity, **YES**, but he brings up technology that I already use: Obsidian. It completely cross platform, and it has a solid ecosystem of community plugins to make things as simple or as complicated as one desires. It also uses Markdown, and it provides extremely effective linking between other Markdown files.

So now I can use my current note-taking application to write the blog, and I can go about writing in place where I am already comfortable.

In NetworkChuck's video he talks about a couple of other technologies, but I am particularly interested in Git and Github. Effectively every software professional uses Git and some sort of host for repositories. The most popular one is Github owned by Microsoft, and Github has an incredible feature for hosting *static* webpages. This means a couple of massive worries can be removed from my plate:
1. Purchasing and maintaining a domain;
2. Securing my home network with exposed services.
I won't have to purchase a domain-- Github provides one, and I won't secure my network since Github is the one hosting and serving the page.

## TechnoTim
Where NetworkChuck is more of a shotgun with cybersecurity and IT content, TechnoTim focuses more on the homelab and self-hosting side of things. His [video](https://www.youtube.com/watch?v=F8iOU1ci19Q) on Jekyll was far less overengineered which helps me to get a product created faster. Remember, I want to use Obsidian to create the markdown documents, but the rest is really about producing something with minimal friction. This video really provided that `minimal friction` mindset. While both Jekyll and Hugo satisfy my want for FOSS, I definitely preferred the [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) theme presented here, so Jekyll is my choice. 

This video also prioritizes hosting through Github itself rather than using the Github Actions to push the site data to a different host. I definitely prefer this method since I am trying to reduce the number of services I have to pay for and rely on. I eventually plan on transitioning this project from Github to a service like GitLab or Forgejo which is FOSS, and both of which provide static site hosting. Once I am far more comfortable and experienced, I plan to host those services *and* the site myself. For now, my focus is production.

With this I have all the pieces I need to get blog that satisfies the need with as little friction as possible, so now we need assemble the puzzle.

# Dependencies and Prerequisites
As I am using Linux, I will be providing the necessary dependencies and prerequisites for my operating system, but both videos provide ample resources for setting this up on your own operating system.

## Obsidian
Luckily for me Obsidian came installed by default on the OmarchyOS Linux distribution, but the website has an AppImage available to download.
The first time I opened Obsidian I was presented with the following:
![Opening Obsidian For the First Time](Obsidian_First_Open.png)

Now we just need to create a Vault-- which is just another way of saying folder-- in a convenient location, and we go on creating this Markdown document.

## Git
By default most platforms aside from Windows will have Git installed. If not, it can be easily installed through whatever package manager your system uses: Brew, Apt, Dnf, etc. 
While Github is also its own separate entity, I am assuming that you already have your own account with that as well.

## Jekyll
Jekyll's website has an up-to-date to listing of its requirements that can be found [here](https://jekyllrb.com/docs/installation/other-linux/). For me since Omarchy is built on Arch I use 
```bash
yay -S ruby base-devel ruby-erb
```

Once those finish installing we can follow the installation instructions to get Jekyll itself installed:
```bash
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc 
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc 
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc 
source ~/.bashrc

gem install jekyll bundler
```
## Chirpy
Please just go watch the TechnoTim video mentioned in the [[Let's Create a Blog#TechnoTim|TechnoTim]]. His explanation and walkthrough is phenomenal.
If you wish to follow the documentation for Chirpy, that can be found [here](https://chirpy.cotes.page/posts/getting-started/).
The short and skinny is that we need to create new repository using the Chirpy Starter as a template, make sure to name it using the Github Pages naming scheme, so for me it is `callycmd.github.io`, then clone the repository to my machine. Once the repo is cloned, we will need to navigate to that folder in the terminal, then executing:
```bash
bundle
```

# Building the Blog
We now have everything we need, and technically we are done! All of the dependencies have been successfully installed (*hopefully*), and a simple execution of 
```bash
bundle exec jekyll s
```
will produce a locally running and serving version of the static site:
![Chirpy Defaults](Chirpy_Base.png)

## Configure Chirpy
In the root of the clone repository there is a file named `_config.yml` where we can change the **global** site settings. Chirpy out-of-the-box (OOTB) supports an immense amount of globally configurable settings, but I have no need for most of them at the moment of writing this; again, the goal is to have as little friction as possible at this moment, and there is a lot of friction already.
I stopped the server while I was configuring the settings, but that may be unnecessary. It is really just what will, or rather won't, work for you.

## Deploy
I mentioned that Chirpy supports a lot of features, and one of the wonderful features is metadata like tags, date, and title. Obsidian calls this metadata `properties`, but since this is all just Markdown the implementation is standardized. I just added the corresponding properties to the document in Obsidian, but you can use a block of "---" at the beginning of the document to achieve the same results. Please see the screenshots below.
Obsidian properties:
![Obsidian Rendered Properties](Obsidian_Properties.png)

Markdown Standard:
![Markdown Text-base Front Matter](Markdown_Preamble.png)

With my metadata implemented, I now need to get my Markdown document and its assets into the repository (so much for *minimal* friction). I imagine eventually I will create a script to automate this process, but this should be a simple mirror:
	Send the photos in the Assets folder in my Obsidian Vault to the Assets folder in the repo
	Send the Markdown document to the Posts folder in the repo.
Simple `cp` command and re-run the server, right?

> No!

Please, please, please read the documentation on the required ***title*** scheme for the document itself. The page will not produce if it does not meet the `YYYY-MM-DD-Title.md` format.
Another small headache is that Obsidian makes inserting documents-- specifically photos-- really easy, when writing these pages in the future I need to use official Markdown syntax.
But once those changes have been made, I have my first successful post on localhost. Now let's put it on Github.

As TechnoTim mentions in his video, all we should need to do now is add the changes, commit the changes, then push the changes. Chirpy has a built in Git Action that should do the heavy lifting for us to get the page setup.
So, let's do that now! From the root of the repository:
```bash
git add .
git commit -m "The first blog post!"
git push
```
