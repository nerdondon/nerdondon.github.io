---
layout: post
title: "[Notes]: requestIdleCallback"
description: "Notes on requestIdleCallback"
tags: notes requestIdleCallback
---

### Prelude

...I got nothing.

### Actual Notes :D

- [requestIdleCallback](https://developer.mozilla.org/en-US/docs/Web/API/window/requestIdleCallback)
  allows you to schedule actions during any remaining time at the end of a frame or when a user is
  inactive.

- This is different from [requestAnimationFrame]({{site.url}}/rAF-notes) in that `rAF` occurs before
  painting.

- The sequence of events is as follows:
    1. execute JS from task queue
    1. `rAF`
    1. style calculations, layout, and paint (collectively called a frame commit)
    1. `requestIdleCallback` (or anywhere between the first 3 steps if the user is idle)

- You cannot tell how much time the first 3 steps will take up, so there is no guarantee that there
  will be any remaining time to allocate to your `requestIdleCallback`'s. This means that only
  non-essential work should be executed using `requestIdleCallback`.

- To ensure that this non-essential work gets done eventually, you can set a timeout when calling
  `requestIdleCallback`. When this timeout is reached, the browser WILL execute the callback passed
  to `requestIdleCallback`.

- It is important to not apply any DOM changes when using `requestIdleCallback` because the callback
  may be executed at the end of frame, i.e. after the frame has been committed. This means that if
  there are any layout reads in the next frame (e.g. getBoundingClientRect), the layout needs to be
  recalculated via a forced synchronous layout. When you need to make a DOM update, schedule the
  update via a `requestAnimationFrame` callback. This also ensures that the time needed for the DOM
  changes doesn't cross the browser deadline prescribed to the `requestIdleCallback`.

- **What is a forced synchronous layout?** This is an extra layout operation that is caused when
  there are DOM changes from JavaScript and those changes are subsequently read. The extra layout
  operation is necessary because the DOM changes from JS invalidate the layout calculation from the
  previous frame.

### Useful references

- https://developers.google.com/web/updates/2015/08/using-requestidlecallback
- https://developers.google.com/web/fundamentals/performance/rendering/avoid-large-complex-layouts-and-layout-thrashing#avoid_forced_synchronous_layouts