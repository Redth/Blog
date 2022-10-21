# Ghost + Docker + Caffeine == New Blog

History of a blog
-----------------

Some time ago I decided static blog generators (namely octopress) were not for me any longer as I had to always fight with them when I finally went to write a blog post. I stopped writing blog posts since it wasn't _fun_ anymore.

After a bunch of work I had migrated everything over to wordpress which I was intimately familiar with from a former life. Unfortunately I made some concessions in this migration (which I came to regret) such as moving my markdown content into wordpress content - an ugly mix of text, html, and wordpress tags...

Fast forward a couple years and wordpress feels as ugly to work with as ever. Just like the static blog generator, I found myself not enjoying writing blogs.

I finally decided it was time to make my blogging platform get the heck outta my way so I can write more freely, easily, and _enjoyably_...

Ghost is simplicity
-------------------

I have to admit I was considering a move to Ghost before one of my favouritest friends [@jamesmontemagno](http://jamesmontemagno.com) announced he moved his entire blog to Ghost. You see, James and I tend to like the same things, so his move sealed the deal for me. I needed to be on Ghost.

Ghost is strikingly simple. It's just markdown and you, and a few other key features that I could enable in the theme I chose (more on the Caffeine theme later).

This blogging platform feels like the best of a static blog generator and an actual blogging engine. It gets out of your way and lets you write - which is exactly what I needed.

Ghost costs what now?
---------------------

Ghost is aimed at perhaps an arguably more journalist crowd than independent bloggers, and it seems they've priced themselves to avoid the 'riff-raff' bloggers who apparently are more worth than it's worth to service.

Now, ghost is technically open source, and can be self hosted. There's a few 3rd party hosts out there, but none of them seemed to support HTTPS which was an absolute requirement for me.

In any case, the Ghost Pro hosting cost is too spendy for my software engineering know-how. I just can't justify the cost when I have the smarts to do this myself. If the price doesn't bother you, by all means it's the easiest way to get up with Ghost!

O' Azure credits dost thou Docker?
----------------------------------

So I have all these azure credits and I'm not using them for anything currently. This seems like a great use!

Also, I've been playing a bit more with Docker containers lately, and ghost has a [docker image](https://hub.docker.com/_/ghost/) that seems well maintained.

I looked at a number of approaches to running docker containers within Azure. I was hoping for a simple way to run containers without having to maintain the actual host machine/OS.

Azure App Services with Containers looked promising, but doesn't really support volumes, and expects all of the _permanent_ data to be written to `/home/wwwroot/` paths which the ghost docker image didn't support without modification. Also, as I learned later, I would need to _compose_ multiple containers to achieve all the features I wanted, and this would mean multiple App Service instances, driving the cost up significantly (hey, even though it's _free_ I still want to do it as 'cheap' as possible).

I tried playing with Container orchestration such as kubernetes, which seemed good in theory, but a bit overkill for my setup and again, increasingly _expensive_.

Finally I settled on a simple plain old Ubuntu 16.04 LTS virtual machine. The biggest downside is that I'll have to maintain this machine myself. Although there are some promising looking preview features on Azure that might help automatically install at least critical updates.

NGINX + LetsEncrypt = HTTPS
---------------------------

My blog gets pulled into an aggregator site which requires my feed be accessible over https. THere are numerous ways to achieve this without my site itself enabling it such as CloudFlare, however in this modern day, it seems silly not to enable https across the entire site.

Ghost itself doesn't really have a concept of support https, which is a bit of a shame. The official recommendation seems to be using NGINX (although I think this is more common place in other apps now too).

I'm also not crazy about paying for SSL certificates since LetsEncrypt can do this for free.

Luckily it's pretty simple to use both NGINX and LetsEncrypt within Docker.

Ghost 0.11.x vs 1.x
-------------------

There are a lot of tutorials and guides out there on how to install ghost on docker. Most of the documents seem to pertain to `v0.11.x` and **not** the newer `1.x` releases.

Keep that in mind as you research.

Compose yourself, docker
------------------------

So, in my limited knowledge of docker, compose was a new concept for me.

`docker-compose` essentially helps you to orchestrate multiple docker containers working together.

You can have it specify container dependencies, shared volumes, etc.

I still can't say I fully understand it, but after a lot of reading tutorials on ghost `0.11.x` releases and issues/posts about Ghost `1.x` I managed to cobble together a `docker-compose.yml` file which my mother could be proud of (ok, she doesn't even know what docker is, so the bar is set pretty low here).

    version: '2'
    services:  
      proxy:
        image: jwilder/nginx-proxy
        container_name: nginx-proxy
        ports:
          - '80:80'
          - '443:443'
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock:ro #same as before
          - /etc/nginx/vhost.d # to update vhost configuration
          - /usr/share/nginx/html # to write challenge files
          - /opt/ssl:/etc/nginx/certs:ro # update this to change cert location
      ssl-companion:
        image: jrcs/letsencrypt-nginx-proxy-companion
        container_name: ssl-companion
        volumes:
          - /opt/ssl:/etc/nginx/certs:rw # same path as above, now RW
        volumes_from:
          - proxy
        depends_on:
          - proxy
      ghost:
        image: 'ghost:latest'
        expose:
          - '2368'
        volumes:
          - /opt/ghost:/var/lib/ghost/content
        environment:
          - url=https://redth.codes
          - VIRTUAL_HOST=redth.codes
          - LETSENCRYPT_HOST=redth.codes
          - LETSENCRYPT_EMAIL=jondick@gmail.com
        depends_on:
          - proxy
    

Of course there's a few things in there you'd need to tweak for your own website (for instance the environment variables under the `ghost` section, but this should otherwise get you up and running with the latest version of Ghost `1.x`, using the built in sqlite database (which is plenty fine for my needs), and HTTPS support with automatic certificate fetching/renewal via LetsEncrypt, all served up on ports 80 and 443 by NGINX.

Fancy!

After `ssh`'ing into my Ubuntu host virtual machine and installing `docker` and `docker-compose` I was finally able to:

    docker-compose up
    

It's alive!

Run on Caffeine
---------------

The default ghost theme is fine, but I wanted something a bit more custom. Given my affection for the Android world, I settled on the [Caffeine Theme](https://github.com/kelyvin/caffeine-theme/).

I did add some customizations to this, and you can view the diffs from the original theme to [see what I did here](https://github.com/kelyvin/caffeine-theme/compare/master...Redth:master)

Export / Import
---------------

Luckily there's a nice Wordpress plugin for Ghost which simply exports posts into a .json file which Ghost can handle importing. It got things mostly right, except for some extra formatting junk that I had to go into each post and manually remove (while cursing my decision to move to wordpress and abandon my lovely markdown that I was now returning to).

Also, it doesn't handle images. For that I signed up a free account at [Cloudinary](http://cloudinary.com) and used their wordpress plugin to move all my images there, which updated the image links in all my posts so that when they were imported into Ghost, they'd all be correct.

I may decide to eventually migrate the images back to Ghost, but for now I see no reason to.

Hello, world!
-------------

That's it! My new blog is up and running!

Backup Strategy
---------------

Right now I'm having Azure backup my virtual machine's hard drive as a backup strategy. There are a number of other approaches I could take or combine:

*   Write a shell script / cronjob on the Docker host to backup `/opt/ghost/` on a regular basis, which is where all the data lives (this seems anti-docker pattern)
*   Add to my `docker-compose.yml` file a docker image to do the backup
*   Manually export my post and settings data from Ghost (this doesn't backup images, but in theory those should all go on cloudinary).
*   Add to either the cronjob or the docker backup image some sync'ing action to Dropbox or Google Drive

There's a few different ways I find equally interesting to take. I'll hopefully have a future post with some implementation details. For now I'm just doing the Azure + some manual backups.

Bonus marks: Portainer.io
-------------------------

I'll admit, I'm not the biggest fan of cli. I have become increasingly proficient on the command line, but some things I find very helpful to visualize.

I discovered this docker container for [Portainer.io](http://portainer.io) which basically gives you a GUI for your docker host's images/containers. It's quite lovely!