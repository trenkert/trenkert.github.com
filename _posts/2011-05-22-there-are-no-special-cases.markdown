---
layout: post
title: "There are no special cases"
categories:
  - design
---

A lot of messy code is caused by new requirements breaking the assumptions of earlier implementations. There's often a noticeable distinction, a fracture, between what the existing requirements were and what's been added. Sometimes this fracture is jarring: these are areas where the new requirements have been "tacked on" to the existing code. There could be lots of conditional statements changing things in very specific circumstances. Chances are every object talks to everything else to gather the information required to fulfill the request.

Tacking on requirements is fast and easy at first. The problem happens after months or years: the same code has been extended in many directions, the initial code has lost its purpose. Change becomes exponentially harder. This makes it easier to justify just tacking on the next requirement; refactoring the whole thing will take too long.

It's really easy to blame customers for this: "the code was clean, but then the requirements changed." There seems to be a tendency towards treating new features as special cases to be draped over the existing code as fast as possible, rather than an integral part of the overall design. It's almost an afterthought. 

This problem can be traced back to a misunderstanding of what "simple" code means. When adding a feature the goal isn't getting from A to B in the simplest way. The goal is to make the code at B look as simple as possible. You're not done with a feature when all your tests pass, you're done when all your tests pass _and_ the code looks as if it was designed from the beginning with your new feature in mind.

How can we achieve this? We can start by refactoring our code mercilessly. The assumptions that were made previously aren't set in stone, they need to be constantly revised. But how do we know what we should be refactoring towards? Recently, I've tried to approach my code with one idea at the forefront: **there are no special cases**. There are only cases the code can handle and those it can't handle yet. 

This could just be considered a different way of stating the Single Responsibility Principle. You can often reframe the new specific case in a more general way that an object can cope with and push the differences into the caller. Sometimes the caller shouldn't be responsible for this feature. Again, find the generalisation that this object can deal with and push the special case up again. The more you can push the special cases up and out of your objects, the cleaner they will feel.

Obviously this guideline has its tradeoffs. Sometimes it will take an inordinate amount of time to make the change, more time than you currently have. Maybe you can't see the right abstraction just yet. Sometimes the new feature may really be a special case and you can't justify the time it would take to make it fit your domain model. But if you approach every new requirement as if it's not a special case it may help you find the steps required to get your code back into shape.
