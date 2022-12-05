# Containerising

In this phase you will wrap a simple web application into containers and deploy
them to a toy cloud hosting system we have provided. We're going to start
with containerising the web server.

## Objective

Learn to containerise a web server using docker.

## Guidance

<!-- OMITTED -->

_Video coming soon._

> **Note**  
> Unlike some previous exercises, this sort of work can be a bit
> paint-by-numbers. We won't shy away from this — if there's one right way to do
> it, we'll show you that way and you should do it just like that.
>
> We'll include some extra tasks and gaps for you to experiment where it makes
> sense to keep things interesting.

### What are containers?

Start by [installing Docker](https://www.docker.com). It might take a while so
you can read down while you wait.

When you containerise a piece of software, you use a tool to gather up
absolutely everything that the software needs to run. Everything from its
dependencies, to the language version, to even having a terminal to run in.

When you've done this you have what's called a 'container'. You can run
containers in any compatible container-hosting system and it will (or really
_should_ run exactly the same as it does on your local computer).

<details>
  <summary>Why are they called containers?</summary>

  ---

  They are called containers in an analogy to a shipping container. Shipping
  containers are a standardised way of shipping goods. They are the same size,
  shape, and have the same fittings. This means that the international shipping
  system can broadly ignore what's inside and treat them all in the same way.

  This is what containerised software is designed to achieve. You can take your
  software, package it up in a standardised way, and then send it to your cloud
  host. That host can totally ignore the specifics of your software and treat it
  in the same way as any other containerised software.

  ---

</details>

One popular system for doing this is called Docker. We're going to use it to
package up a small Flask app.

First, we'll need a Flask app. You can write your own or use one we've provided.

If you want to make your own, write a Flask app that:

* Responds to `GET /` with `Hello, world!` or something similar.
* Runs with `python app.py`

If you'd prefer to use one of ours, you can find one in
[`../codebases/simple_server`](../codebases/simple_server).

### Preparing to containerise

You'll need to make a couple tweaks to your Flask app to make it run properly
in a container. This isn't unusual — most software does need a few adjustments
to prepare it for deployment.

The key change for us is to `app.rb` in the section where you run the flask
server. Yours might look like this:

```python
if __name__ == '__main__':
    app.run(debug=True)
```

Or you might not have it at all! In any case, you'll need one and for it to look
like this:

```python
if __name__ == '__main__':
    app.run(
      debug=True, # Optional but useful for now
      host="0.0.0.0" # Listen for connections directed _to_ any address
    )
```


The `host="0.0.0.0"` option tells Flask to listen for connections that are
directed _to_ any IP address (think of `0.0.0.0` as a wildcard) as opposed to
just the ones coming to localhost `127.0.0.1`. We need this because the
container's new localhost is just going to be connections from inside the
container. We're outside the container, so we need to tell it to listen more
generally.

If you can run this and see roughly this output you're in a good place:

```shell
; pipenv run python app.py
 * Serving Flask app 'app'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://192.168.8.115:5000 # Note the second IP address here
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 738-310-080
```

### Containerising

In order to wrap our server in a container we need to create a recipe describing
everything it will need to run successfully. Since we're using Docker, this is
called a `Dockerfile`.

Here's the `Dockerfile` you will need. It includes some comments that explain
what it does. If you're using our sample code then it's already there for you.

```dockerfile
# Imagine this file as a recipe for setting up a virtual computer.

# Dockerfiles typically start with a 'base image'. There are loads of these
# and you can find them at hub.docker.com.
# We're going to use a base image for Python veresion 3.11
FROM python:3.11

# This base image contains essentially everything necessary for a 'virtual
# computer'. It has a terminal, certain basic commands, and of course Python.

# We run a command to install `pipenv`
RUN pip install pipenv

# We'll need our app's files in the container in order to be able to run them!
# We copy them in from the current directory to the folder `/app` in
# our virtual computer. Reminder `.` means 'the current directory'
COPY . /app

# We set the working directory for commands from this point on
WORKDIR /app

# We run `pipenv install` to install our project's dependencies. Since we've
# copied in our `Pipfile`, `pipenv` will use that to get a list of dependencies.
# We include a couple of extra options suitable for deployment.
RUN pipenv install --system --deploy

# At this point we've set up our virtual computer, but we've not _yet_ run our
# application. And we're not going to! We're just setting up the container
# so that it's ready to do so when we tell it.

# So we're going to tell Docker here that when we _do_ want to run it, this is
# what it should run:
CMD ["python", "app.py"]
```

<details>
  <summary>:speech_balloon: Virtual computer? I thought you said it was a container?</summary>

  ---

  Good observation! A container isn't actually a virtual computer, it's more of
  a virtual environment that we can run programs in. This might seem like a
  very subtle difference, and it is.

  The difference is that a virtual computer — sometimes called virtualisation —
  involves simulating the hardware of a computer. Docker doesn't do that.
  Instead, Docker uses the hardware of the computer it is on but packages up all
  of the environment that each program needs and runs it just like it was one
  single program.

  This _is_ a fairly subtle distinction so don't worry about it too much.

  ---

</details>

Now we've got a `Dockerfile` we'll get Docker to build it:

```shell
# `--tag my-flask-app` gives the resultant container the name `my-flask-app`
; docker build --tag my-flask-app .
[+] Building 7.0s (10/10) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 1.42kB
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/python:3.11
 => [1/5] FROM docker.io/library/python:3.11
 => [internal] load build context
 => => transferring context: 2.06kB
 => CACHED [2/5] RUN pip install pipenv
 => [3/5] COPY . /app
 => [4/5] WORKDIR /app
 => [5/5] RUN pipenv install --system --deploy
 => exporting to image
 => => exporting layers
 => => writing image sha256:e3694d992f1c3140fd7029187d751e2c058e9626c89127455e3898e799ea976
 => => naming to docker.io/library/my-flask-app
```

You can see it running the commands we gave it.

<details>
  <summary>:confused: I see 'Error response from daemon...'</summary>

  ---

  You'll need to start Docker and keep it running for as long as you're using it.
  It should be in your Applications folder.

  If you keep running into trouble, contact your coach.

  ---

</details>

Docker will store our container image in its internal database. We can see it
like this:

```shell
; docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
my-flask-app        latest              e3694d992f1c        2 minutes ago       948MB
```

You can also see them in the Docker Desktop app. These can build up over time so
be sure to keep an eye on them and clear any old ones you don't need.

### Running the container

Now we've got a container image we can run it. We'll do this with the `docker
run` command. This will start a new container based on the image we've built.

```shell
; docker run --publish 5001:5000 my-flask-app
 * Serving Flask app 'app'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.17.0.2:5000
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 124-052-709
```

<details>
  <summary>:confused: I see 'Cannot connect to the Docker daemon at ...'</summary>

  ---

  You'll need to start Docker and keep it running for as long as you're using it.
  It should be in your Applications folder.

  If you keep running into trouble, contact your coach.

  ---

</details>

Let's break down that command:

* `docker run` — this runs a container image
* `--publish 5001:5000` — this tells Docker to forward requests from port 5001
  on the host computer to port 5000 in the container. The container doesn't
  have direct access to the host's ports so we need to tell Docker to connect
  them together explicitly.
* `my-flask-app` — this is the name of the container image we want to run

Remarkable! We've achieved the same thing we had before all of this palaver!
This might seem pointless — but it's actually very good. We can now run our
application without trouble on any other computer running Docker with little
additional setup — this includes all of the major cloud computing providers like
AWS, Azure, and Google Cloud Platform.

In the next step we'll deploy it to the cloud.

## Investigations

_These exercises are marked with :hot_pepper: emojis to denote how challenging
they are. A single chilli :hot_pepper: is the most straightforward, and five
:hot_pepper::hot_pepper::hot_pepper::hot_pepper::hot_pepper: would be
challenging even for a professional engineer. Pick whichever you prefer._

This is a set of questions you can investigate to learn more. Pick the ones that
interest you.

* :hot_pepper: How do you run a container in the background?
* :hot_pepper: How do you stop a container running in the background?
* :hot_pepper::hot_pepper: Docker can in principle containerise any application.
  Can you find any cool or useful containers out there?


[Next Challenge](02_deploying.md)

<!-- BEGIN GENERATED SECTION DO NOT EDIT -->

---

**How was this resource?**  
[😫](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F01_containerising.md&prefill_Sentiment=😫) [😕](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F01_containerising.md&prefill_Sentiment=😕) [😐](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F01_containerising.md&prefill_Sentiment=😐) [🙂](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F01_containerising.md&prefill_Sentiment=🙂) [😀](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F01_containerising.md&prefill_Sentiment=😀)  
Click an emoji to tell us.

<!-- END GENERATED SECTION DO NOT EDIT -->
