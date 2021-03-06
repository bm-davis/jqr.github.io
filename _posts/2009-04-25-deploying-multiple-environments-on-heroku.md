---
layout: post
title: Deploying Multiple Environments on Heroku
summary: Run multiple branches/environments of the same code on Heroku.
---

<p class="update">UPDATE: This post is outdated, check out <a href="/2010/08/27/easy-heroku-deploys-with-heroku-san.html">Easy Deploys with Heroku San</a>.</p>

<p>So you followed yesterday's guide on <a href="/2009/04/24/deploy-your-rails-application-in-2-minutes-with-heroku.html">getting your application deployed to
Heroku</a>, and now you've fallen in love with it. Now you need to setup both a
production and staging environment that you can easily deploy your application
to.</p>

<p>So far Heroku only runs code in one branch, so we'll work around this by
creating two remote repositories that we can push to.</p>

<p>For example simplicity, we'll be pushing all code out of the same git
branch. You'll probably be pushing staging and production servers from different branches, adjust accordingly.</p>

<h3>Create your servers and fix your remotes</h3>

Application names have to be unique on Heroku, so make sure to replace myapp with your application's name. You can do that automatically by <a href="#" id="replace_application_name">clicking here</a>.

```sh
heroku create myapp-staging --remote staging
heroku create myapp-production --remote production
```

<p>Since we'll be pushing to two applications we are using the --remote argument to make two sensibly named remotes.</p>

<h3>Environment specific variables</h3>

<p>Heroku has a nice interface for setting up application specific settings, but
I will assume your application configures itself according to the
RAILS_ENV variable. Check out <a
href="http://docs.heroku.com/config-vars">Heroku's docs</a> if you need more
control.</p>

<p>If you run your staging server in the production environment, you can skip this step.</p>

```sh
heroku config:add RACK_ENV=staging --app myapp-staging
heroku config:add RACK_ENV=production --app myapp-production
```

<h3>Deploy and migrate</h3>

<p>You'll be doing this next command pretty often. We push the current branch to
the staging server's master branch. Since this is the first deploy we'll need to do the same thing for production and run all the migrations.</p>

```sh
git push staging master
git push production master

heroku rake db:migrate --app myapp-staging
heroku rake db:migrate --app myapp-production

heroku open --app myapp-staging
heroku open --app myapp-production
```

<h3>Done</h3>

<p>Celebrate! You now have two servers running on Heroku with different databases and accessible through domains. Seriously, how long did that take?</p>

<h3>Caveats</h3>

<p>The heroku command seems to detect the application name by looking through
git's remotes for remotes that are located at heroku.com. This means it will
get confused by the multiple entries we created. You will need to work around
this by passing --app myapp-staging after the normal heroku command.</p>

<h3>Troubleshooting</h3>

<p>Make sure you followed my original guide on <a
href="/2009/04/24/deploy-your-rails-application-in-2-minutes-with-heroku.html">getting
your application deployed to Heroku</a>.</p>

<script type="text/javascript">
  document.observe('dom:loaded', function() {
    $('replace_application_name').observe('click', replaceApplicationName)
  });
  var old_name = 'myapp';
  function replaceApplicationName(event) {
    event.stop();
    var new_name = prompt('Enter your application name:');
    $$('pre').each(function(element) {
      element.update(element.innerHTML.replace(RegExp(old_name, 'g'), new_name));
    });
    old_name = new_name;
  }
</script>
