---
title: How this blog is being hosted
description: Details on how this blog is being hosted
slug: how-this-blog-is-being-hosted
date: 2024-11-20T00:00:00+0000
lastmod: "{{ .Lastmod  }}"
image: cover.jpg
categories:
    - DevOps
    - Hosting
tags:
    - hugo
    - wireguard
    - github-action
    - ci
---

Hi there, to who whoever came upon this website. Welcome to my first blog post!
\
\
I have always been fascinate by the idea of self hosting useful things for me and my friends \
whether it'd web apps, game servers, and etc. So why not create a blog, and share what I'm doing in my free time?
It'd also be beneficial for me as I can refer back to this blog in case I
need to be reminded of something.
\
\
There are many ways to host a website nowadays, some are incredibly easy and just a few clicks, and you're good to go.
Being me, loving a challenge, did the opposite, and chose one of the hardest way possible (imo).
In this post, I'll share what technologies this blog is being hosted on. Why I chose the current solution,
how it's configured, and what I've learned along the way.
Whatever reason you're here for, whether you're just curious, or you want to host your own, I hope you got what you were looking for!


## The Blog Framework
The blog itself is built using Hugo framework. It's just a static website generator. I'm writing this post using
normal markdown, as you would write a GitHub readme file for example. \
As for the theme, I'm currently using [Stack theme](https://stack.jimmycai.com/) by Jimmy Cai (Thanks for a cool theme Jimmy!).


## Hosting Challenges
Building a website is easy enough, especially a static one like this. The real challenge is on publishing the website so that
it's accessible from outside of the local network. \
\
There are many ways to do that, but as I already mentioned that I like to host things by myself, I want to host it in my local network.
Moreover, I already have a spare computer that's not being used. So, I decided to give that computer its time to shine.
Just hosting it in the computer wouldn't expose its content to the outside world. My ISP doesn't provide static public IPv4 either,
and not even a dynamic one. The solution is that I need to get public IPv4 from somewhere. \
\
Luckily, there are plenty of cheap VPS I can rent, all of which probably provide a public IPv4. But how can I host using my local network
while using the IP address of my VPS. That's where WireGuard came in!


## Solution: WireGuard and VPS
WireGuard is a VPN tunnel protocol. It's faster and newer and more secure than a more known protocol like OpenVPN.
There are plenty of guides on the internet on how to setup WireGuard, so I'll just skip that part.
Normally WireGuard is just a normal VPN protocol. However, we can configure it using `iptables` to forward the IP to my local machine. \
Here’s an example of the `iptables` command I used in my WireGuard configuration:
```
iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 80 -j DNAT --to-destination 10.0.0.4:80
```
This command tells the VPS to forward any incoming TCP traffic on port 80 to the local machine’s WireGuard peer IP (in this case, 10.0.0.4) on port 80.


## Automate Deployment 
Now that the website is accessible from the outside world. Adding an automate deployment make it easier to maintain, and reduce amount of headache along the line.
To add automate deployment, I use `rsync` to and `Tailscale`. Here is how it works.
1. Use Tailscale to allow GitHub Actions job access my local machine.
```
    - name: Tailscale
        uses: tailscale/github-action@v2
        with:
        oauth-client-id: ${{ secrets.TS_OAUTH_CLIENT_ID }}
        oauth-secret: ${{ secrets.TS_OAUTH_SECRET }}
        tags: tag:ci
```
2. Then I can simply use `rsync` to deploy my website like so.
```
    - name: Deploy
        uses: contention/rsync-deployments@v2.0.0
        with:
        FLAGS: -avzh --progress --delete 
        EXCLUDES: ""
        USER: fordkuppp
        HOST: 192.168.1.102
        LOCALPATH: public/
        REMOTEPATH: /home/fordkuppp/www
        DEPLOY_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```


## Wrapping up
And that's how this blog is being hosted in a nut shell. Built using Hugo, hosted on my spare computer, publish by using WireGuard and VPS. And made
easy to update by utilizing GitHub Actions