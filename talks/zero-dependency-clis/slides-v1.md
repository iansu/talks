---
title: Zero Dependency CLIs with Node.js
theme: slidev-theme-iansu
---

# Zero Dependency CLIs

with Node.js

<!--
Hi! Today I'm going to be talking about building zero dependency CLI apps with Node.js.
-->

---

# Who am I?

<h2 v-click>Ian Sutherland</h2>
<h4 v-click class="-mt-2">Head of Developer Experience (DX) at Neo Financial</h4>
<h4 v-click class="-mt-2">Node.js Contributor, Tooling Working Group</h4>
<h4 v-click class="-mt-2">Maintainer of Create React App</h4>

<div v-click class="mt-10 inline-flex">
  <simple-icons-twitter class="mr-4 text-3xl text-black" /> <simple-icons-github class="mr-5 text-3xl text-black" /> <span class="text-3xl text-black">@iansu</span>
</div>

<!--
First let me quickly introduce myself.
My name is Ian Sutherland and I'm the head of Developer Experience at Neo Financial, which is a Canadian fintech startup.
I'm also a Node.js contributor where I mostly work on the Tooling Working Group which I'll be covering more in this talk.
I'm also a maintainer of Create React App.
You can find me on Twitter and GitHub @iansu.
-->

---

# Zero Dependency CLIs

<v-clicks>

- What are dependencies and why are we trying to avoid them?
- What is a CLI app and why are we talking about them?
- What new Node.js features can help us write CLI apps?
- What new Node.js features might be coming next?

</v-clicks>

<!--
Back to zero dependency CLIs
Here's an overview of some of the things we're going to cover
-->

---

# Dependencies

What are they and why do we need them?

<v-clicks>

- Some other piece of code, like a library, that your program needs to run
- In Node.js dependencies commonly come from, and are installed with, npm
- Other users and contributors will also need to install the same dependencies
- Dependencies need to be installed at build time and deployed with your code
- Node.js does not include a big "standard library" and relies on npm packages instead

</v-clicks>

<!--
Other languages have standard libraries of varying sizes.
Node has a particularly small standard library and an early architecture decision was to have a "small core".
This made packages on npm the choice for any additional functionality and is why we see packages like `isnumber` with millions of weekly downloads
-->

---

# Dependencies

Are dependencies bad?

<v-clicks>

- Not necessarily
- Dependencies do introduce some overhead in the development and deploy process
  - Users need to install the dependencies to run and develop on a project
  - Dependencies need to be installed or bundled at build time
- If you're building a complex app it probably already has a build and deploy pipeline and many dependencies so adding more dependencies probably doesn't make much of a difference
- For smaller apps and libraries, like CLIs, not having an install or build step makes setup, contributing and distribution easier

</v-clicks>

<!--

-->

---

# CLI Apps

What are they and why do I keep talking about them?

<v-clicks>

- CLI stands for Command Line Interface
  - Basically a fancy way to describe a script that takes some input and generates output
- These kinds of apps run in a terminal
- Commonly used for dev tools, build scripts and process automation
- Range from simple `cat` to complex `git`

</v-clicks>

<!--

-->

---

# CLI Apps

<div class="-mt-10 w-full text-center">
  <img src="ls.svg" class="mx-auto" />
</div>

---

# CLI Apps

<div class="-mt-10 w-full text-center">
  <img src="git.svg" class="mx-auto" />
</div>

---

# CLI Apps

Why am I talking about them?

<v-clicks>

- I work on a number of CLI apps as a part of my job and use them daily
- I just really like CLI apps
  - They're fun to build and the terminal is a challenging, somewhat constrained, environment
- The things I'm talking about here don't just apply to CLI apps
- There are other types of apps that also benefit from having no dependencies or build and deploy steps
  - Serverless functions
  - Build scripts
  - GitHub Actions

</v-clicks>

<!--
Now that we've covered a bit of background let's take
-->

---

# What's New in Node.js?

#### As of v18.3.0

<!--
Now that we've covered a bit of background let's take a look at what's new in Node.js
-->

---

# Argument Parsing

<v-clicks>

- Simple scripts often don't take any inputs
- As they get more complex it becomes helpful to accept some user input
- This also helps make scripts more general and useful in more cases
- Most languages provide at least a basic argument parser

</v-clicks>

<!--
Argument parsing is something the tooling group worked on for a long time.
Lots of different people participated in shaping the API and implementing this feature. I helped a tiny bit and am one of many people that worked on this.
Available in Node 18.3.0, which, at the time of this recording, was released yesterday.
-->

---

# Argument Parsing

Parsing arguments is surprisingly complicated and can be tedious and error prone

<v-clicks>

```txt{1|2|3|4|5|all}
mycli --silent
mycli -s
mycli --silent=true
mycli --slient false
mycli --no-silent
```

These all do the same thing, set the `silent` argument, but in several different ways

</v-clicks>

---

# Argument Parsing

How did we used to handle this?

```js{1|3|4|5|6|7|all|all}
const args = process.argv;

// ['node', 'mycli', '--silent']
// ['node', 'mycli', '-s']
// ['node', 'mycli', '--silent=true']
// ['node', 'mycli', '--slient', 'false']
// ['node', 'mycli', '--no-silent']
```

<v-after>

More likely you would use a third-party library like `yargs` or `commander`

</v-after>

---

# Argument Parsing

Now there's a new way to do this in Node.js! Introducing `util.parseArgs` 🎉

```js{1|3|4-8|11|all}
import { parseArgs } from 'node:util';

const input = process.argv.slice(2);
const options = {
  silent: {
    type: 'boolean',
    short: 's',
  },
};

const { values } = parseArgs({ input, options });
```

<!--
Note that we're still slicing off the first two element of process.argv. This is to get rid of the node executable and the name of your script.
There is another proposal being worked on to define something called process.mainArgs which would only be the arguments passed to your program.
-->

---

# Argument Parsing

Using our previous examples this is what `parseArgs` will give us

```js
const { values } = parseArgs({ input, options });

// { silent: true }
// { silent: true }
// Option '-s, --silent' does not take an argument
// Option '-s, --silent' does not take an argument
// Unknown option '--no-silent'
```

<!--
You'll notice that all of these don't actually work.
For the third and fourth examples you get an error because the silent flag doesn't take an argument, it's just a boolean.
For the fifth example you don't automatically get a no prefixed flag to set a value to false. You could add this yourself fairly easily or use a third-party library to support this.
You'll notice in each of these error cases that we get a pretty good and clear error message from parseArgs.
-->

---

# Fetch

<v-clicks>

- Web API for making HTTP requests
- People have been attempting to "make fetch happen" in Node.js for several years

</v-clicks>

---

# Fetch

<div class="mt-10 w-full text-center">
  <img src="fetch.png" class="w-120 mx-auto" />
</div>

---

# Fetch

- Web API for making HTTP requests
- People have been attempting to "make fetch happen" in Node.js for several years
- Fortunately we didn't listen to that advice and Fetch is now available in Node.js 18! 🎉

<v-clicks>

- Built on top of Undici (an HTTP/1.1 client, written from scratch for Node.js)
- Built with the Web Streams API which is also now available in Node.js 18

</v-clicks>

---

# Fetch

An example of using `fetch` to query the GitHub API for my profile

```js{1|2|all}
const response = await fetch('https://api.github.com/users/iansu');
const body = await response.json();

// {
//   "login": "iansu",
//   "name": "Ian Sutherland",
//   "company": "@neofinancial",
//   "blog": "https://iansutherland.ca/",
//   "location": "Calgary, AB"
// }

```

---

# Test Runner

<v-clicks>

- Basic API to create tests, both synchronous and asynchronous
- Supports subtests, `skip` tests, `todo` tests and `only` tests
- Can be used with Node's existing built in `assert` library
- Simple runner to run tests
  - You can provide a list of files
  - It will automatically run files in a `test` directory
  - It will automatically run files that end in `.test.js`, `.test.cjs` and `.test.mjs`

</v-clicks>

---

# Test Runner

Let's write some basic tests

<v-click>

```js{1|2|4-6|8-10|all}
import test from 'node:test';
import assert from 'node:assert';

test('synchronous test', (t) => {
  assert.strictEqual(1, 1);
});

test('asynchronous test', async (t) => {
  assert.strictEqual(1, 1);
});
```

</v-click>

---

# Test Runner

Now let's run those tests

```txt{1|all}
node --test

TAP version 13
ok 1 - index.test.js
  ---
  duration_ms: 0.038415792
  ...
1..1
# tests 1
# pass 1
# fail 0
# skipped 0
# todo 0
# duration_ms 0.095865209
```

---

# Recursive Filesystem Operations

<v-clicks>

- Built in methods like `fs.readFile`, `fs.unlink`, etc. are great for working with a small number of files
- If you want to work with a large number of files and/or directories recursive operations are helpful
- In the past doing any recursive filesystem operations required adding one or more dependencies to your project
- Not anymore!

</v-clicks>

---

# Recursive Filesystem Operations

`fs.mkdir` - recursively create a directory and any non-existent parent directories

<v-click>

```js
import { mkdir } from 'node:fs/promises';

await mkdir('/tmp/a/b/c', { recursive: true });
```

</v-click>

---

# Recursive Filesystem Operations

`fs.rm` - recursively delete a directory and all child directories and files

<v-click>

```js
import { rm } from 'node:fs/promises';

await rm('/tmp/a', { recursive: true, force: true });
```

</v-click>

<!--
This is probably the feature from this talk that I worked on the most.
This is implemented based on the rimraf package.
-->

---

# Recursive Filesystem Operations

`fs.cp` - recursively copy a directory and all child directories and files

<v-click>

```js
import { cp } from 'node:fs/promises';

await cp('/tmp/a', '/tmp/z', { recursive: true });
```

</v-click>

<div v-click class="mt-10">
  <heroicons-outline-exclamation-circle class="text-blue-500" /> This API is currently marked as experimental
</div>

<!--
The implementation of this feature is based on the copy method from the fs-extras package.
-->

---

# Recursive Filesystem Operations

`fs.readdir` - recursively read the contents of a directory and all child directories

<v-click>

```js
import { readdir } from 'node:fs/promises';

await readdir('/tmp/a', { recursive: true });
```

</v-click>

<div v-click class="mt-10">
  <heroicons-outline-exclamation-circle class="text-blue-500" /> There is currently an open PR for this feature: <a href="https://github.com/nodejs/node/pull/41439">nodejs/node#41439</a>
</div>

---

# What's Next?

<!--
What do I want to see in Node in the future?
These are just my ideas, not necessarily things that are being worked on or will be worked on.
-->

---

# What's Next?

### Glob?

<div class="-mt-10 w-full text-center">
  <img src="glob.svg" class="mx-auto" />
</div>

<!--
Glob allows you to use patterns to select multiple files in a directory tree.
Pairs very nicely with recursive fs operations.
-->

---

# What's Next?

### Self-contained Executables?

<div class="-mt-10 w-full text-center">
  <img src="compile.svg" class="mx-auto" />
</div>

<!--
Getting rid of the install step when distributing something like a CLI is a great start. You're still depending on your users to have a compatible version of Node installed though.
Building a self-contained executable solves that problem too.
We've been discussing this on and off for the last few years in the Tooling Group but no major progress yet.
If you do want to do this I'd recommend a third-party package like boxednode or pkg.
-->

---

# What's Next?

### TypeScript?!

<div class="-mt-10 w-full text-center">
  <img src="typescript.svg" class="mx-auto" />
</div>

<!--
As far as I know this isn't happening... yet.
I have seen some Twitter conversations about this idea though.
Node.js supports custom loaders (with ESM) and there is already a loader for TypeScript called ts-node.
What if we just automatically used that loader for .ts files?
-->

---

# What's Next?

### Want to help figure out, and build, what's next?

<div v-click class="flex">
  <div>
    <h3>Join the Node Tooling Group!</h3>
    <h3 class="-mt-2"><a href="https://github.com/nodejs/tooling">github.com/nodejs/tooling</a></h3>
  </div>
  <div class="-mt-5 ml-auto mr-30 text-center">
    <img src="wrench.png" class="mx-auto" />
  </div>
</div>

<!--
You can find everything you need to know about the Tooling Group and how to get involved at that URL.
Also feel free to reach out to me directly on Twitter or if you see me at a conference.
-->

---

# Putting It All Together

<v-clicks>

- I wanted a way to try out all these new features
- What's a CLI app that makes API requests, downloads files and manipulates the filesystem? 🤔
- A package manager! 📦
- Let's build a package manager...?! 🥴

</v-clicks>

---

# Bad Package Manager

Introducing Bad Package Manager (or `bad`). It's a package manager that is... bad. For science!

<v-clicks>

- Might one day be a good CLI app
- Will probably never be a good package manager
- Ongoing project to test new features in a Node.js CLI app
- Currently uses `parseArgs`, `fetch`, `test`, `rm`, `cp`, `mkdir`
- Contributions welcome!
  - [iansu/bad-package-manager](https://github.com/iansu/bad-package-manager)

</v-clicks>

---

# Bad Package Manager

Currently supported features

<v-clicks>

- `bad install` - install all `dependencies` and `devDependencies` from `package.json`
- `bad install <package>` - install the named package and add it to `dependencies` in `package.json`
- `bad install --dev <package>` - install the named package and add it to `devDependencies` in `package.json`
- `bad clean` - delete `node_modules`

</v-clicks>

<!--
I should mention that all these features probably don't work great.
It is the Bad Package Manager after all.
If you try to run the install command in a big project with lots of dependencies it will probably fail or, at the very least, not build a correct dependency tree.
-->

---

# Bad Package Manager

Future ideas

<v-clicks>

- `bad uninstall <package>` - uninstall previously installed `dependencies` and `devDependencies`
- `bad run <script>` - run scripts specified in `package.json`
- `bad ls <package>` - list part of the package tree (hopefully with `fs.readdir`)
- ?

</v-clicks>

<!--
If you have ideas please feel free to open an issue or submit a PR to the project.
-->

---

# Bad Package Manager

<div class="mt-10 w-full text-center">
  <img src="bad-github.png" class="mx-auto h-60 border" />
</div>

<!--
Once again this is where you can find the project on GitHub.
-->

---

# Thanks for Watching!

<h2>Ian Sutherland</h2>
<h4 class="-mt-2">Head of Developer Experience (DX) at Neo Financial</h4>
<h4 class="-mt-2">Node.js Contributor, Tooling Working Group</h4>
<h4 class="-mt-2">Maintainer of Create React App</h4>

<div class="mt-10 inline-flex">
  <simple-icons-twitter class="mr-4 text-3xl text-black" /> <simple-icons-github class="mr-5 text-3xl text-black" /> <span class="text-3xl text-black">@iansu</span>
</div>

<!--
That's all I've got for today.
Please reach out on Twitter or GitHub if you have questions or feedback.
Thanks for watching!
-->

---

# Thanks for Watching!

<h2>Ian Sutherland</h2>
<h4 class="-mt-2">Head of Developer Experience (DX) at Neo Financial</h4>
<h4 class="-mt-2">Node.js Contributor, Tooling Working Group</h4>
<h4 class="-mt-2">Maintainer of Create React App</h4>

<div class="mt-10 inline-flex">
  <simple-icons-twitter class="mr-4 text-3xl text-black" /> <simple-icons-github class="mr-5 text-3xl text-black" /> <span class="text-3xl text-black">@iansu</span>
</div>
