---
title: "Going React Native"
description: "Leading a range of technological changes at Sense in the past year, I could see one constant requirement for us had been agility. Health…"
date: "2020-05-09T11:16:14.973Z"
categories: []
published: false
---

Leading a range of technological changes at Sense in the past year, I could see one constant requirement for us had been agility. Health care is a very dynamic sector requiring constant iterations and minor variations of similar ideas focussed for different markets.

To allow experimenting and building apps for multiple platforms like this, I had already architected a solution for context aware coaching that is flexible and allows us to do targeted coaching without worrying about the platform or the app on which the logic is working on. Having flexibility on coaching allows us to be quite dynamic but there were still some restrictions imposed on us by our development environment.

Having developed native applications for the past several years, I have researched on multiple promised solutions (Xamarin, Cordova, Titanium, Phone Gap) for hybrid apps over the years and have consistently been disappointed by some severely limiting point or the other.

All solutions for hybrid applications can be boiled down to using a third party language (Xamarin: C#, Rest: JS) that compiles and interprets layouts in a custom format (Xamarin: Native Layouts, Rest: HTML)

The 3 problems with this approach at the core:

-   Using a language that has a short learning curve and has a massive community of tooling to provide significant advantage over native development.

This is the one of the biggest problems I had with Xamarin. Its really well designed and has pretty decent performance too but the cost of porting every library, animation and SDK/tooling into .NET is just too high to justify the move. 

NOTE: My hands on experience with Xamarin was 2 years ago, since then it has evolved quite a bit and the community might have solved these problems

-   A custom layout format that is preferably already known or is re-usable on some other platforms and is as fluid as native layouts.

Learning a new lay-outing format when you can still only use it for mobile apps makes little sense. If you go this way then usually the advantage of learning the native lay-outing format far outweighs the advantage of learning a third abstraction of the mobile platforms.

-   And finally even if you learn the format, is it still fluid enough for a flawless mobile app experience? 

Almost all hybrid apps fail miserably on this end. The performance of interpreted JS is improving consistently on mobile browsers but the core problem of rendering HTML web views is inherently slow for fluid interactions to be possible.

---

These problems are heightened if you look at their implementation and the amount of control they take away from you as a native developer.

JS based frameworks define layouts as HTML templates which are parsed at run time and inserted as DOM elements into a web view. Using custom styling and custom JS views do allow you to render whatever you want, but the critical performance problem of each click/swipe/move event that you perform having to go through the touchscreen -> OS -> native -> JS loop.

This interaction is so inherently against the optimised nature of mobile app lifecycles that it is almost impossible to have extremely fluid touch interaction unless the OS allows some special methods for this in web apps.

---

Similar to the above problem another key limitation comes from animations. In native applications there is always dedicated UI thread that performs all rendering and is best not blocked by any complex computation. This allows rendering to go by without jitters while the app does stuff in the background. 

This also allows us to animate things flawlessly as we perform our animation logic either in the render loop directly as its usually simple arithmetic or even with background animations, we are modifying render parameters that are instantly picked up by the render thread on the next run.

Animating with JS makes things more complex, as the rendering is still done on the UI thread but the logic for the animation is done in JS which is usually running on a JS environment running in a background thread and doing all JS executions in this loop. Firstly, this will inherently stall your animation loop as your business logic and all other logic also has to run in the same block of code as your animation. Secondly, even with optimisations when you allow JS animations to get higher priority and do code execution with higher priority, it will still need to copy the new data from the JS environment to the native environment where its finally picked up by he main UI thread.

During all this logic, the UI thread usually has already missed a few runs without getting any new data for your animation. This results in choppy animations and makes it extremely hard to have multiple fluid animations in your app.

---

In my experience with hybrid frameworks I always saw such inherent problems putting unreasonable constraints on development, that was until recently with React Native blowing past all the problems above.

React Native, as explainable by its name, is based on the React framework and was built by the team at Facebook. Just looking at their github you can see the framework must be doing something right, having almost 50K stars and an immensely active community.

I have been keeping an eye on react for the past couple of years since it was announced and I have always been sceptical of such frameworks to be production ready. At sense having the need for a flexible and fast development environment, I worked with our key JS developer who is a React ninja and took out 2 weeks in January 2017 to build an example app with React Native.

Having been blown away by the performance and speed of the framework and seeing that all the low level technical control I needed was there (it specially helps as the project is open source), we decided to make the jump and port all our application development to React Native.

5 months into this, we are recently launched our first product with React Native and are gearing up to make all our products and projects with React Native.

I have been a sideline spectator for the teams responsible for developing the actual applications and having seen the codebase evolve and also experimented in high gear for 2 weeks, these are my take aways from React Native at this stage.

Pros

1.  Extremely fast iterations

This is so huge for native developers, specially android, that it is feels like you just got an upgrade from a Volvo to a Tesla. Hot loading code on save cuts of hours of compile time wait and makes you far more productive.

2\. Complete control for native

React Native projects are inherently native projects that start by loading a bridge that hijacks your application and loads the code of your application from a JS file. This means that your project is still native and you have control for the it. You want to add the Health Kit permission in your iOS app, that still going to be the same process of adding items into the plist. No having to go through obscure documentation of a custom framework.

3\. Speed

React Native does something very different from other frameworks. It renders your component as an actual native component. This means that your components aren’t part of a DOM that is rendered in a web view, but they are actual native components that can have their own touch based interactions. 

This makes them extremely performant and allows react to extend this concept allowing fluid animations to with its own [Animation API](https://facebook.github.io/react-native/docs/animations.html) which avoids the whole Native -> JS-> Native loop. Instead it can pass the animations into the Native code and high performance animation loops never have to interact with JS.

Even the layout-ing is done with Yoga, Facebooks open source flex-box like layout-ing engine written in C 

4\. Community

React Native has a massive community. With ~50K stars on Github and hundreds of stack overflow answers, you can get an answer to pretty much every problem you face.

Cons

1.  ‘Weird’ problems

React Native is hacking its way into the native eco system. It is a complex injection of components to seem native and depends on a bunch of tooling for itself. This means there are a LOT of points of failure and you will end up facing extremely ‘weird’ problems that are solved by rebuilding projects, cleaning NPM modules or using a different JS syntax.

2\. Breaking changes

Even though this has been considerably solved with the new React deployment strategy of a new release once a month, it is still a problem as things do change once a month and they can be breaking for existing code. You can expect a few days a month for pure maintenance and upgrade when working with React Native

---

In conclusion having a decent size of native developers, I was looking for ways to improve speed of development without loosing explicit control of native development that we require. In our experience React Native has been a great choice of the right balance of speed with control, even though this does mean everyone in the team has to learn a lot more (React, Redux, Typescript) but considering we will be building multiple products with this tool chain, it allows us to do knowledge sharing and build a unique knowledge base of React development at Sense.

As we move forward with development and learn and understand more about the eco system, I hope we will be able to contribute more back to the community and open source more for the React Native ecosystem.
