---
title: Hash Default Values
date: 2015-04-29
categories: ruby
layout: post
---

Going through challenges on [Exercism](http://exercism.io/),
I've seen many cases where hashes can be initialized with a default value
to remove the logic needed for "if it is set, do this, else do that",
but creating that default value can be a little confusing sometimes.

### When to use Hash.new(default)

Running `Hash.new(default_value)` will make a hash that instead of
returning `nil` when you lookup a key that doesn't exist,
it will return `default_value`.
This can be useful for something like counting occurrences in an array.

If I have some array of letters `[:a, :a, :a, :b, :b]` and want a count of every
element in the array like `{ a: 3, b: 2 }`,
I can calculate this with something like:

```ruby
[:a, :a, :a, :b, :b].each_with_object({}) do |letter, counts|
  counts[letter] ||= 0
  counts[letter] += 1
end
```

In this case, we can remove the entire `counts[letter] ||= 0` line by using `Hash.new(0)`

```ruby
[:a, :a, :a, :b, :b].each_with_object(Hash.new(0)) do |letter, counts|
  counts[letter] += 1
end
```

### When not to use Hash.new(default)

As helpful as the `default_value` is, it doesn't always work as you predict it would.
Our hash will always the exact same that you give it inside the default.
For things like the counts example above,
this doesn't matter because we're reseting the value for a key whenever we add something.
However, if we're trying to change the default object rather than reset the hash value,
we start getting strange behavior.

```ruby
default_array = []
hash = Hash.new(default_array)
hash[:some_key] << 7
hash[:some_other_key] << 8

hash           # => {}
default_array  # => [7, 8]
```

Because `hash[:some_key]` is just giving us `default_array` but never setting that key
in the hash, we're changing that default object but it's still doing nothing to the hash.

I'm just going to leave the following code, and you can look over it if you like,
but I don't suggest this.
The difference here is we're adding an array to an empty array
and just creating more arrays every time we do anything.

```ruby
default_array = []
hash = Hash.new(default_array)
hash[:some_key] += [7]
hash[:some_other_key] += [8]

hash           # => {:some_key=>[7], :some_other_key=>[8]}
default_array  # => []
```

The "REAL" way (the one I suggest) you try to get at this problem is use `Hash.new(&block)`.

This works like the following:

```ruby
Hash.new { |the_hash, key_that_was_called_but_not_set| "do something" }

hash = Hash.new { |h, key| h[key] = [] }

hash[:some_key] << 7
hash[:some_other_key] << 8

hash  # => {:some_key=>[7], :some_other_key=>[8]}
```

When using the block format, the hash will run the block whenever a key that is not set
has been called. In this case, we want the new key to be set to an empty array right away.
This way we have our keys set and to a different empty array every time we call a new one.

### Conclusion

If you're going to use default hash values, pretty much use `Hash.new(value)`
for anything you're using `+=` for (numbers, strings, etc), then `Hash.new { something }` for everything else.
