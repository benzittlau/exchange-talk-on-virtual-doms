# What is a Virtual DOM?
#### @benzittlau
#### http://getjobber.com
#### http://benzittlau.com
#### http://springlaunched.com
#### http://zittlau.ca
#### http://github.com/benzittlau



## Not related to Shadow DOM
Shadow DOM is a feature currently only supported by Chrome and Opera to create an encapsulated "Shadow Tree" within a document.

Useful for isolating the styling and scripting of a web component from it's surrounding document, isolating the semantics.

Not related to Virtual DOM.



## Bitmap Display
![](img/bitmap_voltages.jpg)


## Hantronix HDM64GS24L 240x64 display
Each frame required writing to a memory buffer on the display driver chip.


## Real Time Loop
Software was single threaded, so the render occurred in the main loop of the application.

Writing updates for all 15360 pixels each frame was slowing down the overall system real time loop.

Long render times could cause things like delays to signals to tell the motors to stop driving.

![](img/real-time-loop.jpg)


## BitMap Diff
Maintaining a buffered version of the display memory in MCU memory allowed us to only write changes to the display driver.

![](img/bitmap-diff.jpg)




## Virtual DOM
The name of the engine in React used for making drawing the UI more efficient

Also the name of the general idea of tracking a "Virtual" DOM in memory for performance enhancement.


## Expressing State through UI
In modern web applications we often allow client side *state* to be modified without a page load.  This could be something as simple as a new chat message, or something as complex as changing to a new route within an application.

The primary tool we have to express state through an User Interface is the DOM (Document Object Model). (There are others like SVG or Canvas)

HTML and the DOM were not intended to support dynamic interfaces, changes to the DOM are often slow to redraw.


## Reflows and Repaints
Anything that changes the DOM or it's styling can trigger a Reflow, or a Repaint.

Reflows are generally more expensive than Repaints, as they require performing the layout for the entire DOM.  Reflows will be triggered by anything that can affect the structure of the page (changing page content, font size, dimensions, adding or removing an element).

Repaints are generally less expensive, but are still *expensive*.  Repaints are triggered when anything stylistic changes (background color, text color, etc.)


## Reflows and Repaints - Examples
### Repaint - Color change
![](img/repaint.gif)

[Codepen](http://codepen.io/benzittlau/pen/obPOYZ)

### Redraw - Layout change
![](img/redraw.gif)

[Codepen](http://codepen.io/benzittlau/pen/VeGNvp)



## Things the browser does
As the DOM is a tree changes higher up the tree invalidate more branches, requiring a more expensive Reflow.

Browsers will also try and batch changes themselves, but actions by the user can force them to reflow more aggressively (for example fetching the `offsetLeft` property of an element)



## DEMO!
Let's look at the degraded performance of a fairly simple example

[Basic game grid](http://codepen.io/benzittlau/pen/jWpBxb/)



## Some Ideas To Improve Things
In trying to improve the rendering performance of an HTML5 application there are a few key ideas:

* Dirty Flagging

* Virtual DOM Diffing

* Update batching


## Dirty Flagging
Dirty flagging is used to identify what has and *hasn't* changed, so we can focus on only certain areas (components, templates, subtrees) to re-render.

In Ember this is handled through bindings, observers, and computed property's.

In React this typically a more manual process owned by the user (`shouldComponentUpdate`), unless implementing immutable data structures.


## Virtual DOM Diffing
Once it is determined which areas of the tree might have mutated the Virtual DOM comes into play

It's job is to compare the previous and next state of it's own in-memory respresentation of the DOM to determine where changes have occurred

Deterministicly calculating the ideal most efficient path to mutate the tree is possible but far to expensive.

Instead it applies a series of heuristics to determine the most efficient way to mutate the DOM from the previous, to next, state


### Virtual DOM Diffing - Examples
Simple case:
```
renderA: <div><span>first</span></div>
renderB: <div><span>first</span><span>second</span></div>
=> [insertNode <span>second</span>]
```

Edge case:
```
renderA: <div><span>first</span></div>
renderB: <div><span>second</span><span>first</span></div>
=> [replaceAttribute textContent 'second'], [insertNode <span>first</span>]
```

[Source](https://facebook.github.io/react/docs/reconciliation.html)


### Virtual DOM Diffing - Model
![](img/virtual-dom-model.jpg)


## Update Batching
Ensure that there is at most one layout performed each time you update the state of the page.


## Using a "loop" strategy
A common strategy is to use an event "loop", where during one iteration of the loop all changes are batched, and "flushed" once per loop.

In Ember this ensures that a change to a property and all it's related properties downstream will only trigger a single update.


## Using the request animation frame
HTML5 has a [`requestAnimationFrame`](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) API

> The Window.requestAnimationFrame() method tells the browser that you wish to perform an animation and requests that the browser call a specified function to update an animation before the next repaint. The method takes as an argument a callback to be invoked before the repaint.

In React the batching strategy is pluggable, so one using `requestAnimationFrame` has been implemented as a [proof of concept](https://github.com/petehunt/react-raf-batching)



## Ember (and Glimmer)
Ember uses an engine called "Glimmer" to manage it's in-memory representation of the DOM

It's strategy is different.

Leverages HTMLbars (it's templating engine) to differentiate between static and dynamic components, to avoid checking static components for changes.

Stores it's virtual DOM representation as lightweight "stream-like objects" instead of full objects.


## Ember Streams
DOM trees have one node for each tag and one node for the text between nodes.

Embers streams represent the same *structure* in a very light way:

![](img/ember-streams.jpg)

[Source](https://www.youtube.com/watch?v=DrFXw0QGDLM)


## Ember Bindings
Ember has bindings so it can observe changes to `values` and inject those changes.

![](img/glimmer-model.jpg)



## Write only algorithms
Both React and Ember's algorithms for updating the DOM are *write-only*

The assume that their model of the world is accurate based on the belief that they *created* the world and have mutated it in sync with their own model of it through every variation of state.

This means that you should *not* touch the DOM other than through their objects.



## Surprises
This technology is still very immature and evolving.

Differences in the structures of Ember and React have taken them down very different paths towards addressing similar problems.

What we think of as "Virtual DOM" is a combination of different technologies acting in coordination with each other.  Understanding how your components and data models affect rendering performance will help you write better application structures from the start.



## References
[React - Advanced Performance](https://facebook.github.io/react/docs/advanced-performance.html)
[Ember - Array Computed Refactor](http://www.thesoftwaresimpleton.com/blog/2015/04/18/arraycomputed-refactor/)
[Virtual Dom vs Incremental Dom vs Glimmer](https://auth0.com/blog/2015/11/20/face-off-virtual-dom-vs-incremental-dom-vs-glimmer/)
[React - Glimmer Announcement](https://github.com/emberjs/ember.js/pull/10501)
[React - Component Lifecycle](https://facebook.github.io/react/docs/component-specs.html)
[React - Terminology](https://facebook.github.io/react/docs/glossary.html)
[React - Batch Updating](http://www.simonkrueger.com/2015/11/06/Batch-Updating-In-React.html)
[The Secret's of React's Virtual DOM](http://conferences.oreilly.com/fluent/fluent2014/public/schedule/detail/32395)
[Virtual COM (library)](https://github.com/Matt-Esch/virtual-dom/wiki)
[React's Diff Algorithm](http://calendar.perfplanet.com/2013/diff/)
[Change and it's Detection](http://teropa.info/blog/2015/03/02/change-and-its-detection-in-javascript-frameworks.html)
[React - Reconciliation](https://facebook.github.io/react/docs/reconciliation.html)
[HTMLBars Deep Dive](https://www.youtube.com/watch?v=DrFXw0QGDLM)
[Why is the DOM slow?](https://news.ycombinator.com/item?id=9155564)
[Repaints and Reflows](http://blog.letitialew.com/post/30425074101/repaints-and-reflows-manipulating-the-dom)
[Rendering Repaint and Reflow](http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/)

