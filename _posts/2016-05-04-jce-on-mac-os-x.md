---
layout: single
title: JCE on Mac OS X
category: [howto]
tags: [macos, java, jce, brewcask]
---
I had to install JCE with unlimited strength policy on my MacBook. Usual way:
go to Oracle pages and download JCE manually. Luckily I'm using [brew
cask](https://caskroom.github.io/).

{% highlight bash %}
$ brew cask install jce-unlimited-strength-policy
==> Caveats
Installing this Cask means you have AGREED to the Oracle Binary Code License Agreement for Java SE at
  http://www.oracle.com/technetwork/java/javase/terms/license/index.html

==> Downloading http://download.oracle.com/otn-pub/java/jce/8/jce_policy-8.zip
######################################################################## 100.0%
==> Verifying checksum for Cask jce-unlimited-strength-policy
Password:
🍺  jce-unlimited-strength-policy staged at '/opt/homebrew-cask/Caskroom/jce-unlimited-strength-policy/1.8' (3 files, 16K)
{% endhighlight %}

And quick test it's installed

{% highlight scala %}
scala> javax.crypto.Cipher.getMaxAllowedKeyLength("AES")
res0: Int = 2147483647
{% endhighlight %}

If JCE unlimited strength policy is not installed, then returned value is `128`.
