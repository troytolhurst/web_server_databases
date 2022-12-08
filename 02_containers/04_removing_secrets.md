# Removing Secrets

In this phase you will wrap a simple web application into containers and deploy
them to a toy cloud hosting system we have provided. We've containerised our
web server, deployed it, connected it to a database, and now we're going to
remove the secrets from the config.

## Objective

Learn to explain how secrets are removed from config and code.

## Guidance

[Video Alternative](https://www.youtube.com/watch?v=JmwqJu2cCUQ&t=1729s)

_Video coming soon._

In the last step you moved secrets out of your code into environment variables.
Now it's time to remove them from that config too.

This is done by storing them in a special area in the cloud system itself. We'll
use the `exoframe secret` feature to do this.

Let's take the database config as an example:

```jsonc
{
  "name": "YOURNAME-message-database",
  "domain": false,
  "hostname": "YOURNAME-message-database",
  "image": "postgres:12",
  "env": {
    "POSTGRES_PASSWORD": "47378f40d8ff4bffdfc5b883ee4e250c0eeb26bb"
  },
  "volumes": ["YOURNAME_message_database_volume:/var/lib/postgresql/data"]
}
```

We will change the `POSTGRES_PASSWORD` entry to the following:

```jsonc
  // ...
  "POSTGRES_PASSWORD": "@YOURNAME-postgres-password"
  // ...
```

`@YOURNAME-postgres-password` will act as a placeholder that we can tell
Exoframe to fill in for us with the real secret.

We'll do the same in the `web` project too:

```jsonc
{
  "name": "YOURNAME-message-web",
  "domain": "YOURNAME-messages.xf.mkrs.link",
  "port": 5000,
  "env": {
    // These aren't secrets so we don't need to worry about them
    "APP_ENV": "PRODUCTION",
    "POSTGRES_HOSTNAME": "YOURNAME-message-database",

    // But this is
    "POSTGRES_PASSWORD": "@YOURNAME-postgres-password"
  }
}
```

Then, before re-deploying this, we will tell Exoframe to store the secret in its
secrets database and then use it when it sees that placeholder.

```shell
; exoframe secret set YOURNAME-postgres-password 47378f40d8ff4bffdfc5b883ee4e250c0eeb26bb
Generating new deployment secret for: https://exoframe.xf.mkrs.link
New secret generated:

Name: kaylack-postgres-password
Value: 47378f40d8ff4bffdfc5b883ee4e250c0eeb26bb

DONE!
```

And then let's deploy:

```shell
; cd db
; exoframe deploy --update
; cd ../web
; exoframe deploy --update
```

Check that everything still works.

Now your code and config is entirely free of secrets and safe to work with `git`
and Github! Exoframe secrets are specific to Exoframe, but all cloud systems
will have some way of storing secrets safely. Look for 'environment variables'
or a 'secrets' section in whatever systems you end up using in your career.

In the next phase, we'll put that secrets to further use by setting up CI and
CD.


<!-- BEGIN GENERATED SECTION DO NOT EDIT -->

---

**How was this resource?**  
[😫](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F04_removing_secrets.md&prefill_Sentiment=😫) [😕](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F04_removing_secrets.md&prefill_Sentiment=😕) [😐](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F04_removing_secrets.md&prefill_Sentiment=😐) [🙂](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F04_removing_secrets.md&prefill_Sentiment=🙂) [😀](https://airtable.com/shrUJ3t7KLMqVRFKR?prefill_Repository=makersacademy%2Fcloud-deployment&prefill_File=02_containers%2F04_removing_secrets.md&prefill_Sentiment=😀)  
Click an emoji to tell us.

<!-- END GENERATED SECTION DO NOT EDIT -->
