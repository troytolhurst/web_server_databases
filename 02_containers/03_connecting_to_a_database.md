# Connecting a database

In this phase you will wrap a simple web application into containers and deploy
them to a toy cloud hosting system we have provided. We've containerised our
web server and deployed it. 

Next we'd like to set up a cloud deployment with a database. Our last app was
too simple to need a database however, so we'll shift to one that does. To
deploy this, we'll need to deploy a database and connect to it.

## Objective

Learn to explain how data is persisted in the cloud.

## Guidance

[Video Alternative](https://www.youtube.com/watch?v=JmwqJu2cCUQ&t=0s)

Docker containers are what's called _ephemeral_. You start them, you can change
some files within them, and then if you stop the container those files all go
away.

This is in some ways annoying — but it's rather good for a container. They are
supposed to guarantee that if you start them they're going to run your
application. It's easier to say that if you start from a fresh container every
time.

This means we need to think carefully about how we persist data. Typically data
must leave the container to be saved.

One option is to set up your own container for Postgres using Docker and
configure it to save data to some persistent storage outside of the container.
However this is not the most common approach. Instead, most organisations will
use specific cloud computing services designed for databases.

Cloud providers like AWS and GCP have specific services for this purpose. One of
the AWS services is called [Relational Database
Service](https://aws.amazon.com/rds/) (RDS). You would set up an RDS using a web
panel or a CLI and then receive an address and authentication details to access
the database.

<details>
  <summary>:speech_balloon: What if I was using a non-relational database?</summary>

  ---

  Same idea, but you'd use a different service rather than RDS. You might use
  [AWS DynamoDB](https://aws.amazon.com/dynamodb/) or [MongoDB
  Atlas](https://www.mongodb.com/atlas)

  ---

</details>

The Makers scaled down cloud hosting system doesn't have RDS, so we're going to
take a slightly different approach. We're going to write an Exoframe config file
that sets up a database for us that we can connect to from our web server.

There's an example project in
[`codebases/database_setup`](../codebases/message_server). We'll talk through
the key aspects of it below. After you've read this, we'll ask you to go and set
it up yourself.

There are three key aspects:

1. Setting up the database instance
2. Setting up the web instance with the database details
3. Connecting to the database from the web server

We'll go through each in turn.
### Setting up the database instance

We're going to create an Exoframe config file. I'll share it below and comment
each part. Remember — you're extremely unlikely to use Exoframe in your job, so
don't focus on the specifics but instead the general ideas. I'll highlight the
general things you should take away with `TAKEAWAY`.

```jsonc
// Note: if you want to use this config, all the comments will need to be
// deleted. JSON doesn't support comments sadly!
{
    // You will need to change YOURNAME to your name in this file to avoid
    // clashes with other learners!

    // We'll set the name of the database instance within Exoframe
    "name": "YOURNAME-message-database",

    // We'll say this instance should _not_ have a domain.
    "domain": false,
    // TAKEAWAY: No one else should be accessing this database outside of our
    // deployment. Therefore, we should prohibit access by not giving it an
    // address accessible to the outside. This would be the same in the real
    // world.

    // But we will need a way to access it within our Cloud network. 
    // So we're going to give it a 'hostname'. This will be its name within
    // the cloud environment and we'll connect to it using this.
    "hostname": "YOURNAME-message-database",

    // We're going to use a Docker image for `postgres` version 12.
    "image": "postgres:12",
    // TAKEAWAY: Most real cloud providers will have their own special recipe
    // for their database servers. They won't use the base image from Docker.

    // TAKEAWAY: We're going to set up an environment variable.
    // More on these below.
    "env": {
      // We set one up with the name POSTGRES_PASSWORD
      // And a value for the password
      "POSTGRES_PASSWORD": "47378f40d8ff4bffdfc5b883ee4e250c0eeb26bb"
      // WARNING: No secure application will put the real password in config
      // like this. This is a temporary measure, and you will learn the right
      // way to do this soon.
    },

    // TAKEAWAY: As we said before, data must leave the container to be
    // persisted. So we're going to tell Exoframe to set up a 'volume' storage
    // device. Imagine a volume a bit like a flash drive, Dropbox, or storage
    // service.
    "volumes": ["YOURNAME_message_database_volume:/var/lib/postgresql/data"]
    // It is 'mapped' or 'mounted' onto a directory inside the container. This
    // means that when anything adds, modifies, or deletes files within that
    // mapped directory the changes will be reflected in the storage volume.
    // In this way, these changes will persist beyond the life of the container.

    // In this case, the name of the volume is YOURNAME_message_database_volume
    // And it will be mapped onto the directory `/var/lib/postgresql/data`
    // within the container.
  }
```

#### What are environment variables?

Environment variables are key to handling what we call 'secrets' in a
deployment. Secrets are, well, secret. Like passwords. So we shouldn't just put
them in our code or configuration — these might get uploaded to Github and
therefore not be secret anymore. This is a security risk and is generally
regarded as extremely poor practice.

However, our programs do need to find out about these secrets, so we rely on a
variety of techniques to get them injected into our programs without getting
into source code.

One common technique is environment variables.

Normal variables exist within a function, class, or program. Environment
variables exist within a computing environment and can be accessed by anything
running within it.

Here's an example with a basic Python program:

```python
import os

print("I print out the secret password")
if "SECRET_PASSWORD" in os.environ:
    # `os.environ` is a dictionary of all the environment variables
    print(os.environ["SECRET_PASSWORD"])
else:
    print("But there is no SECRET_PASSWORD environment variable set")
```

And if we run this:

```shell
; python3 printer.py
I print out the secret password
But there is no SECRET_PASSWORD environment variable set
```

Now let's set an environment variable:

```shell
; export SECRET_PASSWORD=fakeexamplepassword
; python3 printer.py
I print out the secret password
fakeexamplepassword
```

The password isn't stored in the code, it is stored in the environment and the
code accesses it.

Let's look at the Exoframe configuration above to see how it applies there.

```jsonc
    // ...
    "env": {
      // We set one up with the name POSTGRES_PASSWORD
      // And a value for the password
      "POSTGRES_PASSWORD": "47378f40d8ff4bffdfc5b883ee4e250c0eeb26bb"
      // WARNING: No secure application will put the real password in config
      // like this. This is a temporary measure, and you will learn the right
      // way to do this soon.
    },
    // ...
```

In the configuration above, we configure an environment variable
`POSTGRES_PASSWORD` with the value `47378f40d8ff4bffdfc5b883ee4e250c0eeb26bb`.

When Exoframe runs our container, it will set up this environment variable for
that container. The Postgres container is set up by default to look for an
environment variable by that name and use it as the database password.

This still isn't good enough though. Our Exoframe config lives along with our
code and so is likely to get uploaded to Github and therefore our secrets become
not-secret. This is very bad security practice. Let me just emphasise that:

![Stop Sign](../resources/stop-sign.png)

| :exclamation:  This is very important |
| ------------------------------------- |

| :zap:  Ignore at your own risk! |
| ------------------------------- |

| :memo:  Take note of this |
| ------------------------- |

> **Warning**  
> **Never, ever, under any circumstances commit passwords or secret keys to a
> git repository.** If you ever do this accidentally, **do not run `git push`.**
> Contact your coach or an experienced colleague for help.
>
> **If you ignore this advice, terrible things can happen.** If you're lucky,
> it will only cost a few thousand pounds to clear up. If you're unlucky, you
> make the news — maybe today, maybe a few years from now.
>
> If you do accidentally do this, and you accidentally push the code **contact
> someone experienced immediately**. They will want to help and be very glad
> that you told them quickly.

| :point_up: Remember to not forget! |
| ---------------------------------- |

I hope that's clear.

### Setting up the web instance with the database details

We'll need to set up the web application and give it the details to connect to
the database server.

We can do this with the following config:

```jsonc
{
  // We'll name it once again
  "name": "YOURNAME-message-web",

  // This one we will give a user-facing domain name
  "domain": "YOURNAME-messages.xf.mkrs.link",

  // Flask will listen on port 5000 so we have to specify that
  "port": 5000,

  // And here we set up three environment variables
  "env": {
    // One to tell the server that it should operate in production mode
    "APP_ENV": "PRODUCTION",

    // One to tell the server the hostname of the database (as specified in the
    // db config).
    "POSTGRES_HOSTNAME": "YOURNAME-message-database",

    // And one to specify the password of the database
    "POSTGRES_PASSWORD": "47378f40d8ff4bffdfc5b883ee4e250c0eeb26bb"
    // WARNING: the password should not be in the config like this!
    // We will address this in the next step.
  }
}
```

Now that we've set up this config to provide the right environment variables,
we can look at setting up the connection.

### Connecting to the database from the web server

This application will let the user add messages and list them out.

It would typically be broken out into multiple files, but to make it easier
to explain we'll keep it just in one file. The key 'cloud' part of this is
`get_database_url` — the rest of this you've seen before in other forms.

```python
import os
import psycopg
from flask import Flask, request, redirect, url_for, escape

# We're going to write a function that constructs an URL for the database
def get_database_url():
    # The app can run in two 'modes' — production mode, or development mode.
    # This is determined by the `APP_ENV` environment variable.
    # Having dev and production modes is quite a common pattern and you will
    # see it in many real-world applications.
    if os.environ.get("APP_ENV") == "PRODUCTION":
        password = os.environ.get("POSTGRES_PASSWORD")
        hostname = os.environ.get("POSTGRES_HOSTNAME")
        # This URL below is constructed out of the password and hostname
        # We'll use this URL to connect to the database in production
        return f"postgres://postgres:{password}@{hostname}:5432/postgres"
    else:
        # This URL is for our local database. You may need to edit it.
        return "postgres://localhost:5432/postgres"

# We're going to write a function that sets up the database with the right table
def setup_database(url):
    # We connect using the URL
    connection = psycopg.connect(url)

    # Get a 'cursor' object that we can use to run SQL
    cursor = connection.cursor()
    
    # Execute some SQL to create the table
    cursor.execute("CREATE TABLE IF NOT EXISTS messages (message TEXT);")

    # And commit the changes to ensure that they 'stick' in the database.
    connection.commit()

# We run the two functions above
POSTGRES_URL = get_database_url()
setup_database(POSTGRES_URL)

app = Flask(__name__)

# Below are two fairly ordinary Flask routes

@app.route("/")
def get_messages():
    # Connect to the database
    connection = psycopg.connect(POSTGRES_URL)

    # Run some SQL
    cursor = connection.cursor()
    cursor.execute("SELECT * FROM messages;")

    # Get the results
    rows = cursor.fetchall()

    # Format the results and add a form too
    return format_messages(rows) + generate_form()

# These two methods generate HTML lists and forms
def format_messages(messages):
    output = "<ul>"
    for message in messages:
        # We escape the message to avoid the user sending us HTML and tricking
        # us into rendering it.
        escaped_message = escape(message[0])
        output += f"<li>{escaped_message}</li>"
    output += "</ul>"
    return output

def generate_form():
    return """
    <form action="/" method="POST">
        <input type="text" name="message">
        <input type="submit" value="Send">
    </form>
    """

# This method receives the POST request from the form above
@app.route("/", methods=["POST"])
def post_message():
    # We extract the message from the request
    message = request.form["message"]

    # Insert a new message record into the database
    connection = psycopg.connect(POSTGRES_URL)
    cursor = connection.cursor()
    cursor.execute("INSERT INTO messages (message) VALUES (%s);", (message,))
    connection.commit()

    # And redirect to the main page
    return redirect(url_for("get_messages"))

if __name__ == '__main__':
    # We also run the server differently depending on the environment.
    # In production we don't want the fancy error messages — users won't know
    # what to do with them. So no `debug=True`
    if os.environ.get("APP_ENV") == "PRODUCTION":
        app.run(port=5000, host='0.0.0.0')
    else:
        app.run(debug=True, port=5000, host='0.0.0.0')
```

## Deployment

Once you've got this project set up, deploying it is like this:

```shell
; cd message_server
; cd db
# Make sure you edit exoframe.json to change YOURNAME to e.g. kaylack
; exoframe deploy
; cd ../web
# Make sure you edit exoframe.json to change YOURNAME to e.g. kaylack
; exoframe deploy

# And if you want to update the web app:
; exoframe deploy --update
```

Wait a minute or so, and then you can visit your new app at something like
`https://YOURNAME-messages.xf.mkrs.link`

## Exercise

Get your own version of this deployed and working.

Once you've done this, upload the full setup to Github and send the link over to
your coach to take a look at and make sure it's all OK.

<!-- OMITTED -->

<details>
  <summary>:hot_pepper::hot_pepper: Can I have a bit more challenge?</summary>

  ---

  Sure — try altering the application to use Jinja templates, like in the web
  module, rather than raw strings.

  ---

</details>

Then move on to the next step. There will be more there will be a lot of
opportunity to work on your project in the CI-CD phase.


[Next Challenge](04_removing_secrets.md)

<!-- BEGIN GENERATED SECTION DO NOT EDIT -->

---

**How was this resource?**  
[😫](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F03_connecting_to_a_database.md&prefill_Sentiment=😫) [😕](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F03_connecting_to_a_database.md&prefill_Sentiment=😕) [😐](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F03_connecting_to_a_database.md&prefill_Sentiment=😐) [🙂](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F03_connecting_to_a_database.md&prefill_Sentiment=🙂) [😀](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F03_connecting_to_a_database.md&prefill_Sentiment=😀)  
Click an emoji to tell us.

<!-- END GENERATED SECTION DO NOT EDIT -->
