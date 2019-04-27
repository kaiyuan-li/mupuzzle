---
title: Create Personal Blog With Hexo & Raspberry Pi
date: 2019-04-20 22:12:13
tags:
- hexo
- raspberry pi
- domain registration
- port forwarding
- mechanical mode
category:
- software engineering
- web
- blog
---

The first post in this blog discusses about how it was created. From the software technique point of view. This is a simple static website hosted on my raspberry pi 3. The static website is generated with a blog generation tool [hexo](https://hexo.io). I have also registered the domain name mupuzzle.com with [google domains](https://domains.google). In order for the domain to find my raspberry pi with dynamic DNS, since ISP keeps shuffling IP address, I used the DDclient tool by following this [tutorial](https://support.google.com/domains/answer/6147083?hl=en) from google domains.

Creating a personal blog is an exciting task as it opened a window for you to exchange ideas with the open world, as well as contributing to the open source community. Though there are multiple mature ways of creating one such as WIX, world press etc, they all cost good amount of money and offer less flexibility to yoru website. I am writing down the key steps of my blog creation process. Hopefully by following these key steps you can as well create your own blog. FYI, though I am a fulltime software engineer, I never had any degree of computer science. So I think anyone should be able to reproduce my work.

## Hexo
[Hexo](https://hexo.io) is a tool to generate static blog. The blog is written in markdown and compiled/translated by hexo to static htmls. The static htmls can then be deployed easily on to git page, heroku, etc. More details can be found online. But in my project, I chose to create my own server on [raspberry pi](https://www.raspberrypi.org).

Hexo is available on npm, so the first task is to install latest npm on raspberry pi. This cannot be simply done with `sudo apt install npm` on raspberry pi, since the installed version is very low. To install the latest version of nodejs and npm, I found an online recipe with:
```
curl -L https://git.io/n-install | bash
```

This will install nodejs and npm in `~/n/bin`. So two bash alias is better added for convenience:
```
echo "alias node='/home/pi/n/bin/node'" >> ~/.bashrc
echo "alias npm='node /home/pi/n/bin/npm'" >> ~/.bashrc
```

Restart terminal and `node -v` and `npm -v` should be able to return latest versions. At this point, we can install hexo:
```
npm install hexo -g
echo "alias hexo='node /home/pi/n/bin/hexo'" >> ~/.bashrc
```
Restart terminal again to make the `hexo` command effective. Then create your fancy hexo project simply with
```
hexo init myblog
```
As previously mentioned, hexo is simply a blog generator so we got to generate and serve it locally as:
```
cd myblog
hexo g
hexo s
```
Tada, now you can go to your browser and enter the URL `localhost:4000` to see the new shiny hello world hexo blog~
For more details about how to write your blogs in markdown, please spend a few minutes on hexo's webpage. I am not diving into the hexo details here.

## Google Domains
Now since I want to publish my website to the internet so anyone can visit my blog from anywhere (the localhost:4000 definitely does not work), I am going to register my domain from google domains. It is as cheap as 12 USD/year and you got couple of benefits from Google. I always believe in big tech companies since they have mature resolutions for all kinds of user scenarios.

Now that I have got my domain name `mupuzzle.com`, I would like to have it connected to my raspberry pi. That is to say, whenever user enters `www.mupuzzle.com` they are actually sending a GET request to the 80 port of my raspberry pi. How can we do that? There are two problems to solve. First, I need to let Google domains know the IP address of my raspberry pi. Second, since my raspberry pi is connected to my home wireless router, I need to tell my router that everytime a request comes to its port 80, forward it to my raspberry pi's port 80. Technically these two problems are called Dynamic DNS and port forwarding, respectively.

### Dynamic DNS
The Dynamic DNS issue can be well resolved by following the recipe on Google domains' [official page](https://support.google.com/domains/answer/6147083?hl=en). In short, install DDclient on the raspberry pi (feel free to leave every configuration blank during installation):
```
sudo apt install ddclient
```
then config `/etc/ddclient.conf`. My example configuration is:
```
ssl=yes
protocol=dyndns2
use=web
server=domains.google.com
login=ba5OOjCmacW9PwgC
password='sOA6yFOzQulB7hXQ'
www.mupuzzle.com
daemon=30
```
The login and password hashes can be found from Google domains's page when you specify dynamic DNS there (Note: do not try mine since I have already changed some letters inside). Pay attention to the single quotes for the password field as it was not explicitly mentioned by the Google domain recipe. After the file is written, try it with
```
sudo ddclient -daemon=0 -debug -verbose -noquiet
```
A success message should be seen at the end of log. Then refresh the google domains page and the IP of raspberry pi should be showing on the dynamic DNS part.

### Port Forwarding
Port forwarding is relatively simple. The only tricky part is to set a fixed local ip address of raspberry pi since router usually do DHCP, which means the raspberry pi might get 192.168.1.4 this time and 192.168.1.5 next time. Then the router will not be able to know which one it should forward message to. I have done the static IP work long time ago so I do not quite remember exactly the steps to do that. But there are plenty of online tutorials about doing so.

About port forwarding, just go to your router's page, usually at 192.168.1.1 and configure the advanced settings. Find the port forwarding page and select forwarding HTTP messages for port 80 to port 80 of the raspberry pi.

## Start Server
Now that everything is at it's place. Start serving files with
```
hexo s -p 8080
```
Why am I setting the port number to 8080? I think ideally it should be 80. But 80 gives me error so I set 8080...

### Screen
Usually, raspberry pi is controlled by an ssh connection and we launch our server in that ssh terminal. Once it's up, we would like to close the terminal but then the process is killed, i.e. server is no longer running. There are multiple ways to resolve this issue and one of them is screen. The workflow is:
```
screen
hexo s -p 8080
^A + D
```
the command `^A + D` means ctrl+A then press D. Then terminal can be closed and the process is still running. Next time logging into raspberry pi (maybe via ssh), just do `screen -r` and the running server process will be showing up there!

## Conclusion
In this blog, I presented how my personal blogging system is built. Hopefully you can create your own with the same recipe. For any question, feel free to email me at why@mupuzzle.com. Good luck!
