---
layout: post
title: Why can't Array return my deleted selections?
categories:
- weirich
tags:
- array
- delete
- enumerable
- reject
- ruby
- select
---
In Ruby, it's not uncommon (some would say it's common...they'd be right) to need to select from an array.  For example:
```ruby
def select_even_numbers
  [1,2,3,4].select { |num| (num % 2) == 0 }
end
# => [2,4]
```
This is all well and good, but what happens when I want to modify the existing array and return my deleted results? Well, ```ruby Array#delete ``` returns the deleted object, so we'd think it would look something like this:
```ruby
def select_even_numbers!
  [1,2,3,4].delete { |num| (num % 2) != 0 }
end
# => ArgumentError: wrong number of arguments given
```
But we'd be wrong. Delete doesn't take a block to evaluate when deleting elements. Delete takes strictly one argument and compares each element in the array to see if it matches then deletes and returns that single element. Let's keep looking through the enumerable docs. A ha! There's ```ruby Array#delete_if ``` but that's essentially an alias for ```ruby Array#reject ``` (reject returns nil if the array is unchanged, whereas delete_if returns the unchanged array).  So reject just does the same thing as select, but where the block evaluates to false instead of true. What recourse do we have now? It seems that Array has failed us.  It's time to go Stack Overflow diving (I won't bore you by copy-pasting the results, feel free to take a look).  I see a lot of suggestions that we monkey-patch Array to add an ```ruby Array#extract ``` method.  This is a good enough solution for a small project, but if you work at a company chances are monkey-patching is frowned upon unless absolutely necessary.  So let's convert that monkey-patch into a plain Ruby method.
```ruby
def extract_from_array!(array, &block)
  extracted = array.select(&block)
  array.reject!(&block)
  extracted
end

extract_from_array!([1,2,3,4]) do |num|
  (num % 2) != 0
end
# => [1,3]
```
This solution is fine, but defining this method into a helper module and including it anywhere we need to extract some methods is a hassle.  Our last hope is ```ruby Enumerable#partition ``` which doesn't modify the array in place, but rather returns two new arrays.
```ruby
[1,2,3,4].partition { |num| (num % 2) != 0 }
# => [[1,3], [2,4]]
```
Hopefully one of these will be the correct solution for your project, or if you have a better solution feel free to leave it in the comments.
