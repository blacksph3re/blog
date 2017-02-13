+++
date = "2016-10-30T19:00:00"
draft = true
tags = ["own cloud", "bananapi", "nextcloud"]
title = "Personal cloud on 3 bananapies"
math = false
+++
My project this halloween was to set up a small personal cloud with simple redundancy, as cheap as possible. I did this together with a friend of mine, as buying 3 bananapies didn't turn out THAT cheap.

# Hardware


# DNS
As apparently DynDNS has stopped offering their free service, we decided to implement something our own. I already owned the webspace you see this post on, so why not having some kind of dyndns from here? In the end I wrote a small nodejs server which redirects to an ip , and on the cloud we put a small python script that determines the current public IP and sets that in the node backend. If you call the node server from a webbrowser, you will automatically be redirected to the current IP of the cloud. So we don't use a DNS record right now, but therefore had the fun of implementing our own dynamic forwarding.

There is not really much to be explained with these scripts. The python