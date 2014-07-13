---
layout: post
title: "Vagrant and Dokku on OSX"
date: 2014-07-13 16:57:00 +0100
comments: true
categories: Vagrant Docker Dokku
---

So I decided that I should probably look into [Vagrant](http://www.vagrantup.com) as it would let me easily provision different development environments [lickity split](http://www.urbandictionary.com/define.php?term=Lickity+split) via [Virtualbox](https://www.virtualbox.org/wiki/Downloads)

Afterwards I decided to look into [Docker](http://www.docker.com) as it was now > v1.0.0 which means it's ready for production, right?
More on that another time.
	
During my venture in to the world of Docker I discovered [Dokku](https://github.com/progrium/dokku).  My very own private Heroku.

Now this was getting interesting...
<!-- more -->
[Digital Ocean](https://www.digitalocean.com) has a Ubuntu + Dokku ready image that can be deployed and tried out for [pennies](https://www.digitalocean.com/pricing/).  But what if I wanted to work on my local development machine and have it be the same as on Digital Ocean?

__Install Dokku__

```
> git clone git@github.com:progrium/dokku.git
> cd dokku
```

__Start the VM__

_This assumes you have already installed Vagrant and Virtualbox_
```
> vagrant up
```
Vagrant will automagically provision and start the VM with Docker, Git, Nginx and anything else Dokku needs to work...
This means from scratch downloading and installing the packages in the VM and even the VM itself if you didn't already have it.  

Yes, that could take a little while (after downloading the VM image Vagrant will cache it locally so it'll be faster next time around).

__Add some entries to /etc/hosts__

The running Dokku server will be on IP address 10.0.0.2, with a hostname of dokku.me

In order to make it easier to deploy and run applications we'll need to add some entries to your /etc/hosts file - one for the Dokku server and another for a sample app, which we'll use to verify it's all working as expected.

```
10.0.0.2 dokku.me
10.0.0.2 sample-node-app.dokku.me
```

This will also allow us to access deployed applications via URLs like `http://sample-node-app.dokku.me` instead of IP address and port number.

__Add vagrant SSH key to the running Dokku server__

I think this should be done by the the Dokku/Vagrant script but for some reason it didn't on my machine so I had to do this manually.

```
cat ~/.ssh/id_rsa.pub | ssh -o "StrictHostKeyChecking=no" -i ~/.vagrant.d/insecure_private_key vagrant@dokku.me "sudo sshcommand acl-add dokku vagrant"
```

__Add your SSH key to the running Dokku server__

In order to deploy your code to the Dokku server it needs your SSH key.

```
> cat ~/.ssh/id_rsa.pub | ssh dokku@dokku.me "sudo sshcommand acl-add dokku jeremychu"
```
What's happening here?  Let's break it down:

1. Your public SSH key file is being piped...  `cat ~/.ssh/id_dsa.pub |`
1. to a shell session at dokku.me with user dokku... `ssh dokku@dokku.me`
(That entry in your /etc/hosts file made it easier as you can use `dokku.me` instead of `10.0.0.2`)
1. Inside the Dokku shell session some commands are executed, which will receive your SSH key... `"sudo sshcommand acl-add dokku jeremychu”`

This will Add a named SSH key `jeremychu` to the user `dokku`.  See the [sshcommand on Github](https://github.com/progrium/sshcommand) for more info.

If it all went well you should see something like the following, say yes and proceed.
```
The authenticity of host 'dokku.me (10.0.0.2)' can't be established.
RSA key fingerprint is 92:51:54:a9:c7:a8:97:9f:37:e6:ed:2a:ad:6a:bc:91.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'dokku.me,10.0.0.2' (RSA) to the list of known hosts.
Permission denied (publickey).
```

If it works you should be able to SSH into dokku.me and call the Dokku `version` command:
```
> ssh dokku@dokkue.me version
v0.2.3-7-g4f5fc58
```
__Grab a sample app from Github__

```
> git clone https://github.com/jeremychu/sample-node-app.git
> cd sample-node-app
```

__Add the local Dokku server as a Git Remote__

```
> git remote add local-dev dokku@dokku.me:sample-node-app
```

Now deploy the app (I find this part rather special):

```
> git push local-dev master
```


If it all worked you should see something similar to the following (trimmed for readability):

```
$ git push local-dev master
...
Counting objects: 21, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (17/17), done.
Writing objects: 100% (21/21), 3.02 KiB | 0 bytes/s, done.
Total 21 (delta 0), reused 0 (delta 0)
-----> Cleaning up ...
-----> Building sample-node-app ...
...
       Node.js app detected

       PRO TIP: Specify a node version in package.json
       See https://devcenter.heroku.com/articles/nodejs-support

-----> Defaulting to latest stable node: 0.10.29
-----> Downloading and installing node
-----> Writing a custom .npmrc to circumvent npm bugs
-----> Installing dependencies
       debug@0.7.4 node_modules/debug
       
       static-favicon@1.0.2 node_modules/static-favicon
       
...
-----> Caching node_modules directory for future builds
-----> Cleaning up node-gyp and npm artifacts
-----> Building runtime environment
-----> Discovering process types
       Procfile declares types -> web
-----> Releasing sample-node-app ...
-----> Deploying sample-node-app ...
=====> Application deployed:
       http://sample-node-app.dokku.me

To dokku@dokku.me:sample-node-app
 * [new branch]      master -> master
```

Now point your browser at [sample-node-app.dokku.me](http://sample-node-app.dokku.me)

<img src="{{root_url}}/images/sample-node-app.png"/>

__Delete the sample app__

When you're finished you can delete the app by calling:

```
> ssh dokku@dokku.me delete sample-node-app
```

Note: This is run on the host OS and not inside the Dokku VM (so on my Macbook Pro I would just run this inside a terminal window).
