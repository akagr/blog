---
toc: true
layout: post
description:
categories: []
title: Accessing child elements in Angular / Ionic
---
Components are at the heart of Angular, and we have some very useful tools to work with them efficiently. Often, we need to access the children - child elements, in other words - of our components. This can mean getting a reference to the DOM element, or to the actual component object if one exists. For this, we have at our disposal two handy decorators `@ViewChild` and `@ContentChild`, along with their list counterparts - `@ViewChildren` and `@ContentChildren`. Let's see why there are two of these and how we can use them.


### Before we begin

We'll create a very simple counter component. All it will do is display its property and have a method we can use to increment it. We'll demonstrate how to access this component's `increment` method to increase the value later.

```
@Component({
  selector: 'counter',
  template: '<h2>{{counter}}</h2>'
})
export class Counter {
  counter: number = 123;
  
  increment() {
    this.counter++;
  }
}
```

### Children in component's template

There are two places a component can have child elements - template and content. Let's talk template children first. Observe the following component:

```
@Component({
  selector: 'my-app',
  template: `
  <my-dialog>
    <counter></counter>
  </my-dialog>
  `
})
export class AppComponent { }
```

Here, both - `my-dialog` and `counter` are children of `AppComponent` in its template. Let's assume that we have `my-dialog` defined for now. If we want to access the `counter` here, we'll use the `@ViewChild` decorator as follows:

```
@Component({
  selector: 'my-app',
  template: `
  <button (click)="counter.increment()">Increment View Child </button>
  <my-dialog>
    <counter></counter>
  </my-dialog>
  `
})
export class AppComponent {
  @ViewChild(Counter) counter: Counter;
}
```

We've added a button to make it easy to play with this example. The `@ViewChild` decorator can take either a type of component or a selector (string) as argument, and returns the first child element from template that matches it. In this case, since we have a `Counter` component, we get the reference to actual component object that's rendered. Clicking the button increases the counter to 124, proving this.


### Children in component's projected content

In the last example, since both - `my-dialog` and `counter` are present inside the template of `AppComponent`, we can access them with `@ViewChildren`. However, `counter` also seems to be a child of `my-dialog`, although not in its own template. Here's the definition of `MyDialog` as we have it for now:

```
@Component({
  selector: 'my-dialog',
  template: `<ng-content></ng-content>`
})
export class MyDialog { }
```

All this component does is project the content that was provided as its child. This is referred to as transclusion. That said, there is a way of accessing the children in the projected component - `@ContentChild`.

Let's modify our `MyDialog` component a bit. Here's what the entire code ends up looking like:

```
@Component({
  selector: 'my-dialog',
  template: `
  <button (click)="counter.increment()">Increment Content Child </button>
  <ng-content></ng-content>
  `
})
export class MyDialog {
  @ContentChild(Counter) counter: Counter;
}

@Component({
  selector: 'my-app',
  template: `
  <button (click)="counter.increment()">Increment View Child </button>
  <my-dialog>
    <counter></counter>
  </my-dialog>
  `
})
export class AppComponent {
  @ViewChild(Counter) counter: Counter;
}
```

Both - `AppComponent` and `MyDialog` are accessing the same `Counter` component. `AppComponent` is using `@ViewChild` because `<counter>` is present inside its template. On the other hand, `MyDialog` is using `@ContentChild` because `<counter>` is a part of its projected content, not the template itself.


### Matching more than one child

Both the decorators we saw above have a corresponding list version, which matches all the child elements in its scope (template or projected content). Unsurprisingly, they are named `@ViewChildren` and `@ContentChildren`. Now that we know the difference in where they look for children, let's see how we can use `@ViewChildren` quickly.

```
@Component({
  selector: 'my-app',
  template: `
  <button (click)="counter.increment()">Increment View Child </button>
  <my-dialog>
    <counter></counter>
  </my-dialog>
  
  <counter></counter>
  <counter></counter>
  <counter></counter>
  <counter></counter>
  <button (click)="incrementChildren()">Increment View Children </button>
  
  `
})
export class AppComponent {
  @ViewChild(Counter) counter: Counter;
  @ViewChildren(Counter) counters: QueryList<Counter>;
  
  incrementChildren() {
    this.counters.forEach(counter => {
      counter.increment();
    });
  }
}
```

We've added a bunch of `<counter>` elements and a button to increment all of them to our template. To support that, we've added `@ViewChildren` decorator to capture all the `counter` children. It's important to note that `@ViewChildren` and `@ContentChildren` do not return an array. They return a `QueryList`, which is an array like object. It is an iterable, which means we can use that with `ngFor`, and has common array methods like `forEach` which we've made use of. The QueryList automatically tracks addition and removal of any children.

Here's the [finished live plunk](http://embed.plnkr.co/YymATt865X0c7FhqYnRx/) of this.

In the plunk, notice that clicking on the button to increment view children increments all the counters, including the one we've matched with @ViewChild and @ContentChild. Clicking on the corresponding button to increment content and view child continues to increment only the one counter that is matched.
When a child can be of more than one type
Consider the following bit of code:

```
@Directive({
  selector: '[highlight]'
})
export class Highlight {
  constructor(private el: ElementRef) { }
  
  changeColor() {
    this.el.nativeElement.style.color = 'red';
  }
}

@Component({
  selector: 'my-app',
  template: `
  <counter highlight></counter>
  <button (click)="counter.increment()">Increment View Child</button>
  `
})
export class AppComponent {
  @ViewChild(Counter) counter: Counter;
}
```

What if we wanted to match the `<counter>` child, but instead of getting the component object back, needed a reference to the attached directive object? In other words, how can we access the `changeColor` method on our view child?

Fortunately, there's an easy way to do so. But before we get into that, we need to understand how angular decides what to return to us when it sees a  `@ViewChild` decorator. Up till now, we were getting the component object, which allowed us to interact with component's methods directly. However, that's not the end of it. For angular, there can be multiple ways to interpret view-child. It is first and foremost a DOM element. It may also correspond to an angular component. Further, it may also have directives or services attached to it. We can help by telling explicitly what we want.

Turns out, in addition to providing the type/selector of child to match against, we can also provide instructions of interpreting the child object via `read` property. Our decorator will change thus:

```
@ViewChild(Counter, {read: Highlight}) counter: Highlight;
```

Now, `counter.increment()` will result in error, because `counter` is no longer a component object. Instead, it represents the directive object, which means `counter.changeColor()` is the new black.

```
@Component({
  selector: 'my-app',
  template: `
  <counter highlight></counter>
  <button (click)="counter.increment()">Increment View Child</button>
  <button (click)="highlight.changeColor()">Color View Child</button>
  `
})
export class AppComponent {
  @ViewChild(Counter) counter: Counter;
  @ViewChild(Counter, {read: Highlight}) highlight: Highlight;
}
```

Note that although both the `@ViewChild` decorators return references to different types of things, in the end, it is all affecting the same element. By specifying the type of reference we want, we can easily switch between our interpretations of an element.

[The finished plunk](http://embed.plnkr.co/UTcnkOWFwO21qYwMZ2jB/) contains another interpretation we can use - as a DOM element - and accesses the native element directly.


### Summing up

We saw how we can access child elements using different decorators that angular provides, depending on where those elements might occur. We also learned how we can specify the type of child element directly, and interpret same element in different ways. This opens up new ways of interaction between components in complex scenarios when communication through input/output might not be the best (or practical) approach.
