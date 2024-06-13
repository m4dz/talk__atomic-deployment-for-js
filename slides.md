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
highlighter: shiki
lineNumbers: true
transition: slide-left

monaco: false
drawings:
  enabled: false

themeConfig:
  aboutme-background: linear-gradient(138deg, rgba(22,28,44,1) 65%, rgba(80,0,72,1) 100%)
  aboutme-color: white

layout: cover
background: https://wallpapercave.com/wp/wp8726590.jpg
---

# Atomic Deployment for **(J|T)S** Hipsters

**by [m4dz](https://m4dz.net)** <br>
![](https://cdn.jsdelivr.net/gh/lyonjs/lyonjs.github.com/graphics/logo/lyon-js.svg) [LyonJS Meetup - Jun. 12, 2024](https://www.meetup.com/lyonjs/events/301132346/)

<style>
p {
  line-height: 1.85 !important;
}
img {
  height: 1.5em;
  display: inline-block;
  margin-top: -0.35em;
  margin-left: -0.15em;
}
</style>

---

<h1 v-click="1">
  <logos-netlify-icon /> <strong>Netlify</strong> killer feature!
</h1>

## tl;dr:

- **Atomic deployment** - Make updates available only when they are complete and totally in place.
- **Immutable deployment** - Guarantee the integrity of previous deploys by insulating them from future actions.

<div v-click="1" class="notes-ln">
  <uil-link-alt /> https://www.netlify.com/blog/2021/02/23/terminology-explained-atomic-and-immutable-deploys/
</div>

---
layout: center
---

# The ~~Hipster~~ **DevOps** Way

<v-clicks>

- Atomic/Immutable Deployment
- Any Hosting Provider
- Easy DX <code v-click="4">git push &lt;origin></code>

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
layout: image-right
image: https://images.pexels.com/photos/1524336/pexels-photo-1524336.jpeg?auto=compressw=960&h=1080&dpr=1
class: icons
---

# Expected **Features**

<v-clicks>

- <uil-file-slash /> Convention-over-Configuration
- <uil-setting /> Extendable 
- <uil-server /> Vanilla Hosting Compatible
- <uil-cloud-computing /> Integrated Micro CI
- <uil-cloud-upload /> Auto-publishable
- <uil-stopwatch-slash /> No (minimal) Down on Deploy
- <uil-bug /> Bug-free

</v-clicks>

---
layout: center
---

# Dive into **Code**

---
layout: image-right
image: https://nextcloud.alwaysdata.org/index.php/s/qjgLNzo3a9GZzMG/download/IMG_7107.jpg
---

# Our Premises

<p class="text-sm">
(for demo's sake)
</p>

<v-clicks>

- Working on the server: Container, VPS…
- Web Hosting Unix (system-)Account <br><span class="text-sm">(e.g `www`)</span>
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

````md magic-move {lines:false}
```sh
$ cd /var
```
```sh
$ ls -l
drwxr-xr-x   2 www:users  www
```
```sh
$ git init --bare repo
Initialized empty Git repository in /var/repo/
```
```sh
$ ls -l
drwxr-xr-x   9 www:users  repo
drwxr-xr-x   2 www:users  www
```
````

---

# Step 2: Create project

**[local]** `~`

````md magic-move {lines:false}
```sh
$ cd Projects
```
```sh
$ npm create vite@latest -- --template vue
```
```sh
$ npm create vite@latest -- --template vue
✔ Project name: … my-app

Scaffolding project in ~/Projects/my-app...

Done. Now run:

  cd my-app
  npm install
  npm run dev
```
```sh
$ cd my-app
```
````

---

# Step 3: Link Repo

**[local]** `~/Projects/my-app`

````md magic-move lines:false}
```sh
$ git init
```
```sh
$ git init
Initialized empty Git repository in ~/Projects/my-app
```
```sh
$ git add .
```
```sh
$ git commit -m "Initial commit"
```
```sh
$ git commit -m "Initial commit"
[main (root-commit) dcef040] Initial commit
 12 files changed, 228 insertions(+)
 ...
```
```sh
$ git remote add
```
```sh
$ git remote add deploy
```
```sh
$ git remote add deploy ssh://www@container:/var/repo
```
```sh
$ git push deploy main
```
```sh
$ git push deploy main
...
To container:/var/repo
 * [new branch]      main -> main
```
````

---
layout: section
---

# Hook!

What happens in the container, stays in the container.

**[SSH]** `/var/repo`

---
layout: iframe-left
url: https://githooks.com/#what-are-git-hooks
---

# <uil-comment-info /> Git Hooks

> These hook scripts are only limited by a developer's imagination.

````md magic-move {lines:false}
```txt
Some example hook scripts include:

- pre-commit:   Check the commit message for
                spelling errors.
- pre-receive:  Enforce project coding standards.
- post-commit:  Email/SMS team members of a new
                commit.
- post-receive: Push the code to production.
```
```txt
- post-receive: Push the code to production.
```
````

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
- Automated process → `#!/usr/bin/env <interpreter>`

</div>

<v-click>

````md magic-move {lines:false}
```sh
$ pwd
/var/repo
```
```sh
$ touch hooks/post-receive
```
```sh
$ chmod +x hooks/post-receive
```
````

</v-click>

---
layout: center
---

![](https://nodejs.org/static/logos/jsIconGreen.svg) *vs* ![](https://deno.news/deno-logo.svg)

<style>
em {
  @apply text-3xl;
  @apply relative;
  @apply inline-block;
  @apply z-1;
  color: var(--slidev-theme-center-headingColor);
}

em::before {
  background: var(--slidev-theme-center-headingBg);
  box-shadow: var(--slidev-theme-header-shadow);
  content: " ";
  display: block;
  position: absolute;
  width: calc(100%);
  height: calc(100%);
  margin-top: 0.1em;
  margin-left: -0.25em;
  padding: 0 1.35rem;
  z-index: -1;
  transform: rotate(-1deg);
}

img {
  display: inline-block;
  height: 128px;
  margin: 2rem;
}
</style>

---
layout: two-cols
---

# ![](https://nodejs.org/static/logos/jsIconGreen.svg) Node.js

- Flexible
- Large ecosystem
- Frontend languages compliant
- Easy to extend


::right::

# ![](https://deno.news/deno-logo.svg) <twemoji-prohibited v-click="1" /> Deno

- Native TS support
- Secure by Design
- Auto-bootstrapped
- Packages from CDNs

<p v-click="1"> 

<uil-image-broken /> No support for [Nodegit](https://www.npmjs.com/package/nodegit) <br><span class="text-sm">(f*ck you Gyp)</span>

</p>

<style>
h1 {
  --img-size: 4rem;
  --gap-size: 0.5rem;

  @apply !flex items-center;
  @apply w-fit;
  margin-left: calc(var(--img-size) + var(--gap-size) * 4);

  .slidev-icon {
    position: absolute;
    margin-left: -calc(var(--img-size) * -1 - var(--gap-size) * 4);
    left: var(--gap-size);
    top: 50%;
    transform: translate(calc(var(--img-size) * -1 - var(--gap-size) * 4), -50%);
    width: var(--img-size);
    height: var(--img-size);
  }

  img {
    width: var(--img-size);
    margin-left: calc(var(--img-size) * -1 - var(--gap-size) * 4);
    margin-right: calc(var(--gap-size) * 4);
  }
}

.slidev-icon {
  @apply mr-1.5;
}
</style>

---

# Initialize

````md magic-move
```js
#!/usr/bin/env node
const readline = require("readline");

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
  terminal: false,
});

rl.on("line", (line) => {
    const [oldRev, newRev, ref] = line.split(" ");
    console.log(`Working on branch ${ref}`);
});
```
```js
#!/usr/bin/env node
const readline = require("readline");

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
  terminal: false,
});

rl.on("line", (line) => {
  (async () => {
    const [oldRev, newRev, ref] = line.split(" ");
    console.log(`Working on branch ${ref}`);
  })();
});
```
````

---

# Setup environment: Git

```js {1-3,9}
const Git = require("nodegit");

const repoPath = path.resolve(".");

/* ... */

(async () => {
  const [oldRev, newRev, ref] = line.split(" ");
  const repo = await Git.Repository.openBare(repoPath);

  console.log(`Working on branch ${ref}`);
})();
```

<div class="notes-ln">
  <uil-link-alt /> https://www.nodegit.org/api/repository
</div>

---

# Setup environment: Tools

```js {1-2,9|10-17}
const Rsync = require("rsync");
const { rimraf } = require("rimraf");

/* ... */

(async () => {
  const [oldRev, newRev, ref] = line.split(" ");
  const repo = await Git.Repository.openBare(repoPath);
  const rsync = new Rsync();
  const props = {
    repo,
    rsync,
    ref,
    rev: newRev,
    refName: ref.substring(11),
    revName: newRev.substring(0, 8),
  };

  console.log(`Working on branch ${ref}`);
})();
```

---

# Work (in) tree

````md magic-move
```js {5}
/* ... */

(async () => {
  /* ... */
  const worktree = await buildWorktree(props);
})();
```
```js {1-2,7|3,6|4-5}
async function buildWorktree({ repo: bareRepo, rev, ref, revName }) {
  const workingDir = path.join(os.tmpdir(), `git.${revName}`);
  const worktree = await Git.Worktree.add(bareRepo, rev, workingDir);
  const repo = await Git.Repository.open(workingDir);
  await repo.checkoutBranch(ref);
  return worktree;
}
```
````

---

# Build

````md magic-move
```js {1,7}
async function buildWorktree({ repo: bareRepo, rev, ref, revName }) {
  const workingDir = path.join(os.tmpdir(), `git.${revName}`);
  const worktree = await Git.Worktree.add(bareRepo, rev, workingDir);
  const repo = await Git.Repository.open(workingDir);
  await repo.checkoutBranch(ref);
  return worktree;
}
```
```js {1,6,8}
async function buildWorktree({ repo: bareRepo, rev, ref, revName }) {
  const workingDir = path.join(os.tmpdir(), `git.${revName}`);
  const worktree = await Git.Worktree.add(bareRepo, rev, workingDir);
  const repo = await Git.Repository.open(workingDir);
  await repo.checkoutBranch(ref);
  await _build(workingDir);
  return worktree;
}
```
```js {1,15|2-3,12-14|4|5-6,8,10|7|9|11}
async function _build(workingDir) {
  try {
    await fs.access(path.resolve(workingDir, "package.json"));
    const npm = `npm --prefix ${workingDir}`;
    try {
      fs.access(path.resolve(workingDir, "package-lock.json"));
      await _run(`${npm} install-clean`);
    } catch {
      await _run(`${npm} install`);
    }
    await _run(`${npm} run build --if-present`);
  } catch {
    /* No package.json found, abort build */
  }
}
```
```js {1,8|2-4|5-6|7}
async function _run(cmd) {
  const child = exec(cmd, (err) => {
    if (err) console.error(err);
  });
  child.stderr.pipe(process.stderr);
  child.stdout.pipe(process.stdout);
  await new Promise((resolve) => child.on("close", resolve));
}

```
````

---

# Deploy

````md magic-move
```js {4}
(async () => {
  /* ... */
  const worktree = await buildWorktree(props);
  const appPath = await deployWorktree({ ...props, worktree });
})();
```
```js {1,7-8|2}
async function deployWorktree({ repo, worktree, rsync, rev, refName, revName }) {
  const deployRoot = await setupDeployDir(arguments[0]);
  const appPath = path.resolve(deployRoot, refName);

  /* Deploy logic here... */
  
  return appPath;
}
```
```js {1,5-6|2}
async function setupDeployDir({ repo, rsync, ref, rev }) {
  const deployRoot = await _mkDeployDir(arguments[0]);
  const deployDir = path.resolve(deployRoot, rev);

  return deployRoot;
}
```
````

---
layout: image-right
image: https://i.giphy.com/l4Jz3a8jO92crUlWM.webp
---

# <uil-cog /> Extend Convention

**[SSH]** `/var/repo`

````md magic-move {lines:false}
```sh
$ git config --local --list
```
```sh
$ git config --local --list
core.repositoryformatversion=0
core.filemode=true
core.bare=true
core.ignorecase=true
core.precomposeunicode=true
```
```sh
$ git config --local dirs.deploy
```
```sh
$ git config --local dirs.deploy /var/www/prod
```
```sh
$ git config --local --get dirs.build
/var/www/prod
```
````

---

# Setup Deploy Env

````md magic-move
```js {2}
async function setupDeployDir({ repo, rsync, ref, rev }) {
  const deployRoot = await _mkDeployDir(arguments[0]);
  const deployDir = path.resolve(deployRoot, rev);

  return deployRoot;
}
```
```js {1,14|2,4,6|2,4,8|10|12-13}
async function _mkDeployDir({ repo }) {
  const repoConfig = await repo.config();

  let deployDir;
  try {
    deployDir = (await repoConfig.getEntry("dirs.deploy")).value();
  } catch {
    deployDir = path.resolve(repoPath, "../www");
  }
  await fs.mkdir(deployDir, { recursive: true });

  console.log(`Deploy branches to ${deployDir}`);
  return deployDir;
}
```
```js
async function setupDeployDir({ repo, rsync, ref, rev }) {
  const deployRoot = await _mkDeployDir(arguments[0]);
  const deployDir = path.resolve(deployRoot, rev);

  return deployRoot;
}
```
````

---

# <uil-comment-info /> Git Common Ancestor

````md magic-move {lines:false}
```md
`git merge-base A B`

     o---o---o---B
    /
---X---1---o---o---o---A
```
```md
`git merge-base A C`

       o---o---o---o---C
      /
     /   o---o---o---B
    /   /
---X---o---o---o---o---A
```
```md
`git merge-base A B C`

       o---o---o---o---C
      /
     /   o---o---o---B
    /   /
---o---X---o---o---o---A
```
```md
`git merge-base A (B C) M`

       o---o---o---o---o
      /                 \
     /   o---o---o---o---M
    /   /
---o---X---o---o---o---A
```
````

<v-clicks class="mt-4xl">

1. Find all branches tips
2. Get merge base commits
2. Iterate over history revisions list from current branch tip
3. If a commit is a merge base: <twemoji-party-popper />

</v-clicks>

<div class="notes-ln">
  <uil-link-alt /> https://git-scm.com/docs/git-merge-base#_discussion
</div>

---

# Find merge-base commit

````md magic-move
```js {1,6}
async function setupDeployDir({ repo, rsync, ref, rev }) {
  const deployRoot = await _mkDeployDir(arguments[0]);
  const deployDir = path.resolve(deployRoot, rev);

  return deployRoot;
}
```
```js {4}
async function setupDeployDir({ repo, rsync, ref, rev }) {
  /* ... */

  const bases = await findMergeBases(arguments[0]);

}
```
```js {1,17|2,8|2-8|10|11-12,15-16|13|14}
async function findMergeBases({ repo, ref }) {
  const refs = (await repo.getReferences()).filter((ref) => {
    try {
      Git.Oid.fromString(ref.name().replace(/^refs\/heads\//, ""));
    } catch {
      return true;
    }
  });

  const refCommit = await repo.getBranchCommit(ref);
  return await Promise.all(
    refs.map(async (tip) => {
      const tipCommit = await repo.getReferenceCommit(tip);
      return await Git.Merge.base(repo, tipCommit.id(), refCommit.id());
    }),
  );
}
```
```js
async function setupDeployDir({ repo, rsync, ref, rev }) {
  /* ... */

  const bases = await findMergeBases(arguments[0]);

}
```
```js
async function setupDeployDir({ repo, rsync, ref, rev }) {
  /* ... */

  const bases = await findMergeBases(arguments[0]);
  const refCommit = await repo.getBranchCommit(ref);
  const revisions = refCommit.history();

}
```
```js {5,20|6,16|7-9|11-12|18|19}
async function setupDeployDir({ repo, rsync, ref, rev }) {
  /* ... */
  const revisions = refCommit.history();

  const baseCommit = await new Promise((resolve) => {
    revisions.on("commit", async (commit) => {
      const isBase = bases.find((baseOid) => baseOid.equal(commit.id()));

      if (!isBase) return;
      try {
        await fs.access(path.resolve(deployRoot, commit.id().tostrS()));
        resolve(commit.id());
      } catch {
        /* Deploy base doesn't exist for this commit, continue... */
      }
    });

    revisions.on("end", () => resolve(false));
    revisions.start();
  });
}
```
````

---

# <uil-comment-info /> UNIX Hardlinks

> A hard link to a file points to the inode of the file instead of pointing to the file itself.
> This way the hard link gets all the attributes of the original file and points to the same data block
> as the original file.

<div class="notes-ln">
  <uil-link-alt /> https://linuxhandbook.com/hard-link/
</div>

---

# Deploy

````md magic-move
```js
async function setupDeployDir({ repo, rsync, ref, rev }) {
  /* ... */

  const baseCommit = await new Promise((resolve) => {
    /* ... */
  });

}
```
```js {7-13|12}
async function setupDeployDir({ repo, rsync, ref, rev }) {
  /* ... */

  const baseCommit = await new Promise((resolve) => {
    /* ... */
  });

  if (baseCommit) {
    console.log(
      `Found deploy-base commit ${baseCommit.tostrS().substring(0, 8)}`,
    );
    rsync.set("link-dest", path.resolve(deployRoot, baseCommit.tostrS()));
  }
}
```
```js {2,13}
async function setupDeployDir({ repo, rsync, ref, rev }) {
  const deployDir = path.resolve(deployRoot, rev);

  /* ... */

  if (baseCommit) {
    console.log(
      `Found deploy-base commit ${baseCommit.tostrS().substring(0, 8)}`,
    );
    rsync.set("link-dest", path.resolve(deployRoot, baseCommit.tostrS()));
  }

  rsync.destination(deployDir + path.sep);
  return deployRoot;
}
```
````

---

# Publish !

````md magic-move
```js {1,8}
async function deployWorktree({ repo, worktree, rsync, rev, refName, revName }) {
  const deployRoot = await setupDeployDir(arguments[0]);
  const appPath = path.resolve(deployRoot, refName);

  /* Deploy logic here... */

  return appPath;
}
```
```js {4,9|5,7-10,12}
async function deployWorktree({ repo, worktree, rsync, rev, refName, revName }) {
  /* ... */
  
  const repoConfig = await repo.config();
  let buildDir;
  try {
    buildDir = path.resolve(
      worktree.path(),
      (await repoConfig.getEntry("dirs.build")).value(),
    );
  } catch {
    buildDir = path.resolve(worktree.path());
  }
  
  return appPath;
}
```
```js {4-9,13|12-14,17}
async function deployWorktree({ repo, worktree, rsync, rev, refName, revName }) {
  /* ... */

  rsync
    .set("archive")
    .set("no-times")
    .set("checksum")
    .exclude([".git"])
    .source(buildDir + path.sep);

  try {
    await new Promise((resolve, reject) =>
      rsync.execute((err, code) => (err ? reject(err) : resolve(code))),
    );
  } catch (err) {
    console.log(err);
    return false;
  }

  return appPath;
}
```
```js {4|6|10-11|13}
async function deployWorktree({ repo, worktree, rsync, rev, refName, revName }) {
  /* ... */

  console.log(`Linking ${refName} -> ${revName}`);
  try {
    await fs.unlink(appPath);
  } catch {
    /* No previous version to unlink, continue... */
  }
  await fs.mkdir(path.dirname(appPath), { recursive: true });
  await fs.symlink(rsync.destination(), appPath);

  return appPath;
}
```
````


---

# Clean

````md magic-move
```js {1,7}
(async () => {
  /* ... */

  const worktree = await buildWorktree(props);
  const appPath = await deployWorktree({ ...props, worktree });

})();
```
```js {5,8-10}
(async () => {
  /* ... */

  let worktree, appPath;
  try {
    worktree = await buildWorktree(props);
    appPath = await deployWorktree({ ...props, worktree });
  } finally {
    await clean({ ...props, worktree });
  }

})();
```
```js
async function clean({ worktree }) {
  await rimraf(worktree.path());
  worktree.prune();
}

```
````

---

# SIGHUP

<p v-click="3" class="text-sm">
(for demo's sake)
</p>

````md magic-move
```js {7}
(async () => {
  /* ... */

  let worktree, appPath;
  try {
    worktree = await buildWorktree(props);
    appPath = await deployWorktree({ ...props, worktree });
  } finally {
    await clean({ ...props, worktree });
  }

})();
```
```js {9-11}
(async () => {
  /* ... */

  let worktree, appPath;
  try {
    worktree = await buildWorktree(props);
    appPath = await deployWorktree({ ...props, worktree });

    if (appPath) {
      await triggerRestart({ ...props, appPath });
    }
  } finally {
    await clean({ ...props, worktree });
  }

})();
```
```js {1,13|2|3-12}
async function triggerRestart({ appPath }) {
  const port = Math.floor(Math.random() * 1000) + 8000;
  const ip = await new Promise((resolve) => {
    const nets = os.networkInterfaces();
    for (const inet in nets) {
      for (const net of nets[inet]) {
        if (["IPv4", 4].includes(net.family) && !net.internal) {
          resolve(net.address);
        }
      }
    }
  });
}
```
```js {4|5-7}
async function triggerRestart({ appPath }) {
  /* ... */
  
  const psList = (await import("ps-list")).default;
  (await psList())
    .filter((process) => process.cmd.includes(appPath))
    .map(({ pid }) => process.kill(pid));
}
```
```js {4-9}
async function triggerRestart({ appPath }) {
  /* ... */
  
  console.log(`Restarting app -> http://${ip}:${port}`);
  const server = spawn(
    "npx",
    ["-y", "serve", "-nSL", "-l", `tcp://${ip}:${port}`, appPath],
    { detached: true, stdio: "ignore" },
  );
}
```
```js {8,10}
async function triggerRestart({ appPath }) {
  /* ... */
  
  console.log(`Restarting app -> http://${ip}:${port}`);
  const server = spawn(
    "npx",
    ["-y", "serve", "-nSL", "-l", `tcp://${ip}:${port}`, appPath],
    { detached: true, stdio: "ignore" },
  );
  server.unref();
}
```
````

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

# **Show** Time !

---
layout: center
---

# <twemoji-party-popper class="animate-bounce" /> <span v-click.hide="1">In code <b><L240</b>!</span><span v-click="1">In code <b><L60</b><sup>*</sup>!</span>

<p v-click="1"><sup>*</sup> in Bash</p>

<style>
h1 {
  @apply !grid grid-rows-1 gap-3;
  grid-template-columns: auto 1fr;

  &::before {
    @apply !m-0;
  }

  span {
    @apply: relative col-start-2 row-start-1;
  }
}

p {
  @apply w-full text-right !text-sm italic;
}
</style>

---
layout: section
---

# **Real** Use-cases

---
layout: image-right
image: https://images.pexels.com/photos/2111759/pexels-photo-2111759.jpeg?auto=compress&w=960&h=1080&dpr=1
---

# API Calls

- Create/Update/Delete Site
- DB Actions: migrate…
- Cache invalidation
- Site restart
- _etc_

---
layout: center
---

# WebHooks?

<p>

<uil-rocket /> Automating push from `origin` to `deploy`

</p>

<div class="notes-ln">
  <uil-link-alt /> https://blog.alwaysdata.com/2023/02/01/devops-react-we-can-be-heroes-just-for-one-app/
</div>

---
layout: image-right
image: https://images.pexels.com/photos/994747/pexels-photo-994747.jpeg?auto=compress&w=960&h=1080&dpr=1
class: icons
---

# <uil-code-branch /> Tips

- <uil-lightbulb-alt /> Keep It Simple 'n Stupid
- <uil-cog /> Use `git config` a lot
- <uil-megaphone /> Abstract API calls
- <uil-expand-alt /> Expand your build system

---
layout: cover
background: https://images.pexels.com/photos/97824/pexels-photo-97824.jpeg?auto=compress&w=1920&h=1080&dpr=1
---

# Questions ?

<style>
h1 {
  --slidev-theme-cover-headingColor: var(--slidev-theme-accents-vulcan);
  --slidev-theme-cover-headingBg: var(--slidev-theme-accents-lightblue);

  @apply !text-4xl;
}
</style>

---
layout: center
---

# **Thanks**!
