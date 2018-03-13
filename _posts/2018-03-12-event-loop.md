---
layout: post
title: "[Notes]: JavaScript Event Loop"
description: "Notes on the JavaScript Event Loop"
tags: notes event-loop browser javascript
---

## Prelude

This is gonna be a long one lol...There's a lot to know about the event loop.

## Actual Notes :D

- `setTimeout(ms, callback)` doesn't work how you would intuitively expect it to. The JavaScript
  main thread doesn't just yield back to the `setTimeout` callback exactly after the specified `ms`.
  It must wait for a "turn" of the event loop in addition to the specified timeout. So really, you
  should think of the timeout you specify as the minimum time that `setTimeout` will sleep for.

- After the timeout, the callback you specify is **queued as a task** to be executed by the main
  thread. It is common to hear that JavaScript is single-threaded, but this is not entirely
  accurate. Asynchronous tasks are still run in parallel on separate threads so that the main
  thread is not blocked from doing other things like rendering. The single-thread notion just comes
  from the concurrency model utilizing a *main* thread as a means to process results from the async
  threads.

- The JS specification allows for browsers to implement multiple queues for different things. This
  allows browser implementers to have some leeway in how they implement the spec. In general, you
  can conceptualize that there will be a (1) main task queue, (2) a micro-task queue, and (3) an
  animation callback queue. Microtasks and animation callbacks are described below.

- Take the following code block as an example:

  ```javascript
  setTimeout(1000, cb1)
  setTimeout(1000, cb2)
  ```
  - The event loop will proceess the callbacks like so:
    1. Execute the statements synchronously in one "turn" of the event loop
    1. Callbacks are queued in the task queue after 1000 ms
    1. cb1 is executed in the current turn
    1. cb2 is executed in the subsequent turn

  - The key takeaway is that tasks are executed one at a time, i.e. one task per turn of the event
  loop

### `requestAnimationFrame`

- So the previous is a summary of the task queue portion of the event loop...a separate part is the
  rendering logic

- When you want to run code with the render steps, you would use a [requestAnimationFrame]({{site.url}}/rAF-notes)
  callback. Conceptually, the render steps are on the complete other side of the task queues.
  Visually...see below:
  <figure>
    <a href="https://vimeo.com/254947206">
      <img src="{{site.url}}/images/archibald-event-loop.png" alt="">
    </a>
    <figcaption>
      <a
        href="https://vimeo.com/254947206"
        title="Event loop screen capture from Jake Archibald's talk The Event Loop"
      >
        Event loop screen capture from Jake Archibald's talk The Event Loop.
      </a>
    </figcaption>
  </figure>

- So far what we have is, on a turn of the event loop **where a render is necessary**:
  1. `rAF` callbacks are executed right at the start of the frame
  1. Layout and paint is performed
  1. A task from the task queue will be processed

- "**where a render is necessary**" is an important note because the browser doesn't need to render
  on every turn of the event loop. Many turns can go by before a render is deemed necessary.

  - Renders are only necessary when there are actual style and layout changes. Render only needs to
    happen when the monitor refreshes--often, 60fps. There won't be any rendering if the browser tab
    is not visible.

- As of this writing (3-15-2018), Safari and Edge(?) put `requestAnimationFrame` after style and
  layout occur which violates the spec.

### Compositor thread

- Though not in spec, most browsers will offload the actual rendering of pixels to what is known as
  the compositor thread. Pixel data is computed during the paint step and then it is sent to the
  compositor thread as a texture to actually be written onto the screen.

- "The compositor can only take already drawn bitmaps and move them around" - Jake Archibald

- This also means that certain CSS properties (e.g. transform) can be entirely offloaded to the
  compositor thread and not be affected by what happens on event loop. (Stay tuned--notes on this
  coming right up).

### Microtasks

- Used for promise callbacks, but were originally designed to be used for MutationObservers.

- Microtasks happen at the end of a task or when the JavaScript stack goes from non-empty to empty.
  This means that microtasks can run **during** a task or during `rAF`'s.

- Because microtasks are run after the stack empties, you are guaranteed that your promise callback
  is not running while other JavaScript is running.

- Taking a look at how the task queues empty:

  - The task queue is processed one per turn of the event loop and new tasks queued by the current
    task go to the end wait for the event loop.

  - During render, the animation task queue will be emptied except for any new callbacks that get
    scheduled.

  - When the JS stack empties, all microtasks in the queue will be processed as well as any
    additional microtasks that are added by pre-existing microtasks. This means that you can have
    an infinite loop of microtasks.

## Useful references

- [SmashingConf London — Jake Archibald on ‘The Event Loop’](https://vimeo.com/254947206)
- [Philip Roberts: What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
- [Tasks, microtasks, queues and schedules - Jake Archibald](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)