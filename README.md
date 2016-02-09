# Heroku SSH Buildpack

This is a [Heroku Buildpack](https://devcenter.heroku.com/articles/buildpacks)
that allows you to SSH into a running [dyno](https://devcenter.heroku.com/articles/dynos).
The SSH port will be proxied by [ngrok](https://ngrok.com/), which makes it accessible from a remote machine.

## Setup

First, create a free [ngrok account](https://dashboard.ngrok.com/user/signup). This is necessary to use TCP with their service. Then capture your API key, and set it as a config var on your Heroku app like this:

```
$ heroku config:set NGROK_API_TOKEN=xxxxxx
```

Next, add this buildpack to your app:

```
$ heroku buildpacks:clear
$ heroku buildpacks:set https://github.com/jkutner/heroku-buildpack-ssh.git
```

Then add your primary buildpack. For example, if you are using Ruby:

```
$ heroku buildpacks:add heroku/ruby
```

Now modify your `Procfile` by prefixing your `web` process with the `with_ssh` command. For example:

```
web: with_ssh bundle exec puma -C config/puma.rb
```

Finally, commit your changes, and redeploy the app:

```
$ git add Procfile
$ git commit -m "Added with_ssh"
$ git push heroku master
```

## Usage

Once your app is running with the SSH buildpack and the `with_ssh` command, you'll see something like
this in your logs:

```
2015-05-19T16:06:36.530988+00:00 app[web.1]: Starting sshd for u18370
```

Download the SSH key and set it's permissions by running these commands:

```
$ heroku run cat .ssh/id_rsa > ~/.ssh/heroku_id_rsa
$ chmod 600 ~/.ssh/heroku_id_rsa
```

Browse to your [ngrok dashboard](https://dashboard.ngrok.com/) to see the address of the proxy.
It will be something `tcp://0.tcp.ngrok.io:40306`. Use this host and port with the
username in the logs to create an SSH command like this:

```sh-session
$ ssh -i ~/.ssh/heroku_id_rsa -p 40306 u18370@0.tcp.ngrok.io
...
Are you sure you want to continue connecting (yes/no)? yes
...
+--------------------
| Welcome to Heroku!
+--------------------
~ $
```

Now you can inspect a running process:

```
~ $ ps aux | grep ruby
u18370      64  0.5  0.0  27792  4760 pts/0    Sl   21:40   0:00 bundle exec puma -C config/puma.rb
u18370      67  0.0  0.0   8864   652 pts/0    S+   21:40   0:00 grep ruby
```
