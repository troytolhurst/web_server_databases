# mkcert

Learn to explain how Internet connections are secured.

[Video Alternative](https://www.youtube.com/watch?v=Ivym1ZaBxfI&t=766s)

## The question: How are Internet connections secured?

Here's an interesting problem — how do you send me a secret message?

You and I might never meet in person, so if you want to send me a message it'll
probably be over the Internet. Remember `traceroute` and how packets go across a
network of many different routers? Do you trust all of those computers?

Perhaps you do. But imagine if it was so secret that you needed to be absolutely
certain that no one in the middle could ever find out. How would you do it?

Here's one way: you and I could agree on a secret password. We could use that
password to encrypt our data. OK that sounds good. Let's try it out. Here's my
data:

```
U2FsdGVkX1/FA+B7o7Ax6R0nsPHBRqPfFoB8McMewLvMspB7d0QfgUaiuYo2Soci
qVYlMOgFbsugjCt38bYCIUsIAT3D1CjVWo88UXJ76v4pdyNUp7N0Qd4Zb/s3jZvH
QjFUsVHGVqCui5IwbLRlnUPaf4CKSnVip6lQaPlGpfE=
```

Save it to a file `secret.txt.enc` and you can decrypt it using this command:

```
; openssl aes-256-cbc -base64 -d -in secret.txt.enc
```

You just need the password... oh wait, no, how am I going to send you the
password? The only way I can do that is by using the Internet. I suppose I could
try to hint at it? It's the name of the music artist I'm listening to at the
moment, lowercase...

But no, this clearly won't work. Not if we need to set up an Internet where
billions of people need to send secret information between each other daily. 

This problem is called the key distribution problem. To communicate securely
over distance you need a secret key, and sending that secret key in an insecure
way doesn't help because someone in the middle could steal the key and use it to
decrypt all your messages.

This problem went unsolved until 1976. It was solved by something called
Public-Key Cryptography. A team of mathematicians worked out how you could
generate a pair of keys, one public and one private. The public key can be used
to encrypt messages, but only the private key can be used to decrypt them.

Here's how we could use this to allow you to send me a secret message:

1. I'll generate a public and private key.
2. I'll publish my public key right here.
3. You use it to encrypt your secret message
4. Send your encrypted message to me on Slack (I'm Kay Lack)
5. I'll decrypt it using my private key and respond.

The public key allows you to encrypt messages, but only the _private_ key can
decrypt them. So no one in the middle will be able to decrypt it.

Here's my public key:

```
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDvK9hhFVu/FDtjcFhSIDHs25Me
JLk5Ovrj3tMaCWLWdBJNoPoFnn+Vzs102c6pvZjZkNUsoRjZsg6+kEeFAqmhvucM
7YPvm1Es8YpTw9T0sa2Kp6PBL6hy7g6ZB9h98XpqKVU9jGmzTvSG50Wx1hVQ+NjN
dhgAV2QsOD0cjyKlUwIDAQAB
-----END PUBLIC KEY-----
```

Save that to a file `kay-public.pem` and then run this:

```shell
; cat - | openssl rsautl -encrypt -inkey kay-public.pem -pubin | base64
# Type in a message and then hit ctrl+d to finish
```

You'll see a big string of letters and numbers. Send over the whole thing and
I'll get back to you.

<!-- OMITTED -->

Let's see how this technique is used to secure web connections.

## The tool: `mkcert`

We're going to set up a local web server and create a secure connection with it.

First, create a new Flask app. Here's a quickstart:

```shell
; mkdir secure_flask
; cd secure_flask
; pipenv install --python 3.11 flask
; pipenv shell
; touch app.py
```

```python
# File: app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return 'Hello, world!'

if __name__ == '__main__':
    app.run(port=4321)
```

```shell
; python app.py
# Server starts
; open http://localhost:4321/
```

Note that when you visit the page, your browser won't show the green padlock to
show the connection is secure. If you click into the icon in the address bar you
will see something like this:

![A browser notice to say that the website connection is not
secure](../resources/insecure-notice.png)

You can also open `termshark` and look at the contents of the packets:

```shell
; termshark -i lo0 -f "port 4321"
# Now reload http://localhost:4321/
```

Here's my packet capture:

![Termshark screenshot showing unencrypted
packets](../resources/termshark-insecure-http.png)

Note that you can see the contents "Hello, world!". If this weren't localhost,
and was instead a website on the Internet, any of the routers in the middle
could see that contents!

Now we're going to use a tool called
[`mkcert`](https://github.com/FiloSottile/mkcert) to create a certificate for
our server. This will allow us to create a secure connection.

```shell
; brew install mkcert
; mkcert -install
# This will install a root certificate on your computer so that you can trust
# certificates created by mkcert.
# You can ignore "ERROR: no Firefox security database found" if you see it.
; mkcert localhost
# This will create a certificate pair valid for localhost and store them in
# `localhost.pem` and `localhost-key.pem`. You can take a look at them if you
# like.
```

Now let's configure our Flask app to use the certificate:

```python
# Replace the two lines at the end of `app.py` with these:
if __name__ == '__main__':
    app.run(port=4321, ssl_context=('localhost.pem', 'localhost-key.pem'))
```

Now restart the server and open the page again:

```shell
; python app.py
; open https://localhost:4321/
```

<details>
  <summary>:confused: I get 'This site can't be reached!'</summary>

  ---

  Make sure you've got `https` at the start of the url _with the `s`_. It won't
  work if you haven't.

  Contact your coach if you're still having trouble.

  ---

</details>

Now when you click into the icon in the address bar you will see something like
this:

![A browser notice to say that the website connection is
secure](../resources/secure-notice.png)

And if you try to snoop on the connection with `termshark` you'll see that the
contents are encrypted. You won't be able to see the plain text contents of the
packets.

## Investigations

_These exercises are marked with :hot_pepper: emojis to denote how challenging
they are. A single chilli :hot_pepper: is the most straightforward, and five
:hot_pepper::hot_pepper::hot_pepper::hot_pepper::hot_pepper: would be
challenging even for a professional engineer. Pick whichever you prefer._

This is a set of questions you can investigate to learn more. Pick the ones that
interest you.

* :hot_pepper: When is it appropriate to share your private key?
* :hot_pepper::hot_pepper: What is meant by trusting or not trusting a
  certificate?
* :hot_pepper::hot_pepper: What is Let's Encrypt? What was its purpose and what
  impact has it had on the web?
* :hot_pepper::hot_pepper: What other protocols use public-key cryptography?
* :hot_pepper::hot_pepper::hot_pepper: So we can secure HTTP, but what about
  DNS? Can anyone see your DNS lookups and thus the domains you're intending to
  visit?
* :hot_pepper::hot_pepper::hot_pepper::hot_pepper: Let's say I gained control of
  a router between you and your bank's website. When you connect via HTTPS and
  ask for the server's public key, in principle I might intercept this and send
  my own public key to you instead. Then when you encrypt your bank details, I
  can decrypt them with my private key. What technologies and systems prevent
  this? In what situation might they fail?


[Next Challenge](07_docker_bite.md)

<!-- BEGIN GENERATED SECTION DO NOT EDIT -->

---

**How was this resource?**  
[😫](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=01_internet%2F06_mkcert_bite.md&prefill_Sentiment=😫) [😕](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=01_internet%2F06_mkcert_bite.md&prefill_Sentiment=😕) [😐](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=01_internet%2F06_mkcert_bite.md&prefill_Sentiment=😐) [🙂](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=01_internet%2F06_mkcert_bite.md&prefill_Sentiment=🙂) [😀](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=01_internet%2F06_mkcert_bite.md&prefill_Sentiment=😀)  
Click an emoji to tell us.

<!-- END GENERATED SECTION DO NOT EDIT -->
