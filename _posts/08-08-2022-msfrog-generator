---
layout: post
title: "Writeup CORCTF: MSFrogGenerator"
categories: Web
tags: Hacking Web
---
This was the second most solved challenge in 2022 [CorCTF](https://2022.cor.team/) competition for the Web category.  We have a funny website in which we have to customize our frog with some emoticons.
<!--excerpt-->
![](/img/msfroggenerator/1-website.png)

We can move the emoticons wherever we want, with only three limitations; in order to generate the picture we have to:

- insert at least one emoticon;

- insert at most three emoticons;

- insert the same emoticon no more than one time

Exploring what the website does when opened, we see that its behavior is based on a javascript file:

![](/img/msfroggenerator/2-jsfile.png)

Everyone would have at least checked the file, finding inside it an enormous wall of text: luckily analizing the website source code we find a note:

```html
<!-- NOTE: There is no (intended) vuln in the frontend, 
please don't waste your time digging into the JS ;) 
--> 
```

So I immediately skipped it.

By generating the image, instead, we would have the opportunity to save it in local: but the most important thing is the call that the website does when the button is clicked:

![](/img/msfroggenerator/3-generate.png)

It calls another page: /api/generate; it contains a JSON with a base 64 that once rendered will be the frog we've created: nothing useful here. 
At this point I tried to play with POST requests, using curl first:

![](/img/msfroggenerator/4-curl.jpg)

we can see that changing the input actually changes also the output. It was easier, to me, continue on Burp Suite. I intercepted the request to api/generate and modified it in order to send a command, and watch its output:

![](/img/msfroggenerator/5-whoami.png)

It actually worked; plus, it returned an error message if i didnt put a correct name for the image, that's why I've left mskiss.png.

I can also see it is searching in the img folder: I tried to list the files, but as expected there was no flag. At this point I've searched into the root folder /:

![](/img/msfroggenerator/6-ls.png)

and here it is: the file /flag.txt is the one we're searching: with a simple 

```bash
$(cat /flag.txt)/mskiss.png
```

we obtain our flag:

![](/img/msfroggenerator/7-flag.png)

*Flag: corctf{sh0uld_h4ve_r3nder3d_cl13nt_s1de_:msfrog:}*  
