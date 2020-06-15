---
title: 'Deploy your website on IPFS: Why and How'
date: 2020-04-19 00:00:00
tags:
    - ipfs
    - web3
    - ens
category:
    - decentralization
description: I deployed my website on the Web3 using IPFS and ENS. Here I'm detailing why that makes sense and how to do it.
photos: /data/images/Deploy-your-website-on-IPFS-Why-and-How/cover.jpg
---

<sup>Photo by [Clint Adair](https://unsplash.com/@clintadair) on [Unsplash](https://unsplash.com)</sup>

I wanted to learn about IPFS and Web 3.0 so I started exploring it and tried to upload my site there. This is an account of the concepts I learnt along the way and how I finally got my site hosted on [tbking.eth][tbking-eth-link]. I'll give a brief intro about IPFS and why hosting static content there makes sense. If you are already familiar with IPFS, you can skip to the [hosting part][hosting-anchor-link].

## What is IPFS

Inter-Planetary File System is a decentralized network of shared content. It has a very simple yet interesting design philosophy:
> Build a network that works inter planets, and it will be a better network for communication across Earth too.

IPFS is a decentralized network where peers are connected and share files, in many ways like BitTorrent. The fundamental principle is that unlike the traditional web, where files are served based on their location, in IPFS files are served based on their content. Consider this comparison:

Google's privacy policy is identified as a file hosted on Google's servers on the address https://policies.google.com/privacy. The content of the policy doesn't matter. This is called **`location-addressing`**.
On the other hand, IPFS identifies files by their content, using the hash of the file. Let's say you want to read the "_XKCD #327 - Exploits of a Mom_". Its IPFS address is https://ipfs.io/ipfs/QmZVjV5jFV7Jo4Hfj6WPyRnHCxf8kbadkqtQBco2gef64x/. This address doesn't depend on the location of the server but the hash. This is called **`content-addressing`**. Whoever cares about XKCD, can host it. This makes [broken links][broken-links-link] unlikely, which is also one of the goals of IPFS.

[IPFS docs][ipfs-docs-link] explain these concepts well. I recommend them to anyone who wants to dig deeper.

## Hosting on IPFS

Deploying static content such as personal websites on IPFS is easy. The steps I'm listing below can be used for any file such as a plain HTML file, a website generated by static site generators like Jekyll, Hugo, Hexo, and Gatsby or even a media file. So let's dive in.

### Adding files

#### IPFS Desktop
If you have [IPFS Desktop][ipfs-desktop-link] installed and running, you can add your files using a normal file selector. Just import the directory which has the contents of your static website.

![Adding file in IPFS Desktop][ipfs-desktop-image]

Right-click on the newly added content and select `Copy CID`. Open `https://ipfs.io/ipfs/<cid>` to see our files on the web! 🚀

Easy right? 😀

 #### IPFS CLI
[IPFS CLI][ipfs-cli-link] allows adding of files and directories using `add` subcommand.
```
ipfs add -r ./sample_website
```
```
added QmaYZE5QQVR3TqQSL5WDcwuxUwBqczbMkmbHeoeNEAumbe sample_website/index.css
added Qme5TzVW4en8HWP49N9nLHwUkAMhCn6zvWteoknmFDhwJg sample_website/index.html
added QmThzr7LgYmUz6tfTArJK5tbrtidXSA12ZCL4ESy6HB8P2 sample_website/index.js
added QmSYDncaMte6GPx5jC7kqeXWnrKsCp4nkLfQXPxiAKtECU sample_website/something.html
added QmeUG2oZvyx4NzfpP9rruKbmV5UNDmTQ8MoxuhTJGVZVTW sample_website
 1.01 KiB / 1.01 KiB [=================================================] 100.00%
 ```

 The hash printed in the last line is the CID for the whole directory and thus our website. We can see the sample website hosted on https://ipfs.io/ipfs/QmeUG2oZvyx4NzfpP9rruKbmV5UNDmTQ8MoxuhTJGVZVTW/


> TIP: It is important to use relative links in your website since IPFS gateways have a URL which looks like `<gateway>/ipfs/<cid>/file.ext`.

### Pinning
In the last section, we added files to **our** IPFS node for the network to find. This is why the IPFS gateway was able to resolve it and show it in the browser. But the site will most likely become unreachable once you shut down your IPFS daemon. Even though, after requesting some content on IPFS, the receiving node also becomes a host of the content, but the content will be garbage collected after 12 hours. How to serve your site round the clock in a decentralized web without a server?

Welcome, [Pinning][ipfs-pinning-doc-link]!

A node that pins some content on IPFS hosts it forever (until it unpins it). There are pinning services like [Pinata][pinata-link] which pin the files on their IPFS nodes. This way the website is available always.
In Pinata, you can either upload a file or just provide it's hash if the content has already been uploaded to IPFS. Here's how I pinned the sample website we uploaded above.

![Pinata pinning example][pinata-pin-example-image]

> TIP: It is good to pin your site using multiple pinning services for redundancy.

### Automated deployments
As you might have noticed already, using IPFS is very easy. I would go as far as to say it's easier than dealing with the traditional web that we use. However, this process has to be repeated every time you want to change your files which is not very ideal. There are tools like [Fleek][fleek-link] which help in automating all the steps listed above.

Fleek is like Travis or CircleCi for IPFS deployments. You can link your Github account with it and using the Github hooks, Fleek will trigger a deployment on every push to the Github repository. They also pin everything they deploy.

So I use Hexo to generate this blog and I was able to add a build step in the Fleek dashboard itself so that I don't need to generate the HTML and push to my repository. Here's the build command that I use:
```sh
git submodule update --recursive --init && npm i && npm run build
```
Yes, I needed to install the submodules myself. They don't do that by default (yet). Check it out, it's super easy.

### Link to a domain

So now we have our site up and running, but content on IPFS is not as easy to look up as on the traditional web. My site can be found at https://tarunbatra.com. But on IPFS, the _current version_ can be accessed at https://ipfs.io/ipfs/QmTPTa1ddoSkuakaW56SaL9dicbC71BbwfjRbVjasshCXs/.
You see the problem. This is not human readable. And the CID changes after every update. There are 2 solutions to this:

#### DNSLink
With [DNSLink][dnslink-link], you can point a normal domain to an IPFS content. It can be set up on Fleek very easily. I've pointed [ipfs.tarunbatra.com](http://ipfs.tarunbatra.com) to the IPFS version using Fleek and you'll be able to open this site.

IPNS (Inter-Planetary Name Service) also exists and is similar to DNSLink but it's much slower right now.

#### Ethereum Name Service
[ENS][ens-link] is a naming service that lives on Ethereum blockchain and uses smart contracts to buy domains and set resolver records to them. Since Ethereum is involved, you'd need [MetaMask][metamask-link] to use it.

![ENS records for tbking.eth][ens-records-image]

I bought the domain `tbking.eth` and pointed it to the IPFS CID of this site. With changing content, the resolver record of the domain also needs to be updated every time to point at the new CID. Good thing is, Fleek has [recently][fleek-ens-release-link] integrated ENS. You can set Fleek as a controller for the domain and they will update the record automatically on each deploy.

Sweet! This was it. IPFS is perfect as the storage layer of the `Web 3.0`. Try for yourself, and if you liked this blog, share it, and maybe pin it! :)

[tbking-eth-link]: http://tbking.eth.link
[hosting-anchor-link]: #Hosting-on-IPFS
[broken-links-link]: https://en.wikipedia.org/wiki/Link_rot
[ipfs-docs-link]: https://docs-beta.ipfs.io/concepts/
[ipfs-cli-link]: https://dist.ipfs.io/#go-ipfs
[ipfs-desktop-link]: https://github.com/ipfs-shipyard/ipfs-desktop
[ipfs-pinning-doc-link]: https://docs.ipfs.io/guides/concepts/pinning/
[pinata-link]: https://pinata.cloud/
[fleek-link]: https://fleek.co/
[dnslink-link]: https://docs.ipfs.io/guides/concepts/dnslink/
[ipfs-companion-link]: https://github.com/ipfs-shipyard/ipfs-companion
[ens-link]: https://ens.domains/
[metamask-link]: https://metamask.io/
[fleek-ens-release-link]: https://blog.fleek.co/posts/Fleek-Release-ENS-Domains

[ipfs-desktop-image]: /data/images/Deploy-your-website-on-IPFS-Why-and-How/ipfs-desktop-file-add.png
[pinata-pin-example-image]: /data/images/Deploy-your-website-on-IPFS-Why-and-How/pinata-pin-example.png
[pinata-pinned-list-image]: /data/images/Deploy-your-website-on-IPFS-Why-and-How/pinata-pinned-list.png
[ens-records-image]: /data/images/Deploy-your-website-on-IPFS-Why-and-How/ens-records.png