I recently became one of the organizers of the [OKCsharp][] user group in Oklahoma City. They use [Hexo.io][] to publish the user group static website onto Github. Hexo is not a CMS, just a Node.js package that generates a static site and will update the appropriate `gh-pages` branch for Github to host.

#### Hexo.io
I'm not going into the use of Hexo; their website has very good documentation. Pay special attention to the [Hexo deployment][] section. That's what I will be modifying a bit here in this post.

I handle a large part of the automated development pipeline process where I work fulltime so the idea of doing a git pull and

```
hexo deploy --generate
```

to deploy the website, although short and simple (much nicer than others I've used) is still so **manual**. It just feels...wasteful.

So I fixed it with continuous integration.

#### Using CircleCI

I ended up using [CircleCI][]. It's fast. And I mean super fast. Much faster than TravisCI (and cheaper if you end up needing more). CircleCI will also build pull requests from forks which is awesome (so will TravisCI). They use a `circle.yml` file to configure the build and that's easy enough.

```
deployment:
  production:
    branch: master
    commands:
      - git config --global user.name "CircleCI"
      - git config --global user.email "noone@okcsharp.net"
      - sed -i'' "s~https://github.com/techlahoma/okcsharp-website.git~https://${GH_TOKEN}:x-oauth-basic@github.com/techlahoma/okcsharp-website.git~" _config.yml
      - rm -rf .deploy_git/
      - hexo clean
      - hexo generate
      - hexo deploy
```

In there `production` is just a name for the set of commands for the deployment; it has no other effect. The `branch` field tells CircleCI which branch this deployment will run for. In this case I only deploy the website when a commit to `master` is made. So pull requests will not trigger this. This lets us review a PR and then merge it to master when we want to. Then CircleCI handles the rest.

The `commands` simply contain shell commands needed to deploy the hexo website to Github. That `sed` command is used to modify the original hexo `_config.yml` file to allow it to deploy to Github with the appropriate token having commit permissions.

That `${GH_TOKEN}` is an environment variable set in the CircleCI web ui. You can generate that token in Github by going to your Github settings then [Personal Access Tokens][]. Put the token into the CircleCI project settings [environment variables][].

##### Environment Variable Security
A very important note about environment variables in CircleCI; environment variables specified in the `circle.yml` are available in fork builds. Environment variables specified in the CircleCI web UI are **not** available during fork builds. This is a security precaution. Anyone randomly could fork your public repo, modify the commands and have it export your sensitive environment variables to another server.

After updating the hexo `_config.yml` file to be able to commit to Github appropriately the rest of the commands just run the appropriate hexo commands to generate and deploy the website.

The end result is someone can send us a pull request containing a new post. We look it over and decide to merge it using the Github web UI. CircleCI is notified of that commit and takes care of the rest. In short:

#### One click hexo deployments

Oh, that sounds like a good name for a talk. But before I get to that I need to mention that I didn't stop there. In my next post I'll describe how I generated screenshots of the PR and even put those screenshots onto the PR as a comment.


[OKCsharp]:http://okcsharp.net
[Hexo.io]:http://hexo.io
[CircleCI]:https://circleci.com
[Hexo deployment]:https://hexo.io/docs/deployment.html
[Personal Access Tokens]:https://github.com/settings/tokens
[environment variables]:https://circleci.com/docs/environment-variables#custom