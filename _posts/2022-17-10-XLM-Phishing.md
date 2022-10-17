---
layout: post
title: XLM Account Viewer phishing
---

The following address: **hxxp://accountviewer.stellar.org.ge** is a malicious clone attacking stellar users.

It all starts with a phishing email:

![_config.yml]({{ site.baseurl }}/images/xlm/phshing.jpg)

Quickly checking the mail headers reveals suspicious details:

![_config.yml]({{ site.baseurl }}/images/xlm/headers.png)

The send is **no-reply@annfinance.co.uk**, which seems to be a legitimate company with records dating from at least 2016. This company however does not seems to have any link with Stellar whatsoever. It is plausible that their mail accounts have been compromised in order to perform phishing attacks.

The mail's body contains a link to Stellar's account viewer page, which looks exactly like the original, as it was probably forked using [AccountViewer v2](https://github.com/stellar/account-viewer-v2), except that the domain is stellar.org.ge instead of stellar.org:

| ![_config.yml]({{ site.baseurl }}/images/xlm/clone.png) | 
|:--:| 
| *Clone Website* |

| ![_config.yml]({{ site.baseurl }}/images/xlm/legit.png) | 
|:--:| 
| *Original Website* |


The TLS certificate is also not issued by the same CA, which is usually very fishy, especially if it comes from Let's Encrypt, as they are free, and easy to deploy:

| ![_config.yml]({{ site.baseurl }}/images/xlm/clonecert.png) | 
|:--:| 
| *Let's Encrypt clone certificate* |


| ![_config.yml]({{ site.baseurl }}/images/xlm/legitcert.png) | 
|:--:| 
| *Original Sectigo certificate* |


Even the domain's DNS records look nothing like the original Stellar's site:

![_config.yml]({{ site.baseurl }}/images/xlm/dns.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
