---
title: Atomic Deployment for JS/TS Hipsters
info: |
  Deploying an app is all but an easy process. You will encounter a lot of glitches and pain points to solve to have it
  working properly. The worst is: that now that you can deploy your app in production, how can't you also deploy all
  branches in the project to get access to live previews? And be able to do a fast-revert on-demand?

  Fortunately, the classic DevOps toolkit has all you need to achieve it without compromising your mental health.
  By expertly mixing Git, Unix tools, and API calls, and orchestrating all of them with JavaScript, you'll master the
  secret of safe atomic deployments.

  No more need to rely on commercial services: become the perfect tool master and netlifize your app right at home!
author: m4dz <m@m4dz.net>

theme: the-unnamed
highlighter: shikiji
lineNumbers: false
drawings:
  enabled: false
transition: slide-left

themeConfig:
  aboutme-background: linear-gradient(138deg, rgba(22,28,44,1) 65%, rgba(80,0,72,1) 100%)
  aboutme-color: white
layout: cover
background: https://source.unsplash.com/vk074XaA70w/1920x1080
---

# Atomic Deployment for **(J|T)S** Hipsters

**by [m4dz](https://m4dz.net)** <br>
![](https://www.devopsjsconf.com/img/favicon.png) DevOps.js Conference - Feb. 15-16, 2024

<style>
img {
  height: 0.75em;
  display: inline-block;
  margin-top: -0.2em;
}
</style>

---

<h1 v-click>
  <logos-netlify-icon /> <strong>Netlify</strong> killer feature!
</h1>

## tl;dr:

- **Atomic deployment** - Make updates available only when they are complete and totally in place.
- **Immutable deployment** - Guarantee the integrity of previous deploys by insulating them from future actions.

<div class="flex items-center gap-1 mt-6xl">
  <uil-link-alt /> https://www.netlify.com/blog/2021/02/23/terminology-explained-atomic-and-immutable-deploys/
</div>

---
layout: image-right
image: https://source.unsplash.com/ozSc1O0zGkM/1920x1080
---

# The ~~Hipster~~ **DevOps** Way

- Atomic/Immutable Deployment
- Any Hosting Provider
- Simple DX

---
layout: image-right
image: https://source.unsplash.com/9DRX_cW48RQ/1920x1080
---

# Expected **Features**

<v-clicks>

- <uil-setting /> Configurable
- <uil-server /> Vanilla Hosting Compatible
- <uil-cloud-computing /> Integrated Micro CI
- <uil-stopwatch-slash /> No (minimal) Down on Deploy
- <uil-cloud-upload /> Auto-publishable

</v-clicks>

---
layout: about-me

helloMsg: Hi Folks!
name: I'm m4dz
imageSrc: /m4dz.jpg
job: DX Engineer @ alwaysdata
line1:
line2:
social1: m4dz.net
social2: www.alwaysdata.com
social3:
---

---
layout: center
---

# Dive into **Code**

---
layout: image-right
image: https://source.unsplash.com/c3tNiAb098I/1920x1080
---

# Our Premises

<p class="text-sm">
(for this talk)
</p>

<v-clicks>

- Working on the server: Container, VPS…
- Web Hosting Unix (system-)Account: `www`
- SSH Remote Access
- No WebHooks<br><span class="text-sm">(let's talk about'em in conclusion)</span>

</v-clicks>

---
layout: section
---

# Boostrapping

Git basics

---

# Step 1: Init

**[SSH]** `www@container`

```sh {1|3-4|6-7|all} {lines:false}
$ cd /var

$ ls -l
drwxr-xr-x   2 www  users  www

$ git init --bare repo
Initialized empty Git repository in /var/repo/

$ ls -l
drwxr-xr-x   9 www  users  repo
drwxr-xr-x   2 www  users  www
```

---

# Step 2: Create project

**[local]** `~`

```sh {1|3|3-12|all} {lines:false}
$ cd Projects

$ npm create vite@latest -- --template vue
✔ Project name: … my-app

Scaffolding project in ~/Projects/my-app...

Done. Now run:

  cd my-app
  npm install
  npm run dev
  
$ cd my-app
```

---

# Step 3: Link Repo

**[local]** `~/Projects/my-app`

```sh {1-2|3-8|11|13-16} {lines:false}
$ git init
Initialized empty Git repository in ~/Projects/my-app

$ git add .

$ git commit -m "Initial commit"
[main (root-commit) dcef040] Initial commit
 12 files changed, 228 insertions(+)
 ...

$ git remote add deploy ssh://www@container:/var/repo

$ git push deploy main
...
To container:/var/repo
 * [new branch]      main -> main
```

---

# Step 4: Configure

**[SSH]** `/var/repo`

```sh {1-6|8|8-11} {lines:false}
$ git config --local --list
core.repositoryformatversion=0
core.filemode=true
core.bare=true
core.ignorecase=true
core.precomposeunicode=true

$ git config --local dirs.build dist

$ git config --local --get dirs.build
dist
```

---
layout: section
---

# Hook!

What happens in the container, stays in the container.

**[SSH]** `/var/repo`

---
layout: iframe-left
url: https://githooks.com/
---

# Git Hooks

> These hook scripts are only limited by a developer's imagination.

```txt {all|6}
Some example hook scripts include:

- pre-commit: Check the commit message for spelling errors.
- pre-receive: Enforce project coding standards.
- post-commit: Email/SMS team members of a new commit.
- post-receive: Push the code to production.
```

<style>
blockquote {
  @apply mb-4xl
}
</style>

---

# `post-receive` hook

> This hook is invoked by `git-receive-pack[1]` when it reacts to git push and updates reference(s) in its repository.
> It executes on the remote repository once after all the refs have been updated.

<div class="my-4xl" v-click>

- Reads **lines** from `/dev/stdin`
- Automated process → `/bin/bash`

</div>

<v-click>

```sh {lines:false}
$ pwd
/var/repo

$ touch hooks/post-receive

$ chmod +x hooks/post-receive
```

</v-click>

---

# Initialize

```bash
#!/bin/bash

while read oldrev newrev ref
do
	echo "Working on branch ${ref}"
done
```

---

# Setup environment

```bash {3|4|6-7|8|3-8}
#!/bin/bash

GIT_DIR=`dirname $(cd -- ${0%/*} >/dev/null 2>&1; pwd -P)`
pushd $GIT_DIR >/dev/null

DEPLOY_DIR=`git config --local dirs.deploy`
echo "Deploy branches to ${DEPLOY_DIR:=$(dirname $GIT_DIR)/www}"
mkdir -p $DEPLOY_DIR

while read oldrev newrev ref
  ...
```

---

# Setup branch

```bash {6-9|8|9|6-9}
...
while read oldrev newrev ref
do
	echo "Working on branch ${ref}"

	REF_NAME=${ref:10}                 # remove `refs/heads`
	REV_NAME=${newrev:0:8}             # 8 chars commit ID
	WORK_DIR=`mktemp -d`
	PROD_LNK=${DEPLOY_DIR}${REF_NAME}  # Human-readable path
...
```

---

# Checkout the Worktree

```bash {7-8}
while read oldrev newrev ref
do
  ...
	WORK_DIR=`mktemp -d`
  ...
  
	git worktree add $WORK_DIR $ref
	pushd $WORK_DIR >/dev/null
	...
```

---

# Build

```bash {4|5|7|8|4-9}
while read oldrev newrev ref
do
  ...
	BUILD_DIR="/$(git config --local dirs.build)"
	if test -f "./package.json"
	then
		test -f "./package-lock.json" && npm install-clean || npm install
		npm run build --if-present
	fi
```

---

# <uil-comment-info /> UNIX Hardlinks

> A hard link to a file points to the inode of the file instead of pointing to the file itself.
> This way the hard link gets all the attributes of the original file and points to the same data block
> as the original file.

<div class="flex items-center gap-1 mt-6xl">
  <uil-link-alt /> https://linuxhandbook.com/hard-link/
</div>

---

# <uil-comment-info /> Git Common Ancestor

```txt
         o---B2 (feat/B2)
        /
---o---o---B1--o---o---o---B (main)
    \                   \
     B0                  C0'--C1'--C' (feat/C) (rebased)
      \
       C0---C1---C (~~feat/C~~) (old)
             \
              D0---D1 (fix/D)
```

<v-clicks class="mt-4xl">

1. Get the revisions list
2. Iterate to find a valid ancestor
3. Confirm this commit is a tip of a branch

</v-clicks>

---

# Find the Common Ancestor

```bash {6|7-8,14|9|11|13|16|18-19|6-20}
while read oldrev newrev ref
do
  ...
	pushd $WORK_DIR >/dev/null

	revlist=(`git rev-list $ref main`)
	for rev in ${revlist[@]:1}
	do
		if git merge-base --is-ancestor $rev $ref
		then
			grep -r $rev refs/heads/* >/dev/null && test -d ${DEPLOY_DIR}/${rev} && break
		fi
		unset rev
	done
  
	if [[ -n $rev ]]
	then
		echo "Found deploy-base commit ${rev:0:8}"
		rsync_opts="--link-dest ${DEPLOY_DIR}/${rev}"
	fi
  ...
```

---

# Deploy

```bash {4-5}
while read oldrev newrev ref
do
  ...
	echo "Deploying ${SRC_DIR:=${WORK_DIR}${BUILD_DIR%/}} -> ${DST_DIR:=${DEPLOY_DIR}/${newrev}}"
	rsync -a --no-times --exclude '.git' --checksum ${rsync_opts[@]} ${SRC_DIR}/ ${DST_DIR}/
```

---

# Clean

```bash {4|5|4-5}
while read oldrev newrev ref
do
  ...
	popd >/dev/null
	git worktree remove $WORK_DIR
```

---

# Publish

```bash {4|5|6|4-6|9|10|8,11|8-11|4-11}
while read oldrev newrev ref
do
  ...
	echo "Linking ${REF_NAME} -> ${REV_NAME}"
	mkdir -p ${PROD_LNK%/*}
	ln -sfn ${DST_DIR%/} $PROD_LNK

	port=$(expr 8000 + $RANDOM % 100)
	echo "Restarting app -> http://localhost:${port}"
	pkill -lf $PROD_LNK >/dev/null 2>&1
	(nohup npx -y serve -nSL -p $port "${PROD_LNK}" > /dev/null 2>&1 & disown)
```

---
layout: section
---

# Show Me!

**[local]** `~/Projects/my-app`

---

# Update `my-app`

```sh {1-2|4-5|7|9-11}
$ pwd
~/Projects/my-app

$ git checkout -b feat/update-name
Switched to a new branch 'feat/update-name'

$ $EDITOR src/App.vue

$ git commit -am "Update name"
[feat/update-name 51f0790] Update name
 1 file changed, 1 insertion(+), 1 deletion(-)
```

---

# Ship It!

```bash {1|3|4|5-6|8|10-11,13|14-15|16|17|1-19}
$ git push deploy feat/update-name
...
remote: Deploy branches to /var/www
remote: Working on branch refs/heads/feat/update-name
remote: Preparing worktree (detached HEAD 51f0790)
remote: HEAD is now at 51f0790 Update name
remote: 
remote: added 28 packages, and audited 29 packages in 558ms
...
remote: > vite build
remote: vite v5.0.11 building for production...
...
remote: ✓ built in 244ms
remote: Found deploy-base commit 7f039e99
remote: Deploying /tmp/tmp.XVla4Uaaj6/dist -> /var/www/51f07903f8b57cd56b7f5bbbeb6d08d9d3af42f4
remote: Linking /var/www/feat/update-name -> 51f07903
remote: Restarting app -> http://localhost:8022
To container:/var/repo
   322902f..51f0790  feat/update-name -> feat/update-name
```

---
layout: center
---

# <twemoji-party-popper class="animate-bounce" /> In code <strong><L60</strong>!

---
layout: center
---

# **Real** Use-cases

---
layout: image-right
image: https://source.unsplash.com/nlKFtadpueY/1080x1920
---

# API Calls

- Create/Update/Delete Site
- DB Actions: migrate…
- Cache invalidation
- Site restart
- _etc_

---

# WebHooks?

Automating push from `origin` to `deploy`

<div class="flex items-center gap-1 mt-6xl">
  <uil-link-alt /> https://blog.alwaysdata.com/2023/02/01/devops-react-we-can-be-heroes-just-for-one-app/
</div>

---
layout: image-right
image: https://source.unsplash.com/_kdTyfnUFAc/1080x1920
---

# Tips

- Keep It Simple 'n Stupid
- Use `git config` a lot
- Abstract API calls
- Use Nodejs / Deno as a build system

---
layout: center
---

# **Thanks**!
