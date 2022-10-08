---
layout: post
title: CSS Float is Magic
categories:
- weirich
tags:
- css
---
After adding the ability to view an application form in Footprints came making it look pretty.  With data in the database and the fields rendering with their answers came a `float: left` here and `float: right` there.  Everything looked like it was working.  There was a column on the left, the list of fields generally looked like a form, and all was good, right?

The next set of functionality was adding new fields and answers, easy enough, it should just all fit into our pretty new form.  Adding new worked just fine, awesome.  Then we introduced a bug where an answer wasn't being saved but all of our existing fields had already been deleted (to be replaced later).  Darn, but let's pretend we did that on purpose to give our new field adding ability a thorough manually test.  Click 'Add Field' aaaaaaaaaaand it's a small box in the top left of the page. Huh?  The cards should be floating to the right, that's what we told them to do!  At least that's what I thought.

As it turns out, when all of the fields disappeared, the div on the right side shrunk (duh). But what we didn't realize is the size of the div was what let the float work in the first place. Add enough fields and bingo-bango the cards are back where they belong.  That's not a great solution.

After a lot of digging around (read: asking the front end engineers how CSS actually worked), we learned not to trust float to reliably put data where it needs to go.  Instead, because we know how wide the sidebar is, we can add a margin-left and set its value using some math in CSS. A `margin-left: calc(23% + 32px);` later and the CSS no longer had any tricky floats in it.

Moral of this story?  Be explicit.
