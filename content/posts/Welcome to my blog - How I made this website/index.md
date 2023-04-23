---
title: "Welcome to my blog - How I made this website"
date: 2023-04-22T7:30:04-07:00
---

This is the first blog entry of this site. It contains the process I went through to set up the site and the things I learned along the way.

## An Idea

The idea for this blog came from seeing static site generator frameworks and how some of them can create html from markdown documents. I use Obsidian for my notes which uses markdown for its note formatting. I wanted to see if I could create a blog where I would write the entries in Obsidian and then deploy them to the website.

The initial workflow I drafted up for this project was as follows:

1. Find something I want to share
2. Write out a blog post in Obsidian
3. Push the new markdown file to a git repository
4. Have a Github action run to deploy the site with the new page

## Choosing a Framework

After deciding the workflow I wanted, it was time to select which SSG framework I would use. I have some experience with Next.js but I remembered seeing a Fireship.io video talking about some other SSG frameworks. The one I specifically remembered was one on Hugo which seemed interesting at the time. I decided to go and find the video and watch it again to see how Hugo worked. Based on what I saw in the video, Hugo seemed interesting for the following reasons:

1. It's fast
2. Content is created by writing markdown (a requirement)
3. Uses templates to create pages from the content
4. Ability to add prebuilt themes or create my own theme easily

Overall I liked how the templates were made and it drew me in wanting to learn more about it.

## Setting up Hugo

After watching the Fireship.io video, I went to the official Hugo website and clicked the "Quick Start" button.  This took me to their quick start guide which would help me get started making my new blog. The first thing they had me do was install Hugo on my system. I had originally thought that Hugo would be another Node library like Next.js but Hugo appears to be a standalone application. After finding this out, I wondered if I would have any issues due to running an Arch based distro. Would there be an entry in the official repositories, would there be an AUR package, or would I have to compile it myself? Thankfully, there is a binary in the official repos I could use which I installed it with no issues. With Hugo installed, I could create the project and get started with actually creating the website.

The first thing the quick start guide had me do was run the following commands.
```
hugo new site quickstart
cd quickstart
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke
echo "theme = 'ananke'" >> config.toml
hugo server
```
This created a Hugo starter project, initialized a git repository, and added the "anake" theme to get me started. I am not familiar with using git submodules but they seem to be a way to manage nested git repositories which seems very useful. After running `hugo server` and opening the site in Firefox I was presented with the following:

![Initial Site Screenshot](img/Initial%20Site%20Screenshot.png)

Not much to look at yet but I was a fan of the minimalist black and white theme.

## My First Blog Post

Now I had a site but I needed to add some content. The next section in the guide had me create a new page for the site. To create a new page they had me run `hugo new posts/my-first-post.md`. This created a file in `/content/posts/` and added the following into the new file:
```
---
title: "My First Post"
date: 2023-03-21T14:24:59-07:00
draft: true
---
```
They call this section "Front Matter" which can be formatted in yaml, toml, or json. toml is identified by using `+++`, yaml with `---` and json with `{}`. In the Front Matter section you can specify metadata for the post which can help with some of Hugo's other features that will be touched on later. There are many predefined variables which will cover most use cases but users can add their own variables if they want. Additionally the tags can be passed down to children if they are defined in the `cascade` tag. Front Matter looks similar to the DataView plugin for Obsidian in terms of storing data at the beginning of the file in YAML. I do not use DataView but I can see myself using some of the provided tags for Front Matter. More information on Front Matter can be found [here](https://gohugo.io/content-management/front-matter/)

One of the tags that was set in the Front Matter of the new post was `draft`. This tag specifies that the document is a draft and that Hugo will not publish it. I can see myself using this to make sure I don't accidentally post an incomplete post. This is useful for when I have deployed the site but how do I preview the drafts? To build posts including drafts run `hugo server --buildDrafts` or `hugo server -D` This will start the Hugo server and will compile the drafts so I can view a preview of my work. After running the server in draft mode we can see the new post appear on the home page displaying the Front Matter title and the first couple lines of the post.

![Draft Post Home Page](img/Draft%20Post%20Home%20Page.png)

After explaining Front Matter and how to preview drafts the guide explained how to configure the site. To do this we use the config.toml file. The file contained the following:

``` toml
baseURL = 'http://example.org/'
languageCode = 'en-us'
title = 'My New Hugo Site'
theme = 'ananke'
```

The `baseURL` should be set to the baseURL of the site, in my case `https://blog.johnbrehm.dev/`. It must contain the protocol and end with a slash. The other fields are fairly self-explanatory except for the theme. This is the theme that the guide had me set earlier in the guide. That would normally not be generated by default. With a post written and the site configured it was time to learn how to publish it.

The next part of the guide explained how to publish the site. Publishing involves having Hugo create the site in the `public` directory in the root of the project which includes the HTML and asset files. To publish the site was the simplest step so far, all I had to do was run `Hugo`. As mentioned before this will ignore any draft posts and only include the posts that are ready to be published. With the site published it was time to deploy it.

## Deployment

After explaining the arduous task of publishing the website, the guide left a link to the website's section on Hosting & Deployment. Looking through the provided articles I clicked on the "Host on GitHub" section. Since my plan was to deploy to Github using Github actions and Github Pages,  this page looked like it had the steps that I was looking for.

The guide started off following the steps I expected

1. Create a Github repo
2. Push code to the new repo
3. On Github go to Settings>Pages
4. Change the Source to Github Actions
5. Create `.github/workflows/hugo.yaml` in the local repo

In the new file I needed to add the Hugo deployment script. The guide provided a starter script which needed to be modified to have the correct branch and Hugo version. In my case I wanted the script to run on main which was the default but I needed to change my version number.

``` yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.110.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb          
      - name: Install Dart Sass Embedded
        run: sudo snap install dart-sass-embedded
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"          
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

By following these steps and using this configuration I was able to get the site up and running. However, I wanted it deployed to a different URL. With this configuration the site was deployed to https://johnbrehm.dev/hugo-blog/ I wanted it to be on https://blog.johnbrehm.dev/. This was a simple fix which I had figured out in the past. All I needed to do was change the Custom domain to blog.johnbrehm.dev in the Pages settings, add a CNAME file to the repository with the new custom URL, and add the CNAME DNS record to my domain's DNS settings. Once that was done all I needed to do was wait for the new DNS record to propagate and for Github's certificate to update for HTTPS to work.

## My First Real Blog Post

Now I had a website deployed but I still needed to add the content to it. The first thing I tried was to add the blog's repository into Obsidian but I didn't like the way that all of the repositories' files were visible in the file view. I also realized that I should probably use the Hugo command line commands to create new posts like I had done earlier. I wasn't sure what other configuration Hugo did when running the command so I checked to see what files were modified after running the command. Looking at the git diff all it did was create a new file and add the default Front Matter.

After deciding I didn't want to put the repository in my Obsidian vault I decided I needed to slightly alter my original workflow. Since all `hugo new` does is create a new file and add the Front Matter I could create an Obsidian template to do the same thing. Knowing this I created the following template.

```
---
title: "{{title}}"
date: {{date:YYYY-MM-DDTHH:mm:ssZ}}
draft: true
---
```

This creates a file with the same content as `hugo new`.

By using the template I could create new posts from Obsidian as if I were running `hugo new`. This would create the new post in my Obsidian vault and then I would copy it (write a script to copy it) to the posts folder in the repository.

The next issue I ran into was how I should handle the images. It wan't obvious to me how you are  supposed to add images with Hugo. Where are they stored and how are they included in the markdown? In Obsidian images are included by using custom markdown with `![[Link to image]]`. This creates a link in Obsidian which is very useful when taking advantage of some of the program's features. But this is a post about Hugo so let's focus on what I needed to do to get images to work.

In order to learn more about how to include images in Hugo I searched their docs for how to handle images. This search brought me to [this](https://gohugo.io/content-management/image-processing/#image-resources) page which details how to add images to your content. In Hugo you can reference an asset as a page resource or a global resource. The images I want to include are only relevant to the current blog post so I read more about the page resources. This led me to look at page bundles. The main resource on Page Bundles can be found [here](https://gohugo.io/content-management/page-bundles/). 

Page Bundles can either be Leaf Bundles or Branch Bundles. Simply put, Leaf Bundles are pages with no children and Branch Bundles are pages with children. This post has no children so it is a Leaf Bundle. To make a Leaf Bundle, the post needs to be put into a folder within the `content/` directory and have an `index.md` file. The images that will be included can be added to this directory. 

Since the md file for the blog post needed to be renamed to index.md I needed to change the Obsidian template that I created earlier to accommodate this change. <u>This is the updated template:</u>
```
---
title: ""
date: {{date:YYYY-MM-DDTHH:mm:ssZ}}
draft: true
---
```
The only change is the removal of `{{title}}`. Since the title of the file needs to be index.md I do not want to have Obsidian fill out the title field for me. Since I am naming the containing folder the same as the title of the blog post title, there might be a way to have Obsidian set the title for me but i decided that I would leave that for later if I find adding it manually too annoying.

With the images included in the bundle I went back to the first page to see what markdown I needed to include to reference the images. The page includes some examples on how to include the images but they use HTML. The examples can be seen [here](https://gohugo.io/content-management/image-processing/#image-rendering). The example, which I liked, was the third example shown here:
```go-html-template
{{ with .Resources.GetMatch "sunset.jpg" }}
  <img src="{{ .RelPermalink }}" width="{{ .Width }}" height="{{ .Height }}">
{{ end }}
```
Since this was HTML it would not work for me. Although the HTML would have probably worked there was a different issue, the parts in the curly braces were not working. This snippet should find a resource with the name "sunset.jpg" and set it to `.RelPermalink` but it wasn't. Instead I was seeing the curly braces and the content in them as plain text.

After reading through more of the docs I found that the code in the curly braces is referred to as a Function. These functions are go-template functions which Hugo uses to generate the HTML from the markdown. The Functions that can be used include the standard ones that come with go-templates as well as additional functions provided by Hugo. The full list of Hugo functions can be found [here](https://gohugo.io/functions/).

Functions seemed very helpful except for the fact that they were not working... After searching a bit online for why they were not working I realized that I was overthinking the whole thing and that I could just treat it a a normal markdown image. This worked with one slight hiccup. I had spaces in the filename which was initially making the image not work. To solve this I had to replace the spaces with %20 since the markdown is expecting a URL. The markdown for one of the images came out like this:

```
![Hugo home page](img/Draft%20Post%20Home%20Page.png)
```

After finishing with that wild goose chase I decided that the blog was good enough and I should post this entry.

## Conclusion

Overall the experience of writing this post was fine. It was fairly easy to write and it was easy to get Hugo working but there were some slight issues which slowed down the process of creating the blog. Most of these issue were not enough to stop me, however, and I still got the post done. There are still more thing I want to explore within Hugo like Taxonomies but I will leave that for another time. The workflow for writing posts isn't exactly as I thought it would be but I think it is close enough to what I wanted which is a satisfying conclusion to this story.