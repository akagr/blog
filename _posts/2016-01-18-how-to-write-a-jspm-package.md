---
toc: true
layout: post
description:
categories: []
title: How to write a JSPM package
---
## Why

So I was assigned with building payment capabilities in my project at work and we decided to go with stripe. If you haven't heard of it, it's an awesome and developer friendly (looking at you, paypal!) payment gate which powers many cool web apps like shopify, kickstarter etc.

Now before I proceed further, let me tell you a couple of things about project and stripe. This is an angularjs app using JSPM (duh!) to manage dependencies. There's already an awesome [angular-stripe](https://github.com/bendrucker/angular-stripe) but it doesn't include `stripe.js`, the base code for all the front-end stuff stripe exposes to developer.

Now there is obviously no problem in downloading this file, placing it somewhere in project and doing:

```javascript
import 'lib/js/stripe';
```

It'd work. But it didn't feel right for two reasons:

1. I had to create that folder just for stripe. This being an angular application, all of the internal libraries were written as services and `stripe.js` wouldn't have fit in with them too well.
2. `stripe.js` is very much an external dependency and should go with other external dependencies in `package.json`.

In case you've not already guessed it, I decided to create a jspm package for stripe.js and import that into project through cli like any other dependency. I was surprised to see how little there is on the web to tell me how to do this. This post is aimed to remedy some of that.

## How

I created a directory for this package (outside of project obviously) and initialised JSPM in it.

```bash
mkdir -p stripe-jspm/lib && cd stripe-jspm
jspm init -y
```

That gave me a `package.json`, `config.js` and an empty `lib` directory.

Next, I downloaded the `stripe.js` and placed it in `lib`.

Since stripe requires jQuery, I added it as dependency for our `stripe-jspm` package.

```bash
jspm install jquery
```

Finally, I needed to specify a `main` file in `package.json` which `system.js` uses to expose the module exported by the package. So I created an `index.js` in project root and populated it with the following:

```javascript
import 'jquery';
import './lib/stripe';

export default Stripe;
```

And added it in package.json:

```json
 {
     "main": "index.js", // <======== This right here
     "jspm": {
         "dependencies": {
             "jquery": "npm:jquery@^2.2.0"
         },
         "devDependencies": {
             "traceur": "github:jmcriffey/bower-traceur@0.0.93",
             "traceur-runtime": "github:jmcriffey/bower-traceur-runtime@0.0.93"
         }
     }
 }
```

Simple. Right?

With that out of the way, I had just some boilerplate stuff left. Adding package details in package.json, name, author... that kind of stuff.

That is pretty much it. I pushed this off to github and was able to install it as a dependency the correct way in my work project.

```bash
jspm install stripe=github:akagr/stripe-jspm
```

Here's the finished thing: [akagr/stripe-jspm](https://github.com/akagr/stripe-jspm)

## On

This package has some room for improvements. For example, we can probably remove `traceur` as a dev dependency since we're not really using it in this package. And since it's written in ES5 and earlier stuff, we're not going to need it either.

This was my first time writing a package for jspm and I liked how I was able to do that and import it within a few minutes. Any suggestions for improvements to package or this post will be much appreciated. Feel free to shoot comments over at [reddit](https://www.reddit.com/r/javascript/comments/41iuaq/how_to_write_a_jspm_package/).

