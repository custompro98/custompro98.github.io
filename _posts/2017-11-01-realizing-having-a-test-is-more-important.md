---
layout: post
title: Realizing Having a Test is More Important
categories:
- weirich
tags:
- testing rspec helper module controller test session dependency injection
---
A few weeks back at work, I got around to using the venerable `SessionPersistenceHelper` which did a decent job abstracting away the work needed to persist checkbox selections in the session (aptly named, I'm surprised). All that's needed are a few business rule methods to be defined on your new controller (how many selections are allowed, what those selections are, how to count them, etc.) and thank-you-very-much that page now persists a user's checkbox selections. Easy enough. 

A week or so later, I had to use the same module. My sprint wasn't as hectic, so I took a minute to actually look at what the helper was doing. Grab those selections defined in your new controller, grab the previous selections, smash them together, put it in the session. Got it, looks like the module name describes what it does pretty well. Naturally in a big-ole legacy application, there's going to be exceptions written into this god-helper because it was just too darn convenient to use this round peg in someone's slightly oblong hole problem. I was curious where these exceptions were used in the application so I opened up the `spec` directory to find the tests.

Huh, no tests. I looked back at the module to see if there was any *reason* why there might not be tests.  There it was looking me in the face -

```
render json: {count: session[session_id].count, ids: session[session_id]}, status: :ok`
```

In the timeless battle between those who believe in controller testing versus those who don't, this module was forgotten.  And living now in a world where controller tests are despised (read: we have slow, flakey builds), I was not in a position to add a new controller test.  But I couldn't just leave this module untested, it's used in almost every index page in the application!

Here's where I started thinking about what I had been reading in Working Effectively with Legacy Code.  I needed to get this module into a test harness.  Easy enough in Ruby.  Whip up a new spec file for the helper, include the module in the test so I can call its methods. Great.  

First (obvious) problem: I can't call the method directly because it renders at the end.

Solution: break the meat of the solution into a new method that the old one will call.  Now we have `store_selections_in_session` (don't want to mess up the routes that depend on this method name and `store_selections`. Great.  

Second (less obvious) problem: this private method has dependencies on methods defined in whichever controller includes this module and params being passed in via the network. Not very easy to test.

Solution: inject the dependencies of our new private method.  So now we have `store_selections_in_session` and `store_selections(new_ids, session_limit` where `new_ids` are the new selections as defined in the controller and `session_limit` is the session limit defined in the controller. Great.

Third (maybe more obvious than the last) problem: we can't access the session in this plain old Ruby method via the test.

Solution: remove the session dependency and let the old method named `store_selections_in_session` handle session manipulation since it would need to be in a controller test anyway.  So now we have `store_selections_in_session` which passes any dependencies into our new method and puts the result of the new method into the session and the tested method `combine_selections(stored_ids, new_ids, session_limit)` which now just manipulates two arrays and raises an error if their combined size passes `session_limit`.
Great.

Now there's a heavily trafficked piece of code that's been blindly used in nearly every part of the application that has tests around it to help verify functionality.  Making the codebase better a little bit at a time.  I would like to come back in the future and figure out a way to test this without testing a private method (which I know is big faux pas in the development world).
A quick sidebar: a major point in Working Effectively with Legacy Code is the requirement to have some sort of test in place before you can begin refactor work like this.  In the steps above, I omitted my use of scripted characterization tests that opened two of the pages with checkboxes, checked some values, refreshed the page, and verified the boxes were still checked.  I would not recommend doing the refactor work above without something similar at the minimum.
