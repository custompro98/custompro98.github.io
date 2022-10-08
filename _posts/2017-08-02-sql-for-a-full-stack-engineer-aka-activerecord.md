---
layout: post
title: SQL for a Full Stack Engineer (aka ActiveRecord)
categories:
- weirich
tags:
- activerecord
- n+1
- rails
- ruby
- sql
---
Many web applications are built by engineers without experience in at least one layer of the stack.  Unfortunately, it seems the database layer is that layer. But who needs to know how databases work when ActiveRecord abstracts away all of your data access logic, right?  I'm sure we've all heard that ActiveRecord is great for a small to midsize Rails project.  For the most part, this is true.  ActiveRecord handles all of our database calls and that means no more writing database adapters, no more looking directly at your table schema, and certainly no more writing queries for the most basic requests.  ActiveRecord turns writing a bunch of these
```sql
SELECT *
FROM users
```
into these
```ruby
User.all
```
That is so much cleaner, and it reads like Ruby code for us non SQL developers.  Win-win, right? Wrong. Once our new web application takes off and your data store grows, we start to see some of ActiveRecord's downfalls.  Maybe we didn't have the foresight to limit the number of users that would appear on an admin page that lists registered Users.  Each time we have to render a User to the page, a small cost is incurred.  Have enough users, and the collective cost is way too much for our page to handle and it doesn't load (or worse, it takes more than a few seconds :gasp:). That one might be a stretch, we obviously thought of paginating.  But we also thought to link Users to a set of Books they're interested in reading.  Some of our Users have hundreds of Books in their to read list.  When a User is clicked on our admin page, we want to see the users read list.  But we're smart and don't want to fire off an AJAX request when we click, so we use ActiveRecord associations to to grab the Users books.
```ruby
- Users.all.each do
  = user.books.each do
  = book.title
  = book.author.name
- end
```
(don't mind my use of Slim instead of ERB, - means the code is interpreted but not rendered and = means the output is rendered) Doesn't this look harmless? It's not.  Even with 10 Users and each one having a read list of 20 Books, that's 200 elements being rendered onto the page.  But our application blew up, so imagine we have 1,000 Users - that's 20,000 Books being rendered.  Let's look beyond the rendering costs (since this post is about why ActiveRecord can be dangerous as an opaque SQL replacement), ActiveRecord doesn't eager load by default. This means we're actually making 1 additional database (and inherently network) calls per User on the page and another per Book on the page.  21,000 database calls is going to cause some serious slowdown.  Luckily ActiveRecord offers us some tools we need to know to use to help out.
```ruby
class User < ActiveRecord::Base
  has_many :books
end
```
```ruby
class Book < ActiveRecord::Base
  has_one :author
end
```
```ruby
- Users.includes(book: [:author]).each do
  - user.books.each do
    = book.title
    = book.author.name
- end
```
ActiveRecord now loads all of our Users with each of their Books and the respective Author, cutting down the number of database calls tremendously. ActiveRecord isn't a substitute for knowing how databases work.  This post touches on one of a whole slew of common database problems that ActiveRecord can hide from us.  ActiveRecord will get us off the ground, but it's up to us to either find someone who knows about the database layer or pick it up ourselves before our application scales up.
