---
toc: false
layout: post
description: Find out how to build cancellable async calls with observables
categories: []
title: What Promises Do That Observables Can’t
---
*Note: This is a cross-post of an article I authored. The original was published at [Modus Create, Inc.](http://moduscreate.com/observables-and-promises/) on April 06, 2016.*

**Nothing.**

Observables are grabbing the spotlight as one of the cool new things Angular 2 is doing, despite having been around for some time. They are positioned to fully eclipse promises as the goto abstraction for dealing with async, among other things. Let’s dive into what Observables are and how they compare against promises in dealing with async data.

### Promises and Observables

Promises are great. They were designed to be an answer to callback hell. Javascript’s tendency to keep edging to the right of the screen due to nesting has made many eyes bleed and brains explode. In comparison, the `do this then this then that` approach of promises was not only prettier and easier to read, it also provided a standard which could be embraced by developers to communicate with. Promises soon became the de-facto approach in most major frameworks and libraries. So much so in fact that the ES2015 specification incorporated promises as a core javascript API sans any third party library.

But promises act on data, and then return. They’re just sugar over the callback pattern. You can’t do much with a promise apart from getting a value or error out of it. How do we wrap a socket stream in a promise? Technically, it is never fulfilled. We just keep getting data out of it. We can use the `progress` method but it isn’t exactly meant for that and seems like a hack.

Enter Observables, the shiniest new abstraction for javascript devs. In reality though, Observables are simply the observer pattern at work. We’ve been attaching listeners to DOM events and reacting to events since the big bang. That pattern got abstracted to make it possible to interact with basically all data flow using the observer pattern. The resulting interface (or class/prototype/thing whatever you want to call it) was named Observable. Fitting.

Let’s see if we can reinforce this info by looking at a couple of examples.

```javascript
/* This is how we've been doing observer pattern */
document.addEventListener('mousemove', e => console.log(e.clientX));

/* This is how the new Observable looks like doing same thing */
Rx.Observable.fromEvent(document, 'mousemove')
    .subscribe(e => console.log(e.clientX));
```
Why all the fuss, you ask? Because `Observable` doesn’t only work with DOM events. We can use it to deal with an amazing variety of data.

```javascript
Rx.Observable.range(1, 10)
    .subscribe(e => console.log(e));

const sub = Rx.Observable.interval(1000)
    .subscribe(e => doThisEverySecond());
setTimeout(()=> sub.dispose(), 3000);
```

Notice how we’re using the same interface (`subscribe`) to deal with totally different types of operations (including the document listener we added in the previous example). This is one of the major strengths of using Observables. We don’t need to wire our brains differently. If the data can be thought of as evented, stream or async, we can wrap it in an Observable.

The abstraction of Observables is cool in itself. But that’s only one of the two things that make it awesome. The other half is the extensive Rx.js library itself. It has an amazing collection of methods (called operators) that can be employed to bend data to our will including some really handy utility methods. I’ll let the code do the talking.

```javascript
/* Print 1 to 10 instantly then print a number every 2 seconds */
Rx.Observable.range(1, 10)
    .concat( Rx.Observable.interval(2000) )
    .subscribe(e => console.log(e));

/* Retry the Observable up to 2 times in case of error */
myHttpRequestObservable
    .retry(3) // 2 retries + 1 initial run
    .subscribe(e => console.log(e));

/* Print message if mouse moved within an area */
Rx.Observable.fromEvent(document, 'mousemove')
    .map(e => [e.clientX, e.clientY])
    .filter(e => isInArea(e)) // isInArea return boolean
    .subscribe(e => console.log('Mouse moved!'));
```

This is just the tip of the iceberg. Check out [their repo](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md) for all operators available out of the box.

### Observables in Angular 2

Angular 2 uses Rx.js Observables instead of promises for dealing with HTTP. This means, as we saw in the examples above, they come with some serious batteries included. That’s one of the reasons that HTTP operations in Angular 2 is so amazing. Here’s some code which consumes a quotes API via HTTP get. The result is an observable.

```javascript
export class App {
  qlist: String[] = [];

  constructor (public http: Http) {
  }

  getQuote () {
    return this.http.get('http://quotesondesign.com/wp-json/posts?filter[orderby]=rand&filter[posts_per_page]=1');
  }
}
```

Note that since http.get method returns an Observable, merely calling the getQuote method won’t actually fire a request. An Observable starts emitting data when a subscriber is attached to it. Technically, this type of Observable is called a cold Observable. There’s also hot Observables that can emit data regardless of whether or not there are any subscriptions.

Now that we’ve seen how a simple request observable is defined in Angular 2, let’s see some operators on it. A code snippet is worth a thousand words:

```javascript
class App {

   /* Existing methods … */

    addQuote () {
        this.getQuote()
            .retry(2) // in case of error, try 1 more time
            .repeat(3) // do this 3 times
            .map(res => res.json()) // convert response to json
            .filter(res => res.length > 0) // drop empty array responses
            .map(res => res[0].content.replace(/\<.*?\>/g, ''))
            .subscribe(quote => {
                this.qlist.push(quote);
            }, e => console.log(e.message));
    }
}
```

The above example could be found in action at [this plunk](http://plnkr.co/edit/XiMzik?p=preview).

### Conclusion

Observables are powerful. And we’ve barely scratched the surface. They’re one of the proposed standards in ES2016. Looks like popular libraries are ending up in language specification these days :)

I’d love to hear your takeaways from this post and how you used observables in some new cool way. Any points for improving this post will be deeply appreciated as well. Use [reddit](https://www.reddit.com/r/Angular2/comments/4dmpd9/what_promises_do_that_observables_cant/) or reach me on [Twitter](https://twitter.com/akshagrwl).
