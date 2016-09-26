+++
date = "2016-09-25T23:00:00"
draft = false
tags = ["passwords"]
title = "Memorizing strong passwords"
math = false
+++
Though this is not really a too groundbreaking article, I noticed that quite a lot of my friends didn't know the trick, so I thought I could share it.

## Intro

Nowadays there are some requirements to a good password, most people know them

* **Entropy** - Mixed case, special characters, no natural language words... I personally think this is not too important and makes passwords [harder to remember](https://xkcd.com/936/) while not increasing strength too much. 
* **Length** [Size matters](http://imgur.com/gallery/zFyBtyA)! With every additional letter you theoretically multiply the effort needed to crack your password by 256 (practically maybe factor 60).
* **Uniqueness** From time to time, companies screw up [badly](http://zed0.co.uk/crossword/), so you should use a different password on each site and a password other users are unlikely to choose aswell. No "qwerty" is not innovative, you should go for [Mb2.r5oHf-0t](http://www.der-postillon.com/2014/04/it-experten-kuren-mb2r5ohf-0t-zum.html) ;-).
* **Lifetime** You should change your password from time to time, as all passwords can be cracked with just enough patience

Yes, nice dude, who can remember those? I mean I just went to the basement and forgot on the way there why I went there, how should I remember like 20 passwords, each 16 characters long? One way, maybe the most secure one, to achieve these goals is [randomly generate](https://www.random.org/passwords/) passwords and store them in a [password manager](http://keepass.info/). However, there are few easy-to-use implementations out there and it gets difficult if you need your passwords at a place where you can't access the keyfile (like if you try to log into your mail from an internet-cafe on holidays (which you should try to avoid for so many reasons, only one of them being the better weather outside (I like nested brackets))).

## Idea

However, there is a simple trick to have a different, strong password for each site/account you hold, yet still just having to remember one. So, what about this:
```
<mypassword>-google
```
You generate the `mypassword`-fraction yourself and memorize it, then you append the name of the website wherever you use it. This has some nice advantages


 1. You will have a **different password** for each site. Simple password crack bots won't recognize the similarity and you have a fairly low chance to get your password guessed by a bruteforce attack.
 2. Your password will be even longer through the additional website name (remember, size matters!)
 3. You just need to **remember one** password! Even if you struggle with your mom's birthday, I believe in you!

*Please, as this is the only password you have to remember, choose something more sophisticated than "asdfgh"!* Doing that would also make it easier to use the password on sites that require your password to fulfill ceratin criterias, like "at least one uppercase letter" or "at least 8 letters". It is a pain when there is one single site using a password policy which doesn't match and you have to think of something new (and forget that seconds after (I could nest another pair of brackets here)).

## Critics

There are still two drawbacks to this. 

* If you just append the name, a slighly more sohpisticated passwort-bot could still guess the password and crack it. Especially if more people are starting to do this, it will eventually end up being a standard-method to crack passwords. To overcome this, you could obfuscate the website name a little. Be creative!
```
<mypassword>-elgoog
go<mypassword>le
<my>go<pass>og<word>le
```

* If a human sees some of your passwords, it might figure out the algorithm you build them with and instantly crack all your passwords. This approach does not protect against that. Though it is unlikely someone will manually read through all the passwords in a password leak, you could still be disclosed when typing it. I personally decided to take the risk and have the comfort of not using a password manager, but you may decide differently. In case of a disclosure I will have to change a lot of passwords in contrast to one single one with the password manager.
* And finally, if you still, after being warned this whole article long, choose asd-google, you will not gain much.

But all in all I think of this as a nice way to memorize passwords, and you could think of switching your password policy to this. Also try to use two-factor authentication if the website offers that.