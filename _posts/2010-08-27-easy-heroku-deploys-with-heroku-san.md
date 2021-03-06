---
layout: post
title: Easy Heroku Deploys with Heroku San
summary: Deploy and maintain multiple Heroku apps with ease.
---

We deploy a great deal of our apps to <a href="http://heroku.com">Heroku</a>, but maintaining multiple environments (staging, production, demo) was traditionally <a href="/2009/04/25/deploying-multiple-environments-on-heroku.html">very painful</a>.

<a href="http://github.com/fastestforward/heroku_san">heroku_san</a> is a simple set of rake tasks to make standard Rails deployment easy. Multiple apps, environments, branches and users are handled with the minimum of effort.

### Initial Setup

I won't bore you with the details of <a href="http://github.com/fastestforward/heroku_san/blob/master/README.rdoc">installing the gem and including the rake tasks</a>, but you need to know that heroku_san works by using a YAML file with shorthand versions of your Heroku application names. It looks like this:

```yaml
apps:
  # shorthand: heroku app
  production: awesomeapp
  staging: awesomeapp-staging
  demo: awesomeapp-demo
```

### First Time

If this app has never been deployed to Heroku, you will need to create the remote applications. heroku_san can do this for you, using the data in config/heroku.yml.

```sh
rake all heroku:create
```

Most multi-app deployments change the environment to something different for each app, allowing you to override things like S3 buckets, email policies, etc. heroku_san is happy to handle this too, it assumes you want the RACK_ENV set to the shorthand name of the application.

```sh
# set RACK_ENV on each application to the shorthand:
#   awesomeapp-staging => staging
#   awesomeapp-demo    => demo
rake all heroku:rack_env
```

The old Heroku stack (aspen) requires you to use a Gem manifest (.gems) to list the gems your application depends on. heroku_san can auto-populate this using config.gem requirements for Rails 2 applications.

```sh
rake heroku:gems
```

### Everyday Use

```sh
rake staging deploy            # deploy staging and migrate
rake production console        # open a console for production
rake demo staging heroku:share # add a new developer to demo and staging
rake all heroku:unshare        # remove a developer from all apps
```


### Branches

Deploying always uses the current branch, so be careful! Occasionally you might need to force a deploy, especially if you're deploying multiple feature branches to the same app.

```sh
rake staging deploy            # deploy current branch to staging
rake staging force_deploy      # force deploy current branch to staging
```

### Only One App?

If you're only using one app, you can skip the server names and just issue the commands directly.

```sh
rake deploy                    # deploy the only app you have configured
rake console                   # open a console on your single app
```

### Curious?

heroku_san works entirely by using the git and heroku binaries. So if you're ever curious about how a command works or need to debug something you can just watch the screen as you run a command to see exactly what it's up to.

```sh
rake staging deploy
# git push git@heroku.com:awesomeapp-staging.git  master:master && \
# heroku rake --app awesomeapp-staging db:migrate && \
# heroku restart --app awesomeapp-staging
```

### Even More!

A full list of commands heroku_san provides is available in the <a href="http://github.com/fastestforward/heroku_san/blob/master/README.rdoc">README</a>.

### Special Thanks

Thanks to <a href="http://github.com/glennr">Glenn Roberts</a>, who was kind enough to convert heroku_san from a Rails plugin into a gem, making updates a breeze.
