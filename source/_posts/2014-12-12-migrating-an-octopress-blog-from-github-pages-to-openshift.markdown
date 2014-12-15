---
layout: post
title: "Deploying an Octopress site to Openshift"
date: 2014-12-14 20:15:00 -0500
comments: true
categories: octopress openshift ruby
---

I'll detail how to take a new or existing GH Pages blog in Octopress and deploy it to OpenShift.

###Why?

I recently moved my blog from Github Pages to [OpenShift Online](https://www.openshift.com/). I did this because I wanted to utilize SSL with my custom domain - [a feature not currently available on Github Pages](https://github.com/isaacs/github/issues/156) (it's 12/14/2014 - let me know when this becomes possible!). OpenShift supports SSL with a free account, so I decided to make the switch. I wanted to use the existing architecture set up by [Octopress](https://github.com/imathis/octopress) that built my GH Pages blog. Hopefully you see a little lock icon in your address bar if you're reading this from [the site](/).

###<a name="step0"></a>Step 0: Create an OpenShift account

Here's the link: [https://www.openshift.com/app/account/new](https://www.openshift.com/app/account/new). They're serious about the first and second options being free. The difference is signing up for the second option requires credit card information to give you the possibility of scaling your application. The second option is what allows you to add certs for doing SSL.

###<a name="step1"></a>Step 1: Create a Ruby 1.9.3 application

Octopress still uses ruby 1.9.3; for simplicity's sake, so will we. Assuming you have ruby-1.9.3 installed and the rhc gem installed (gem install rhc), create a new ruby-1.9 application on OpenShift:

``` bash
$ rhc app create octopress ruby-1.9
```

replacing "octopress" with your desired application name. I've found this action takes some time, usually over a minute, while OpenShift allocates resources. When your application has been created, you should see an `ssh://` URL for your git repository on OpenShift. Make sure you can get access to this later.

If you're using a custom domain, now is as good a time as any to set it up. OpenShift has [its own docs](https://blog.openshift.com/custom-url-names-for-your-paas-applications-host-forwarding-and-cnames-the-openshift-way/) to cover this topic.

###<a name="step2"></a>Step 2: Merge in "Octoshift"

I've made some updates to the octopress repo that I haven't requested be pulled into the master branch yet. I'll update this section in the future if necessary.

If you're new to Octopress, clone from the master branch:

``` bash
$ git clone git@github.com:imathis/octopress.git
```

Either way, `cd` into your octopress directory and merge in my fork:

``` bash
$ git remote add octoshift git@github.com:larryprice/octopress.git
$ git pull octoshift master
```

###<a name="step3"></a>Step 3: Set up deployment

There is a rake task for setting up an OpenShift deployment called `setup_openshift`. This rake task clobbers the `_deploy` directory and reinitializes it using the URL for your OpenShift repository, which we committed to memory in [Step 1](#step1).

``` bash
$ rake setup_openshift["ssh://548e1873e0b8cddccf000094@octopress-username.rhcloud.com/~/git/octopress.git/"]
rm -rf _deploy
mkdir -p _deploy/public
cd _deploy
Initialized empty Git repository in /home/lrp/Projects/2014/octopress/_deploy/.git/
remote: Counting objects: 21, done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 21 (delta 2), reused 21 (delta 2)
Unpacking objects: 100% (21/21), done.
From ssh://octopress-username.rhcloud.com/~/git/octopress
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> origin/master
cd -
cp config.ru _deploy/
cd _deploy
[master 5204046] Octopress init
 3 files changed, 27 insertions(+), 295 deletions(-)
 create mode 100644 Gemfile
 rewrite config.ru (99%)
 create mode 100644 public/index.html
cd -

---
## Now you can deploy to ssh://548e1873e0b8cddccf000094@octopress-username.rhcloud.com/~/git/octopress.git/ with `rake deploy` or `rake openshift` ##
```

###<a name="step35"></a>Step 3.5 Set up forcing SSL

Do you already have SSL for your domain? Bully for you! I've included an option to `setup_openshift` to force traffic to use `https` in a production environment. Just send a second parameter to rake and it'll do the rest.

``` bash
$ rake setup_openshift["ssh://548e1873e0b8cddccf000094@octopress-username.rhcloud.com/~/git/octopress.git/",true]
```

Note that you will still need to change your URL in `_config.yml` if you are using a custom domain.

###<a name="step4"></a>Step 4: Deploy

Now we can just run the `openshift` or `deploy` rake tasks to deploy our app to OpenShift. This task verifies we have the latest version of our OpenShift `master` branch, installs missing gems, copies over your `public` folder, and pushes the application to OpenShift.

``` bash
$ rake deploy
## Deploying branch to OpenShift
## Pulling any updates from OpenShift
cd _deploy
From ssh://octopress-username.rhcloud.com/~/git/octopress
 * branch            master     -> FETCH_HEAD
Already up-to-date.
Fetching gem metadata from https://rubygems.org/...........
Resolving dependencies...
Using rack (1.5.2) 
Using rack-protection (1.5.3) 
Using tilt (1.4.1) 
Using sinatra (1.4.5) 
Using bundler (1.3.5) 
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
cd -
cd _deploy

## Committing: Site updated at 2014-12-14 23:13:54 UTC
[master b17225e] Site updated at 2014-12-14 23:13:54 UTC
 1 file changed, 17 insertions(+)
 create mode 100644 Gemfile.lock

## Pushing generated _deploy website
Counting objects: 9, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (7/7), done.
Writing objects: 100% (9/9), 1.29 KiB | 0 bytes/s, done.
Total 9 (delta 1), reused 0 (delta 0)
remote: Stopping Ruby cartridge
remote: Waiting for stop to finish
remote: Building git ref 'master', commit b17225e
remote: Building Ruby cartridge
remote: bundle install --deployment --path ./app-root/repo/vendor/bundle
remote: Fetching gem metadata from https://rubygems.org/..........
remote: Installing rack (1.5.2) 
remote: Installing rack-protection (1.5.3) 
remote: Installing tilt (1.4.1) 
remote: Installing sinatra (1.4.5) 
remote: Using bundler (1.3.5) 
remote: Your bundle is complete!
remote: It was installed into ./vendor/bundle
remote: Preparing build for deployment
remote: Deployment id is b4ef1149
remote: Activating deployment
remote: Compilation of assets is disabled or assets not detected.
remote: Starting Ruby cartridge
remote: -------------------------
remote: Git Post-Receive Result: success
remote: Activation status: success
remote: Deployment completed with status: success
To ssh://548e1873e0b8cddccf000094@octopress-username.rhcloud.com/~/git/octopress.git/
   a5071bb..b17225e  master -> master

## OpenShift deploy complete
cd -
```

###Fin

That's that. Treat it as any Octopress install. I'll see if the owners of the octopress repository would be interested in pulling in my customizations and update this blog post as necessary.

As for my switch from GH Pages to OpenShift: I've found OpenShift to be just as fast as the GH Pages static server, especially when combined with [Cloudflare's](https://www.cloudflare.com/) CDN and optimization systems. No regrets so far. Drop me a line in the comments if you have a beef with OpenShift or know of any equivalent alternatives.
