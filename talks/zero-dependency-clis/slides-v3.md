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
<h4 v-click class="-mt-2">Architect, Head of DX and OSS at Neo Financial</h4>
<h4 v-click class="-mt-2">Node.js Contributor, Tooling Working Group</h4>

<div v-click class="mt-10 inline-flex">
  <simple-icons-twitter class="mr-4 text-3xl text-black" /> <simple-icons-github class="mr-5 text-3xl text-black" /> <span class="text-3xl text-black">@iansu</span>
</div>

<!--
First let me quickly introduce myself.

My name is Ian Sutherland and I'm an Architect and the Head of Developer Experience and Open Source at Neo Financial, which is a Canadian fintech startup.

I'm also a Node.js contributor where I mostly work on the Tooling Working Group

You can find me on Twitter and GitHub @iansu.
-->

---

# What We're Covering Today

<v-clicks>

- What is a CLI app and why are we talking about them?
- What are dependencies and why are we trying to avoid them?
- Why build CLI apps with Node.js?
- What new Node.js features can help us write CLI apps?
- What new Node.js features might be coming next?

</v-clicks>

<!--
**BEFORE**

Here's an overview of we're going to cover
-->

---

# CLI Apps

What are they?

<v-clicks>

- CLI stands for Command Line Interface
  - Basically a fancy way to describe a script that takes some input and generates output
- These kinds of apps run in a terminal
- Commonly used for dev tools, build scripts and process automation
- Range from simple like `ls` to complex like `git`

</v-clicks>

<!--
**AFTER**

Who here has used a CLI app before?

Who here has built a CLI app before?

Who here has written a script or a shell script before?

Who's used Node.js to build a CLI or script before?

A lot of you are already familiar with CLI apps but we're going to look at a few examples and cover some basic terminology
-->

---

# CLI Apps

<div class="-mt-10 w-full text-center">
  <img src="/ls2.svg" class="mx-auto w-180" />
</div>

<!--
Here's an example of the `ls` command

Relatively simple CLI that takes a number of arguments and options

Some terminology here: those arguments after the dash are sometimes called options, short options, shortopts or just arguments

There are actually two short options there: l and 1

Could be written as `-l` and `-1` but it's common to combine them
-->

---

# CLI Apps

<div class="-mt-10 w-full text-center">
  <img src="/git.svg" class="mx-auto" />
</div>

<!--
Git is a much more complex CLI app that has a number of different subcommands (like status shown here)

Those subcommands take all kinds of different arguments and options
-->

---

# CLI Apps

Why am I talking about them?

<v-clicks>

- I work on a number of CLI apps as a part of my job and use them daily
- I just really like CLI apps
  - They're fun to build
  - The terminal is a challenging and constrained environment
  - Serious 80s vibes
- The things I'm talking about here don't just apply to CLI apps
- There are other types of apps that have similar constraints
  - Serverless functions
  - Shell scripts
  - GitHub Actions

</v-clicks>

---

# Why Build CLI Apps with Node.js?

<v-clicks>

- Shell scripts are often written with scripting languages like Perl or Python so why not Node.js?
- Use the language you (and your team/collaborators) are already familiar with
- Node.js is really good at some things
  - Working with JSON data
  - Doing tasks in parallel like reading and writing files, making network requests, etc.
- Node.js has a huge package ecosystem
  - Even though we're trying to avoid dependencies for this talk they are still very useful and npm has _a lot_ of them

</v-clicks>

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
**AFTER**

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
- If you're building a complex app it probably already has a build and deploy pipeline and many dependencies so adding more dependencies likely doesn't make much of a difference
- For smaller apps and libraries, like CLIs or shell scripts, not having an install or build step makes setup, contributing and distribution easier

</v-clicks>

<!--
**AFTER**

In the case of a shell script, having dependencies means you either have to install those dependencies to use the script, which is really not how shell scripts work. They are generally self-contained.

The other option is to bundle your code and dependencies using something like webpack. There are tools like ncc from Vercel that do this but that results in large files. For example, we have a basic CLI app at Neo called the Neo CLI and the compiled version of it is 12MB!

Bundling with webpack also doesn't work with all dependencies
-->

---

# What's New in Node.js?

#### As of v18.3.0

<!--
What's new in Node.js

A little disclaimer before I talk about these new features: I did not build all of these things and I wasn't involved at all in some of them

Node.js is a huge project with lots of collaborators and contributors and a lot of people worked on these new features and features that they're built on top of

Before we get to the code I want to explain some new syntax I'm going to be using...
-->

---

# New Syntax

I'm going to be using some new syntax in these examples

<v-clicks>

- `import` instead of `require`
  - ES Modules instead of CommonJS
  - Add `type: module` to `package.json` to use
  - Or name all your files with the `.mjs` extension
- Import Node.js internals with `node:` prefix
  - For example: `import fs from 'node:fs'`
  - Disambiguates internal libraries from npm packages

</v-clicks>

<!--
**AFTER**

Since pretty much every package name is taken on npm it's basically impossible to add new modules to Node.

This new namespace solves that and makes it more clear where something is imported from.

Okay, now on to the fun stuff!
-->

---

# Argument Parsing

<v-clicks>

- Simple scripts often don't take any inputs
- As they get more complex it becomes helpful to accept some user input and arguments
- This also helps make scripts more general and useful in more cases
- Most languages provide at least a basic argument parser

</v-clicks>

---

# Argument Parsing

Parsing arguments is surprisingly complicated and can be tedious and error prone

<v-clicks>

```txt{1|2|3|4|5|all}
mycli --silent
mycli -s
mycli --silent=true
mycli --slient true
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
// ['node', 'mycli', '--slient', 'true']
// ['node', 'mycli', '--no-silent']
```

<v-after>

More likely you would use a third-party library like `yargs` or `commander`

</v-after>

<!--
**BETWEEN**

There are even more possible ways to pass arguments.

I mentioned position arguments before like a filename to operate on.

I also mentioned that short options can be specified individually or combined.

Some long options can take multiple arguments.

Some arguments contradict like a silent and a verbose argument.

This gets very tedious very fast.

More likely you're going to use a library.
-->

---

# Argument Parsing

Now there's a new way to do this in Node.js! Introducing `util.parseArgs` 🎉

<v-after>

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

</v-after>

<!--
**BETWEEN**

Note that we're slicing off the first two element of process.argv. This is to get rid of the node executable and the name of your script.

There is another proposal being worked on to define something called `mainArgs` which would give you only the arguments passed to your script.
-->

---

# Argument Parsing

Using our previous examples this is what `parseArgs` will give us

```js{all|3-4|5-6|7|all}
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

As I mentioned earlier, this is available in Node 18.3 but I believe it's also being backported to Node 16 very soon.
-->

---

<div class="mt-10 w-full text-center">
  <img src="/parseargs-github.png" class="mx-auto h-80 border" />
</div>

<!--
There's also a polyfill at `@pkgjs/parseargs`

This is where we initially developed the library and where we work on and discuss new features
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
  <img src="/fetch.png" class="w-120 mx-auto" />
</div>

<!--
You're legally required to use this GIF when talking about Fetch in Node.js
-->

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

<!--
**AFTER**

You'll notice you have to await getting the response body as JSON.

This is because the response body is streamed from the server. This is much more efficient in the case of large responses.
-->

---

# Life Before Fetch

<div class="mt-10 w-full text-center">
  <img src="/caveman.gif" class="h-80 mx-auto" />
</div>

<!--
I'm going to tell you a story and we'll look at some code

I learned a while ago that if you use a subcommand that git doesn't recognize it will look for a matching file in your path.

For example, if you typed `git hello` it would realize that `hello` doesn't match any built in subcommands and will look for a file named `git-hello` on your path. If it finds it, it'll run it.

This is a pretty cool way to make a CLI app extensible.

A little while later I saw this tweet...
-->

---

<div class="mt-10 w-full text-center">
  <img src="/git-blast-tweet-1.png" class="mx-auto h-100" />
</div>

<!--
If you're not familiar with `git blame` it displays a file and beside each line of code it shows the name or email address of the person who wrote it

I guess the idea with the name is that you can blame whoever introduced the bug you just found (it usually ends up being you)
-->

---

# Git Blame

<div class="w-full text-center">
  <img src="/git-blame.png" class="mx-auto" />
</div>

<!--
Here's an example of what git blame shows when I run it on these slides

And then I saw a followup tweet...
-->

---

<div class="mt-10 w-full text-center">
  <img src="/git-blast-tweet-2.png" class="mx-auto h-100" />
</div>

<!--
That's a great name and I just recently learned you can add your own subcommands to git

So I decided to build it.

I figured I would need to write a script that would run `git blame` and then every time it encountered a new email address it would get that person's public profile from GitHub and if they provided their Twitter username display that instead of their email address.

But this needed to be a self-contained shell script and it would need to make a request to the GitHub API.

I wanted to use Node.js but that meant using the built in HTTP library to hit the GitHub API and here's what that looks like...
-->

---

```js{all|1|4|5-11|14|17-19|21-23|25-29|all}
const https = require('https');

const githubRequest = async (path) => {
  return new Promise((resolve, reject) => {
    const options = {
      hostname: 'api.github.com',
      port: 443,
      path: path,
      method: 'GET',
      headers: { 'User-Agent': 'git-blast' },
    };

    https
      .get(options, (resp) => {
        let data = '';
        // A chunk of data has been received.
        resp.on('data', (chunk) => {
          data += chunk;
        });
        // The whole response has been received. Print out the result.
        resp.on('end', () => {
          resolve(JSON.parse(data));
        });
      })
      .on('error', (err) => {
        console.log('Error: ' + err.message);

        reject(err);
      });
  });
};
```

<!--
This is the code that makes a request to the GitHub API using Node's HTTP client. You can see that it's an event based API and we have to handle various success and error events, we receive the data via multiple `data` events and have to assemble that all ourselves

This is a lot of code. In fact it doesn't even all fit on this slide. It's also not particularly readable.

Let's see what this same code would look like if I were to use `fetch` instead.
-->

---

```js
const githubRequest = async (path) => {
  try {
    const response = await fetch(`https://api.github.com/${path}`, { headers: { 'User-Agent': 'git-blast' } });
    const body = await response.json();

    return body;
  } catch (error) {
    console.log('Error: ' + error.message);
  }
}
```

<!--
That's it.

This code does all the same stuff, including error handling.

From more than 30 lines of code to less than 10.
-->

---

# Git Blast

<div class="w-full text-center">
  <img src="/git-blast.png" class="mx-auto" />
</div>

<!--
Here's what the output looks like now. You can see my Twitter name there next to each line instead of my email address.
-->

---

<div class="mt-10 w-full text-center">
  <img src="/git-blast-github.png" class="mx-auto h-80 border" />
</div>

<!--
I really did make this and you can use it.

Please don't actually go yell at people on Twitter about code they wrote though. If you do, be nice.

Let's move on from fetch...
-->

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

<!--
**AFTER**

Normally you might reach for something like Jest or Mocha.

Those are great test frameworks and are a good option in a larger project like a microservice or a frontend App.

A lot of libraries on npm literally have a single JS file and a single test file with a handful of tests.
-->

---

# Test Runner

Let's write some basic tests

<v-click>

```js{1|2|4-6|8-12|all}
import test from 'node:test';
import assert from 'node:assert';

test('synchronous test', (t) => {
  assert.strictEqual(1, 1);
});

test('asynchronous test', async (t) => {
  return Promise.resolve(1).then(value => {
    assert.strictEqual(value, 1);
  });
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

This is implemented based on the rimraf package which was also made by Isaac, the founder of npm.
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

<!--
So that's all the currently available new stuff I wanted to talk about.

Now let's take a look at what might be coming next...
-->

---

# What's Next?

Wild speculation edition

<!--
What do I want to see in Node.js in the future?

These are just my ideas, not necessarily things that are being worked on or will be worked on.
-->

---

# What's Next?

### Glob?

<div class="-mt-10 w-full text-center">
  <img src="/glob.svg" class="mx-auto" />
</div>

<!--
Glob allows you to use patterns to select multiple files in a directory tree.

Pairs very nicely with recursive fs operations.
-->

---

# What's Next?

### Self-contained Executables?

<div class="-mt-10 w-full text-center">
  <img src="/compile.svg" class="mx-auto" />
</div>

<!--
Getting rid of the install step when distributing something like a CLI is a great start. You're still depending on your users to have a compatible version of Node installed though.

Building a self-contained executable solves that problem too.
-->

---

# What's Next?

### TypeScript?!

<div class="-mt-10 w-full text-center">
  <img src="/typescript.svg" class="mx-auto" />
</div>

<!--
As far as I know this isn't happening... yet.

I have seen some Twitter conversations about this idea though.

There's also a Node working group called "Next 10" that is trying to plan or imagine what the next 10 years of Node.js might look like and they actually had a mini summit meeting this morning to talk about TypeScript...

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
    <img src="/wrench.png" class="mx-auto" />
  </div>
</div>

<!--
You can find everything you need to know about the Tooling Group and how to get involved at that URL.

Also feel free to reach out to me directly on Twitter or in person.
-->

---

<!--
That's all I've got for today.
-->

---

# Thanks for Listening!

<h2>Ian Sutherland</h2>
<h4 class="-mt-2">Architect, Head of DX and OSS at Neo Financial</h4>
<h4 class="-mt-2">Node.js Contributor, Tooling Working Group</h4>

<div class="mt-10 inline-flex">
  <simple-icons-twitter class="mr-4 text-3xl text-black" /> <simple-icons-github class="mr-5 text-3xl text-black" /> <span class="text-3xl text-black">@iansu</span>
</div>

<!--
Thanks!
-->

---
clicks: 100
---

# Thanks for Listening!

<h2>Ian Sutherland</h2>
<h4 class="-mt-2">Architect, Head of DX and OSS at Neo Financial</h4>
<h4 class="-mt-2">Node.js Contributor, Tooling Working Group</h4>

<div class="mt-10 inline-flex">
  <simple-icons-twitter class="mr-4 text-3xl text-black" /> <simple-icons-github class="mr-5 text-3xl text-black" /> <span class="text-3xl text-black">@iansu</span>
</div>

<!--
**STOP CLICKING**
-->
