---
title: "esbuild: Present and Future"
seoTitle: "Integrate esbuild with react"
seoDescription: "This blog post compares tsc and esbuild, and provides sample of how to integrate esbuild with react."
datePublished: Wed Mar 01 2023 14:53:26 GMT+0000 (Coordinated Universal Time)
cuid: clepssdlu00010akyf19vca0t
slug: esbuild-present-and-future
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1677682258834/e5446629-3123-4381-85ba-098c0733ede5.png
tags: web-development, reactjs, typescript, frontend-development, nextjs

---

It’s 2023. Fast may not be strong always, but it’s a must-have attribute, especially in technology. As someone who likes to tinker around JavaScript and its frameworks, I spend quite a lot of time playing with bundling tools. And…most of them are not fast enough. Take webpack, rollup etc. for example. *(Except Vite though. As for why, it uses esbuild under the hood for dependency pre-bundling, and esbuild is extremely fast.)* Thus began my journey to find a bundler that is extremely fast and efficient. Then, I encountered [**esbuild**](https://esbuild.github.io/).

It’s no surprise that more and more languages such as Rust and Go are entering the web realm along with JavaScript. esbuild is a very good example. It’s written in Go and is not only a **JavaScript Bundler** but interestingly enough, can also **transpile TypeScript into JavaScript**. And, it is **blazingly fast**.

**esbuild is 100x faster than webpack as a bundler and 500x faster than the typescript compiler**. First, let’s quickly compare the speed between esbuild and tsc when transpiling into JavaScript. Let’s quickly run through some tests to see how fast esbuild is.

First, you can install typescript and esbuild. esbuild is available in npm registry. You can use your favorite package manager to install them. Here, I am using npm.

```bash
npm install -g typescript
```

```bash
npm install -g esbuild
```

I am installing them globally to test them out quickly. We will, later on, use esbuild in our mini-test project as a bundler. Now, let’s create an extremely complex typescript file index.ts as follows:

```typescript
const testFunction = (name: string)=>{
    console.log(`Hello, ${name}`);
}

testFunction("Anurag");
```

Let’s compile it with our old buddy tsc first and see how much time it takes to compile it down to JavaScript. To view the time, we can use the “time” command to find out how long it takes to run:

```bash
time tsc index.ts --target es6
```

I’m using the target as es6 because esbuild transpiles the JavaScript to es6 too. It’s just a way to create a similar scenario as much as possible for both cases. The above command should result in something like this:

![](https://cdn-images-1.medium.com/max/800/1*jtITnxAfrlbd7Y1IS4rJdg.png align="left")

Output of tsc

The typescript compiler is taking **2 whole seconds** just to compile a file into JavaScript.   
*Note: “real” is the wall clock time. It's from start to finish of the command execution.*

Now, let’s try to do the same with esbuild and see the art for ourselves.

```plaintext
esbuild index.ts --outfile=index.js
```

![](https://cdn-images-1.medium.com/max/800/1*dtHf6QjV9VLAzWe7h9pOpw.png align="left")

Output of esbuild

Voila!   
It took just **4ms** for esbuild. That’s more than blazingly fast. That’s blazing-blazingly fast, right? It’s more than 500x faster than the typescript compiler.  
*We need to mention the output file to esbuild via — outfile flag.*

Now that we’ve looked at how awesome an esbuild is, let’s see its magic as a bundler too. We can create a JavaScript file to pass all the available options and logic and run through the node instead of having to pass all the flags via CLI. Let’s **npm init** and **install esbuild as a dev dependency,** go through basic project setup and create a **esbuilder.js** *(you can name it anything)* as follows:

```javascript
require("esbuild").build({
    minify: true,
    outdir: "./public/dist",
    bundle: true,
    incremental: true,
    entryPoints: ["index.ts"],
    target: ["safari11", "chrome58", "firefox57", "edge16"],
    sourcemap: true,
    watch: {
        onRebuild(err, result) {
           if (err) {
            console.error(err);
           }
           else {
            console.clear();
            console.log("[BUILD SUCCEEDED]");
            console.log(`Errors: ${result.errors.length}`);
            console.log(`Warnings: ${result.warnings.length}`);
           }
        }
      }
   });
```

Let me quickly explain what it is doing.

a. **minify**: Do you want to minify your JavaScript?

b. **outdir**: What’s the output directory of your generated ‘magic’ bundle?

c. **bundle**: Should enable bundling or not?

d. **entryPoints**: Where should esbuild start bundling? As it’s an array, you can provide multiple files.

e. **target**: What should be the target environment of the generated JavaScript? Note that esbuild is not going to automatically add polyfill for old browsers. We have to use something like core-js or you can write the polyfills that you need.

f. **sourcemap**: Should esbuild generate source map for you? Source maps make debugging easier but you should avoid it in production to reduce your bundle size.

g. **watch**: Should esbuild watch what is changing in your code and generate bundle continuously? Think of it as real-time bundling. It can be both a boolean and an object. We can pass a function onRebuild if we like as written above.

h. **incremental**: Should esbuild enable incremental build? It’s highly useful when we have to build with esbuild frequently.   
Imagine we are building a library and we’ve enabled watch to constantly monitor what is changing in our code. That means we are rebuilding on each change in our code, right? Is it a good way?   
Here comes the incremental option to the rescue. It enables caching and reusing. It stores Abstract Syntax Tree in the memory the first time it runs and the next time it rebuilds, it compares it with the stored AST if anything has changed. If not, esbuild won’t bother to rebuild again. If you don’t know what AST is, it is a syntax tree created by a syntax parser when you run your JavaScript.

esbuild also stores files it is watching in the memory and compares metadata before rebuilding.

And… that’s it. There are a bunch of other awesome options such as   
[**plugins**](https://github.com/esbuild/community-plugins),  
**loglevel**,  
**format**,  
**platform** and tons of others. You can check out [esbuild docs](https://esbuild.github.io/api/#build-api) and explore yourselves.

Here’s a sample project that implements the above-mentioned configurations for esbuild.

[https://github.com/anurag-dhamala/esbuild-tester](https://github.com/anurag-dhamala/esbuild-tester)

It is also easy to set up esbuild in combination with react and use esbuild as a development server. Then you’ll realize how fast your development flow becomes. You can even use create-react-app starter template and replace it with esbuild and plugins. You have to modify your **package.json, index.html and App.js** a bit, but it is worth it. Here’s an example project that was created using **create-react-app**, but is using **esbuild for bundling the react project.**

[https://github.com/anurag-dhamala/esbuild-intergration-with-cra](https://github.com/anurag-dhamala/esbuild-intergration-with-cra)

As awesome as esbuild is, we need to use some caution while working with it. Transpiling es6+ JavaScript to es5 and below is not fully supported. You can find the JavaScript caveats mentioned in the [documentation](https://esbuild.github.io/content-types/#javascript-caveats). In the future, esbuild may work on some issues it has now, and will become even more reliable.

On the final touch, let’s start using esbuild right away on our projects and experience its magic !!