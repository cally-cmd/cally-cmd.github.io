---
title: Reducing Friction
date: 2025-11-19 19:18 -400
categories:
  - blog
  - homelab
tags:
  - blog
  - chirpy
  - git
  - github
  - homelab
  - obsidian
  - automation
  - TypeScript
  - Bash
media_subpath: /assets
---
> Obsidian calls documents Notes. I will be using note, post, and document interchangeably throughout.

## The Goal
In my previous post, my entire focus was to get blog up and running, and for me to publish my first post as frictionless as possible. Honestly, it is plain to see that I really more focused on having something that conformed to the following:
1. Allowed me to write the posts with an application I already use and enjoy;
2. The blog site wasn't hosted by me (yet);
3. Everything *could* be set up in about an hour.
Debugging is a natural process of learning and developing, so while it can be frustrating it was quite welcome. I also like to think that I satisfied each of those requirements by using Obsidian to write, using Github Pages to host, and using Chirpy as a blog site template with embedded Github Actions.

What I aim to do now is really streamline the process of me creating a post in Obsidian, transferring the post and any assets from Obsidian to the local repository, and pushing the new post to Github (and yes, I know it is only like four commands to push, I want to automate).

## Creating a Post
This feels like a comfortable place to start because it is the natural step one in my process: generating the Markdown file in Obsidian. One of the ***super*** fun troubleshooting endeavors I found myself in last time was that Chirpy requires some specific frontmatter to be placed at the beginning of each document, and it required the document to have a specific naming scheme. How much of that frontmatter can be templated, and which ones need to be added manually? All this will really come down to use case. I use the following frontmatter properties:
- title,
- date,
- categories,
- tags,
- media_subpath.

Since `media_subpath` is unlikely to ever change then that is going to be easy to set, and since `categories` and `tags` will be entirely dependent on the post itself, there isn't going to be a way to template those. That leaves us the document name, title property, and date property. Obsidian has a built-in core plugin called Templates, but there is a community plugin called Templater that adds much more functionality for creating templates. In the beginning I tried using the core Template plugin, but I encountered quite a few issues with the way the frontmatter-- which is all YAML-- was handling the built-in variables like `{{date}}` and `{{time}}`, and this ended up forcing my hand to Templater anyways. Regardless of how I wound up with Templater, honestly this is the way to go if you are using Obsidian in general let alone for writing blog posts; the documentation is incredibly thorough, and there is a ton of community support for help as well.

### Let's Make a Template
The easy part finding out how to setup the front matter scheme:
```yaml
<% "---" %>
title: 
date: 
categories:
tags:
media_subpath: /assets
<% "---" %>
```

The brackets here are Templater's built-in syntax that indicate when to execute a code block. There are single line blocks as show above, and there are multi-line blocks that we will see shortly. 

Like I mentioned, `media_subpath` is a static path that is very unlikely to change, so there isn't any need to be be fancy here, but how do we fill out the other two properties and the document name is where it gets interesting. We will start with the date first because the documentation is super clear on how that works, and the Chirpy schema is very specific on how this needs to be.
Templater has a function available that can pull the creation date of the current file: `tp.file.creation_date()`, and while the exact format of the date returned can be changed, the default is `YYYY-MM-DD HH:mm`. This matches exactly to what Chirpy requires, so let's add that to the working template:
```yaml
<% "---" %>
title: 
date: <% tp.file.creation_date() %> -400
categories:
tags:
media_subpath: /assets
<% "---" %>
```

### Convenient Settings
I could condense the last 1.5 hours of research into about 30 seconds of reading, but where is the fun in that? The next thing I really wanted to was the document name. I plan on saving the title property for last since it is derived from the document name. Obsidian has a ton of incredibly useful settings that require user exploration and maybe even some guidance. For instance, there is a setting in the ==Files and Links== section that lets us specify default Note and Attachment locations. Templater also has a couple of useful settings like ==Trigger Templater on new file creation== which lets us choose a template to use anytime we create a new note in Obsidian, and lastly Templater has ==Folder templates== that let us trigger specific templates when a note is created in a specific folder. Armed with this knowledge, we can configure these three settings together to make it to where I can press `CTRL + N` to create a new post and automatically apply the template I am creating. 

### Making a Name
Now for the document name! Templater let's us change the name of a document after it has been created, so for our first iteration we do the obvious:
```yaml
<%*
let filename = `${tp.date.now()}-Temp`;
await tp.file.rename(filename);
-%>
```
We create a temp filename using the Templater's date function, and this will work. We'd have to change the filename to use the actual post title instead of `Temp` though.

> Let's make a special note of that syntax when creating the `filename` variable. The \` creates something akin to a f-string in Python, but the variable/function execution operates more closely to Bash. I did *not* notice that, and I proceeded to struggle immensely when debugging why the title property wasn't updating.

I would normally push the RTFM ideology, but even after looking through the documentation I did not find much mention of this specific syntax. I'm not here to blame them though this was 100% a Problem Exists Between the Keyboard and Chair. Moving on!

### Dynamic and Frontmatter
The documentation provides a beautiful example for me to update the frontmatter properties *after* the document has finished initializing which lets us avoid the guaranteed `undefined` results that will populate `title`. Without deeply investigating, this appears to be because the template-- and thus the frontmatter-- get generated at creation time which is why the `date` shows correctly which happens before the filename is applied which is why we have this lovely hook:
```yaml
<%* 
tp.hooks.on_all_templates_executed(async () => {
  const file = tp.file.find_tfile(tp.file.path(true));
  await tp.app.fileManager.processFrontMatter(file, (frontmatter) => {
    frontmatter["title"] = `${tp.file.title}`;
  });
});
-%>
```

So if I create a new post we will have this frontmatter:
```yaml
title: 2025-11-19-Temp
date: 2025-11-19 18:30 -400
categories:
tags:
media_subpath: /assets
```
That's technically good enough for Chirpy, but it isn't good enough for me. I for one don't want the date representing the post's title since Chirpy uses `title` to create the blog post entry.
So where do go? Since this is really just JavaScript under-the-hood we could parse the `tp.file.title` string to leave `Temp`, and that would also work with something like `tp.file.title.split("-")[3]`. The problem then is when I update the filename, we don't get the re-trigger of the frontmatter without using something like the Dynamic Commands mentioned in the docs, but what my post is going to have spaces in the title? How could I both structure the filename to facilitate parsing and parse it? I could use \_ and replace with " ", but I noticed something in my searching through Google and the docs: `tp.system.prompt`. This will provide an interactive prompt to the user through Obsidian to enter a string, and it supports defaults.
Will I need to undo some of the work done to make use of this, yes, but it will also save me the headache of renaming the document at all; this can be triggered on file creation!

### Refactoring them Together
With this new knowledge we can now refactor the Templater code to be something like this:
```yaml
<%*
let title = await tp.system.prompt("What is the title of this post?", "Temp");
let filename = `${tp.date.now()}-${title.replace(" ", "")}`;
await tp.file.rename(filename);
tp.hooks.on_all_templates_executed(async () => {
  const file = tp.file.find_tfile(tp.file.path(true));
  await tp.app.fileManager.processFrontMatter(file, (frontmatter) => {
    frontmatter["title"] = `${title}`;
  });
});
-%>
```
This way I can create my blog post title using spaces without having to worry about spaces being used in the actual filename. I also don't need to worry about about dynamic renames or manually updating `title` and the filename.

### Altogether Now!
With those lovely settings to automatically trigger the template every time I create a new note, I can show off the lovely template:
```yaml
<% "---" %>
title: 
date:  <% tp.file.creation_date() %> -400
categories:
tags:
media_subpath: /assets
<% "---" %>

<%*
let title = await tp.system.prompt("What is the title of this post?", "Temp");
let filename = `${tp.date.now()}-${title.replace(" ", "")}`;
await tp.file.rename(filename);
tp.hooks.on_all_templates_executed(async () => {
  const file = tp.file.find_tfile(tp.file.path(true));
  await tp.app.fileManager.processFrontMatter(file, (frontmatter) => {
    frontmatter["title"] = `${title}`;
  });
});
-%>
```

Now that the first full step is completed, let's move on to transferring the post to the local repo.
## Transferring the Post
There are probably dozens of different ways to implement and execute the idea, but the idea I had was as follows:

![Workflow Automation Diagram](Workflow_Automation_Diagram.png)

### The Plug(in)
This way I shouldn't need to really leave Obsidian at all for the duration of a post creation (with the obvious exception of debugging or adding stuff like images)! Obsidian has some incredible documentation on designing a plugin, and they even include a sample plugin project template to create a repository from. This is exactly what I did, and now while I have *minimal* experience with TypeScript, Node, and npm, I should be able to piece something together that accomplishes this goal. 
The template Obsidian provides also covers a ton of boilerplate and niceties OOTB, so I don't have to figure out how to create commands, add settings, and the more internal app components. I just need to learn TypeScript! fun..

Honestly, if you are actually interested in making your own plugin, just leave this blog, and go read the docs; if you are just here to watch me struggle, stay.
Cloning, building, and adjusting the `manifest.json` file were frictionless, so I went to work on looking through the dummy code.
It comes with a single setting, so I duplicated it. I named one `scriptName` and the other `path`. This way I wouldn't have to figure out what exactly the working path would be, and I did test that. Sometimes the script would execute from the plugin directory, sometimes not, so in order to eliminate that potential bug altogether I made it user defined. Does this introduce potential security concerns? Not really, if you don't already have the permissions to execute the command or file, then this isn't going to magically give you those permissions.

With two settings now, one for the script name itself and the other for the path to the script, I can go about figuring out how to execute commands and files with TypeScript. Luckily there is an incredibly simple option:
```typescript
import { execFile } from 'child_process';
execFile('/path/to/script');
```

I'm not completely sure why the import syntax is that way, but since that is how every resource I looked at structured I didn't find any point in arguing. Next I need to be able to pass my user defined settings as the parameter for `execFile` and be able to string them together. Back to Google we go! TypeScript users will probably have already noticed back when discussing the weird syntax with Templater, and it really is at this point where I should have realized too, but the \`${}\` is the correct syntax for this when not using `string` types. That wasn't just some quirky Templater syntax; it's TypeScript because the plugins use TypeScript..
Deflated some by that realization, I can now craft the plugin side of the automation:
```typescript
execFile(this.settings.path + this.settings.scriptName);
```

I inserted this in one of the boilerplate commands that came with the template, and I renamed it in a way that suited me. There far more to creating the whole command, and the entire plugin is available in my Github.

### Bash My Head Against the Wall
If I only work on **new** posts, then this isn't that difficult. I can just diff the two directories, awk for the file and assets that only exist in the Obsidian vault, and either rsync or cp them to the Github repo. What if I update or change a blog post? With that thought I mind, I want to consider the Github repo as the truth. This way I can take the SHA256 hash of each file in the `_posts` directory, compare them to the Obsidian vault posts, and copy over everything that is different. Since Git has built-in reset commands, I'm also not worried about clobbering my posts.

Now let us start iterating this Bash script! For some same sane variables, I define the paths to the directories of interest:
```bash
GIT_POSTS="/home/cally/Documents/cally-cmd.github.io/_posts/"
GIT_ASSETS="/home/cally/Documents/cally-cmd.github.io/assets/"

OBS_POST="/home/cally/Documents/Thoughts/blog/"
OBS_ASSETS="/home/cally/Documents/Thoughts/assets/"
```

The assets themselves shouldn't change after creation, and if they do I will just cross that bridge when I get there. With that in mind, I'm going to run a simple `diff` between the two folders, and sync any that are only in `OBS_ASSETS`. That is going to look like:
```bash
diff $GIT_ASSETS $OBS_ASSETS | awk '/Thoughts/ {print $4}'
```

I `awk` on Thoughts because that is the name of the Obsidian vault, and I am fairly certain that it won't change nor will it be part of the Git structure. It isn't hard to imagine that `awk` will produce several matches, so will use a `for`-loop with the result of that command as the iterable. We will see that here shortly, first we need to be able to actually copy to data over.  All in all, we get the following:
```bash
for file in $(diff -q $GIT_ASSETS $OBS_ASSETS | awk '/Thoughts/ {print $4}'); do
	cp $OBS_ASSETS$file $GIT_ASSETS
done
```

Perfect! Next I need to start copying the new post(s) over to the Git repo which will follow a very similar structure:
```bash
for file in $(diff -q $GIT_POSTS $OBS_POSTS | awk '/Thoughts/ {print $4}'); do
  cp $OBS_POSTS$file $GIT_POSTS
done
```

Lastly, we can enumerate the Obsidian posts and compare the SHA256 hashes to the repository as well, if any of them aren't a match, then we can also copy those files over as well. Linux has a built-in command aptly named `sha256sum` which has a flag `-c` that lets us compare a given file with a given SHA256 hash. That is going to look something like:
```bash
echo "$(sha256sum $GIT_POSTS$file | awk '{print $1}') $OBS_POSTS$file" | sha256sum -c | awk '{print $2}' # filename: status
```

In my case, I really only care about that `status`, so I only grab it. Let us put everything together now:
```bash
GIT_POSTS="/home/cally/Documents/cally-cmd.github.io/_posts/"
GIT_ASSETS="/home/cally/Documents/cally-cmd.github.io/assets/"

OBS_POSTS="/home/cally/Documents/Thoughts/blog/"
OBS_ASSETS="/home/cally/Documents/Thoughts/assets/"

for file in $(diff -q $GIT_ASSETS $OBS_ASSETS | awk '/Thoughts/ {print $4}'); do 
  cp $OBS_ASSETS$file $GIT_ASSETS
done

for file in $(diff -q $GIT_POSTS $OBS_POSTS | awk '/Thoughts/ {print $4}'); do
  cp $OBS_POSTS$file $GIT_POSTS
done

for file in $(ls $OBS_POSTS); do
  res=$(echo "$(sha256sum $GIT_POSTS$file | awk '{print $1}') $OBS_POSTS$file" | sha256sum -c | awk '{print $2}')
  if [ $res != "OK" ]; then
    cp $OBS_POSTS$file $GIT_POSTS$file
  fi
done
```

This will technically satisfy my needs by only moving changed files and only moving unique files. Did I find an OOTB command that can do all of this for as I was figuring out how to write this script? Yes, yes I did, and I finished writing the script anyways because it helped me understand the built-in Linux tools better. That command is `rsync`, and when I eventually migrate this blog into my homelab and self-host, I will be abusing that command.

### Git(ting) it Done
I'm not going to implement the Git side to this automation, at least not yet. This post is long enough already, but even with that aside I realized that there is something personal about writing my own Git commit message. 
But overall, this has been an incredibly informative experience! I've been using Linux as my main operating system for a few years now, and I've never gotten into the Bash scripting side of things. It's a quirky language-- not *nearly* as quirky as JavaScript-- for sure, but it's natural integration into the whole operating system ecosystem is convenient and powerful. 

Until next time!
=^.^=