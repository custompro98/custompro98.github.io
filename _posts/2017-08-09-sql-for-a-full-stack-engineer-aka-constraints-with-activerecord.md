---
layout: post
title: SQL for a Full Stack Engineer (aka foreign keys with ActiveRecord)
categories:
- weirich
tags:
- activerecord
- cascade
- delete
- foreign
- keys
- rails
- ruby
- sql
---
Last time we focused on how the database layer is often forgotten by web engineers in their quest to build out the latest and greatest.  The problems that can arise from that mentality certainly aren't few, so we're going to talk about another today.
Another common pitfall is the exclusion of foreign keys in a schema.  I'm sure any database-centric developers are cringing right now, but it happens to many of us.  There are many types of constraints that can be added to the database layer, but let's focus on just foreign keys.
Foreign keys are often forgotten <em>because</em> ActiveRecord abstracts away all our data access. Why would we bother with the verbose and cumbersome SQL required to generate a foreign key when we can simply say:
```ruby
class ModelA < ActiveRecord::Base
  belongs_to :model_b
end

class ModelB < ActiveRecord::Base
  has_a :model_a, dependent: destroy
end
```
That's so easy and we could simply do
```ruby
  ModelB.find_by_id(1).destroy
```
and the associated ModelA is taken care of for us, much like using CASCADE in SQL.  So what's the benefit of using a foreign key if ActiveRecord even abstracts away CASCADE-ing deletes?
There may not be a noticeable benefit when working on a small enough application where our domain objects are only directly associated, but as soon as we are in a situation where we have
```ruby
class ModelA < ActiveRecord::Base
  has_many :model_c, dependent: :destroy
  has_many :model_b, through: :model_c
end

class ModelB < ActiveRecord::Base
  has_many :model_c, dependent: :destroy
end

class ModelC < ActiveRecord::Base
  belongs_to :model_a
  belongs_to :model_b
end
```
Calling
```ruby
  ModelB.find_by_id(1).destroy
```
now only destroys any associated ModelC records leaving us an orphaned ModelA.
We're lucky yet again that ActiveRecord provides a great set of tools to help manage our fear of writing SQL.  Going back to the simpler ModelA and ModelB example, ActiveRecord offers
```ruby
add_foreign_key :table_a, :table_b
```
to quickly add a foreign key of table_b_id to table_a.
I'm going to say this again since it's so important: ActiveRecord isn't a substitute for knowing how databases work.  This post touches on one more of a whole slew of common database problems that ActiveRecord can hide form us.  ActiveRecord will get us off the ground, but it's up to us to either find someone who knows about the database layer or pick it up ourselves before our application scales up.
