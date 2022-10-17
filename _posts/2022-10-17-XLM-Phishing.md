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


Even the domain's DNS records look nothing like the original Stellar's site, which is protected by Cloudflare:

![_config.yml]({{ site.baseurl }}/images/xlm/dns.png)

This is a pretty good indication that the two domains are not controlled by the same entity.

The page offers to create a wallet for sending XLM. After reading the modified Javascript code, we found out what it was doing:

> Wait until the wallet reaches 15k XLM
> Create a new private/public key pair using stellar sdk
> Trigger a wire transfer to the newly created wallet
> Save a transaction's history containing the **wallet's private key**

By sending the private key to **/api/gdata**, the rogue site ensures they keep their hands on all the accounts where the XLM has been transfered.

The modified sources can be found here:

![_config.yml]({{ site.baseurl }}/images/xlm/sources.png)

The algorithm generates a JSON payload of the following form:

```json
{"ffatale":"BLOBBLOB 97023.3555 => SCT62N4GYPBBTJCMGSHGEUTJNCXXCULPSYFFYAS6D5H62HD7SQF6RFZS"}
```

Then, the javascript source encrypts the json array using AES and a harcoded passphrase:



The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
