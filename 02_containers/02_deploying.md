# Deploying

In this phase you will wrap a simple web application into containers and deploy
them to a toy cloud hosting system we have provided. We've containerised our
web server and now we're going to deploy it to the cloud.

## Objective

Learn to explain how containerisation helps with deployment.

## Guidance

[Video Alternative](https://www.youtube.com/watch?v=N4ZtwCXP6C4&t=1085s)

We're going to use a tool called Exoframe to deploy our container to the Cloud.
This is a simplified cloud deployment system, providing only the most essential
functionality. In the real world you will use a cloud provider like AWS Elastic
Container Service or similar.

### Setting up Exoframe

We need to install and then configure the Exoframe tool. 

> **Note**  
> You will need a certificate from your coach to follow the below instructions.
> It will end in `.pem`. It's important you keep _any_ `.pem` file either here
> or at Makers secret. Treat it like a password — don't commit it to git or
> upload it to Github.
>
> If you don't have one or can't find it, speak to your coach.

Once you've got your certificate, follow the instructions below.

```shell
# We'll need to install NodeJS first as it is a 
; brew install node
# Now we'll install exoframe
; npm install exoframe -g
# And log in to the Makers exoframe server
; exoframe login https://exoframe.xf.mkrs.link -k path/to/key.pem
Endpoint URL updated!
Logging in to: https://exoframe.xf.mkrs.link
? Username: # Type in your name here as one word, e.g. 'kaylack'
Successfully logged in!
; exoframe ls # Let's just check it works
No deployments found on https://exoframe.xf.mkrs.link!
# We're good to go! If not, contact your coach.
```

We've used our private key to log in to our cloud host. If you were using AWS or
another cloud provider the process would be similar. You would set up a command
line tool and log in using secret credentials.

### Configuring our app to deploy

Now let's configure our app to deploy.

```shell
; exoframe config -d YOURNAME-simple.xf.mkrs.link --port 5000
```

Let's break this down:

* `exoframe config` — this is the command to configure our app for deployment
* `-d YOURNAME-simple.xf.mkrs.link` — this is the domain we want to deploy to.  
  It has to end in `.xf.mkrs.link`, and the part before needs to be unique
  across all your fellow students who are using the service. So, best thing to
  do is have it be `firstnamelastname-simple.xf.mkrs.link`.
* `--port 5000` — this is the port our container is listening on. Our container
  is listening for HTTP requests on port 5000, so we need to tell Exoframe that.

### Deploying our app

Now we're ready to deploy our app. When we deploy our app, Exoframe is going
to send over all of our files including our Dockerfile, build our container, and
then run it in the cloud.

<details>
  <summary>:thought_balloon: So it's not going to run the image we built?</summary>

  ---

  No. It could but that's not how we're doing it this time. It's just going to
  use the recipe in our Dockerfile.

  ---

</details>

Let's do it!

```shell
; exoframe deploy
Deploying current project to endpoint: https://exoframe.xf.mkrs.link
✔ Deployment finished!
Your project is now deployed as:

   ID                                   URL                               Hostname       Type
   exo-kay-simple-server-24754400       kaylack-simple.xf.mkrs.link       Not set        Container
```

We can see it starting by looking at the logs:

```shell
# We pass the ID we've got in the list above to the logs command
; exoframe logs exo-kay-simple-server-24754400
Getting logs for deployment: exo-kay-simple-server-24754400

03/12/2022 22:02:14  * Serving Flask app 'app'
    * Debug mode: on
   WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
    * Running on all addresses (0.0.0.0)
    * Running on http://127.0.0.1:5000
    * Running on http://172.17.0.4:5000
   Press CTRL+C to quit
    * Restarting with stat
    * Debugger is active!
    * Debugger PIN: 383-003-380
```

Now we can go to the URL and see our app running:

```shell
# Your domain will be different, so use the one you specified above
open https://kaylack-simple.xf.mkrs.link
```

<details>
  <summary>:thought_balloon: I see a security error...</summary>

  ---

  This can happen if you visit it very quickly — the security certificate may
  not be properly in place yet. Give it a minute or so, close the tab, and then
  try again.

  ---

</details>

Congratulations, your application is running on the Internet! Anyone can go and
visit it.

Which means it's time to make it your own. Hello world isn't enough! Spend
an hour or two building a website you can be proud of.

When you want to update your server with the latest code, run:

```shell
; exoframe deploy --update
```

If you don't include that `--update` you will end up with multiple copies of
your server that are randomly picked between! If you end up in that situation,
run `exoframe ls` to look at all your deployments, and then `exoframe remove
ID_OF_DEPLOYMENT` to remove all the extras.

<details>
  <summary>:speech_balloon: Will my website be there forever now?</summary>

  ---

  Sadly not — we need to keep the server clear for future students, so we will
  periodically delete all the hosted containers.

  If you want to deploy your container to another cloud hosting system though
  you're very welcome! [Render](https://www.render.com) or
  [Heroku](https://www.heroku.com) come well-recommended for small projects,
  while providers like Azure, GCP, and AWS are typically used in professional
  engineering teams.

  Nonetheless, just because it doesn't last forever doesn't mean it isn't
  special. Go build yourself a website.

  ---

</details>


[Next Challenge](03_connecting_to_a_database.md)

<!-- BEGIN GENERATED SECTION DO NOT EDIT -->

---

**How was this resource?**  
[😫](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F02_deploying.md&prefill_Sentiment=😫) [😕](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F02_deploying.md&prefill_Sentiment=😕) [😐](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F02_deploying.md&prefill_Sentiment=😐) [🙂](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F02_deploying.md&prefill_Sentiment=🙂) [😀](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F02_deploying.md&prefill_Sentiment=😀)  
Click an emoji to tell us.

<!-- END GENERATED SECTION DO NOT EDIT -->
