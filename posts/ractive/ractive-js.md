# Ractive.js

One of the most frustrating parts of writing web apps is DOM manipulation and event handling.
To demonstrate this, let's use this common scenario - we want to hide an element and show another one when a user click on the 'add-note' textbox.

We will look at two way of doing it - one with jQuery and the other with [Ractive.js](http://www.ractivejs.org).
At the end we'll talk a bit about Ractive.js and how it compares to AngularJS and Ember.js.


## The jQuery version

```js
$('.add-note').on('click', 'label.default', function (e) {
    var $this = $(e.currentTarget);
    var $pair = $this.next('.adding');

    $this.toggle();
    $pair.toggle();
});
```

```html
<section class="notes">
    <form action="post">

        <label class="default">
            <input class="add-note" type="text" placeholder="Add a note...">
        </label>

            <div class="adding">
                <label>
                    <textarea name="content" value="" autofocus></textarea>
                </label>

                <button class="save" type="submit">Save</button>

                <a class="cancel" href="#cancelAdd">Cancel</a>
            </div>
    </form>
</section>
```
There are a few problems with this code:

1. It will get messy and hard to maintain (This example is deliberately simple so you won't fall a sleep)
1. DOM changes might break our handler
1. CSS changes (for example changing .add-note to .add) will break our code as well

## The Ractive.js version

```js
// calling the Ractive constructor function will display our template on the page.
// read more about it [here](https://github.com/Rich-Harris/Ractive/wiki/Initialisation-options).
var ractive = new Ractive({
  el: 'notes',
  template: '#notes-template',
});

// click on add will change the 'adding' variable to true which
// results in hiding part of the DOM and showing another one
ractive.on('add', function () {
    this.set('adding', true);
};

```

Ractive uses [mustache](http://mustache.github.io/) for it's HTML templates:
```html
<section class="notes">
    <form action="post">

        {{^adding}}
        <label class="default">
            <input type="text" placeholder="Add a note..." proxy-click="add">
        </label>
        {{/adding}}

        {{#adding}}
            <div class="adding">
                <label>
                    <textarea name="content" value="{{ note.content }}" autofocus></textarea>
                </label>

                <button class="save" type="submit">Save</button>

                <a class="cancel" href="#cancelAdd" proxy-click="cancelAdd">Cancel</a>
            </div>
        {{/adding}}
    </form>
</section>
```
## What's going on here?

Ractive.js is a small library that helps us keep our sanity while working on client-side JavaScript. It offers mainly two features - DOM manipulation and event handling. In our little example above, we added `proxy-click="add"` this allow us to bind the input type to a function. In that function we set the variable `adding` to true. By doing that we hide the DOM that is inside `{{^adding}} ... {{/adding}}` and showing the DOM inside `{{#adding}} ... {{/adding}}`

## When to use it?

If you are familiar with AngularJS or Ember.js you might be asking yourself why not using them instead?  
The answer is (as always) - it depends. Let's look at the following:

###Performance
We haven't done formal testing but observed noticable improvement of client-side rendering with Ractive.js.

### Architecture
Ember.js and AngularJS are great for a single-page-app. But if your app has a lot of server-side rendering and it spits html to the browser, you might not enjoy many features that the bigger frameworks offer.

### Modularity
If all you need is one or two features (in our case it's data binding and event-handling) it might be better to find smaller libraries that ease your pain. AngularJS comes with testing framework, router, data-binding, event handling, dependency injection and more. Right now We don't need many of them.

### Learning curve
Ractive.js takes a few hours to learn. Rumors say that it can be learned inside-out on a flight from LA to Chicago!
AngularJS will require a few weeks of investment and Ember.js...aham..[good luck with that](http://emberjs.com/blog/2013/03/21/making-ember-easier.html).
