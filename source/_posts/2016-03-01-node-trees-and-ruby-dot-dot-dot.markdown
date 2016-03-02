---
layout: post
title: "Node Trees and Ruby..."
date: 2016-03-01 20:44:34 -0500
comments: true
categories:
---
...Don’t really mix. I started playing around today trying to build a node tree after getting an interview question about how I would build a spell-checker (short answer: node trees) and became curious about how one would go about building the tree itself from scratch. The result I came up with is certainly not a perfect node tree, but it does build a structure that could be traversed letter by letter to check if a given word is in the dictionary. If you’re interested in actually implementing a node-based tree data structure in your Ruby program, I’d recommend checking out the [RubyTree gem](https://github.com/evolve75/RubyTree).

##THE CODE##

First things first: build the `Node` class:

```ruby
class Node
  attr_reader :letter
  attr_accessor :word_end, :children

  def initialize(letter)
    @letter = letter
    @word_end = false
    @children = {}
  end
end
```
I initialize each `Node` based on the letter it’s tracking. The `@word_end` variable keeps track of whether a given node represents a valid end to a word (so that if you were traversing a path that included the word “hearts” the following would represent valid end points for your word:  “e”, “r”, “t”, and “s”). This defaults to `false` and is overridden if the node can be the last letter in a word. Finally, the `@children` hash tracks all of this node’s child nodes.


To actually build the node tree, I built a `DictionaryTree` class which would initializes with an array of words and then converts that array into a node tree:
```ruby
class DictionaryTree
  attr_reader :word_array
  attr_accessor :word_tree

  def initialize(word_array)
    @word_array = word_array
    @word_tree = {}
    make_tree
  end

  def make_tree
    word_array.each do |word|
      current_hash = word_tree
      i = 0
      while i < word.length
        if !current_hash.has_key?(word[i])
          current_hash[word[i]] = Node.new(word[i])
        end
        current_hash[word[i]].word_end = true if i+1 == word.length
        current_hash = current_hash[word[i]].children
        i += 1
      end
    end
  end
end
```
Let’s break down what’s happening here. In order to build the tree, I iterate through each word and compare it to the pre-existing `@word_tree`. Each letter is checked by index, and is added to the hash if the node for that letter does not already exist. The method then checks to see if the letter is the last in the word, and sets the node’s `@word_end` value to true if it is. Finally, the `current_hash` is set to be not the `@word_tree`, but the hash containing the children of the current node.

Here’s an example tree for `DictionaryTree.new(["hats", "hat", "her", "cat"])`

```
{"h"=>#<Node:0x007f8fbb83dc48
  @letter="h",
  @word_end=false,
  @children={
    "a"=>#<Node:0x007f8fbb83d770
      @letter="a",
      @word_end=false,
      @children={
        "t"=>#<Node:0x007f8fbb83d388
        @letter="t",
        @word_end=true,
        @children={
          "s"=>#<Node:0x007f8fbb83d068
          @letter="s",
          @word_end=true,
          @children={}>}>}>,
    "e"=>#<Node:0x007f8fbb83ca28
    @letter="e",
    @word_end=false,
    @children={
      "r"=>#<Node:0x007f8fbb83c730
        @letter="r",
        @word_end=true,
        @children={}>}>}>,
"c"=>#<Node:0x007f8fbb83c410
  @letter="c",
  @word_end=false,
  @children={
    "a"=>#<Node:0x007f8fbb83c0f0
      @letter="a",
      @word_end=false,
      @children={
        "t"=>#<Node:0x007f8fbb837d70
          @letter="t",
          @word_end=true,
          @children={}>}>}>}
```
Of course, writing a method like this in Ruby leaves something to be desired. Really, a data tree like this should be written with pointers, not as a nested hash. But, for a quick Ruby experiment, this did create a functional equivalent of a node tree that could be used as a spell-checker. For more info on how spelling *correction* works, I highly recommend [this blog](https://engineerbyday.wordpress.com/2012/01/30/how-spell-checkers-work-part-1/).
