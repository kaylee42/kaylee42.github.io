---
layout: post
title: "Reflexive Self Joins in Rails"
date: 2016-01-12 06:56:20 -0500
comments: true
categories:
---

##OMG SELF JOINS##
Simply put, a self-join table allows us to join  an instance of a class to another instance of the same class in a many-to-many relationship. At first glance this is not so different from the more traditional join table used for has-many-through relationships, however because you cannot have two columns in the join table named `user_id`, implementing self joins requires some finagling.

Perhaps the most common real-world example of needing a self-join table is in social networking, connecting users to users through friendships or following relationships. I set out to implement this in my recent Rails project, and became particularly interested in the different methods available for using self join tables to create reflexive relationships. This blog will walk through all of my steps, but will particularly focus on reflexivity options.


##MIGRATION##

Creating a self-join table migration can be done nearly the same as to any other join table. Similarly, you can use `t.references`. In this case, I am creating a friendships model where one column is for :user_id, and the other is for :friend_id, although both have the id of an instance of the User class.

The main difference here is the way that the foreign key is added to the friend column. Because the two columns cannot both be named :user_id, and Rails will not automatically know which groups of foreign keys to check unless we explicitly tell it, this foreign key needs to be added outside of the create_table method. The example of the code is below, but it will always follow the format `add_foreign_key :join_table, :model_table, column: :second_name_id`


```ruby
class CreateFriendships < ActiveRecord::Migration
  def change
    create_table :friendships do |t|
      t.references :user, index: true, foreign_key: true
      t.references :friend, index: true
      t.timestamps null: false
    end

    add_foreign_key :friendships, :users, column: :friend_id
  end
end
```

##MODELS##

Now that we’ve migrated our table correctly, it’s time to write our models! Our Friendship model looks pretty familiar - it belongs to a `:user` and a `:friend`. However, a `:friend` is actually an instance of the User class. Usually Rails uses magic to automatically associate these relationships using naming conventions, but we can override these conventions when needed. It’s even pretty easy! Just append `class_name “Class”` to any foreign key that’s not named after its class.

```ruby
class Friendship < ActiveRecord::Base
  belongs_to :user
  belongs_to :friend, class_name: "User"
end
```

The implementation in our model looks super familiar. No need to identify which class `:friends` belongs to, as that’s handled in the Friendship model.

```ruby
class User < ActiveRecord::Base
  has_many :friendships
  has_many :friends, through: :friendships
end
```

Now we can call `user.friends` to see what friends a given user has. However, as it stands our friends & friendships leave something to be desired, as only the user identified in the `:user_id` column of our friendships table. Wouldn’t it be create if these relationships were more reflexive?


##ONE DIRECTIONAL (Twitter Model)##

One way to create reflexive relationships is what I think of as the Twitter model. These relationships are one-directional; a user can follow someone, and they can have followers, but those relationships are not mutual. In order to set up this relationship, we just need to add two lines of code to our previous User model:

```ruby
class User < ActiveRecord::Base
  has_many :friendships
  has_many :friends, through: :friendships

  has_many :follows, class_name: "Friendship", foreign_key: "friend_id"
  has_many :followers, through: :follows, source: :user
end
```

What’s going on here? Let’s break it down. In line 5, we are defining a relationship to the friendships table that goes in the opposite direction. I’ve named it `:follows`, so the first thing we have to do is override Rails naming conventions to direct it to the proper class (Friendship), which we’ve seen before. But what about this `:foreign_key` business? Again, all we’re doing here is making explicit something that is implicit in Rails in order to override naming conventions. In a has_many relationship, it is assumed that the foreign key will be named after the class we’re in. However, in this case, we actually want to refer to the other side of that relationship - the `:friend_id`.

Line 6 is also fairly familiar - we just define `:followers` through the `:follows` relationship with just made. However, we again need to override our Rails naming conventions using source. This refers not to the name of the class, but the way the column is named in your join table - so just make it the opposite of whatever you defined as your foreign key in the previous line. Now we can call `user.followers` and see who has followed a give user!





##MUTUAL (Facebook Model)##

The second major way I found to create reflexive relationships is mutual - or what I think of as the Facebook model. In other words, if you friend someone, they also become friends with you. The easiest way I found to do this involved using callback methods.

```ruby
class Friendship < ActiveRecord::Base
  belongs_to :user
  belongs_to :friend, class_name: "User"

  after_create :create_inverse

  def create_inverse
    self.class.create(user_id: self.friend.id, friend_id: self.user.id)
  end

  validates_uniqueness_of :user_id, scope: :friend_id
end
```
This is actual pretty simply. All that’s happening is that a callback is being implemented in line 5 immediately after a new friendship is created. It simply calls the `:create_inverse` method which creates a second instance of the Friendship class with an inverse relationship to the original, guaranteeing that both users will be in each other’s collection of friends.

We avoid getting into an infinite creation loop because of uniqueness validation in line 11, which will validates for the uniqueness of `:user_id` as it relates to the scope of `:friend_id`.

Note: for some reason this works for me when implemented on my website, but does NOT work in the rails console.


##RESOURCES##
+ [Railscasts: Self-Referential Association](http://railscasts.com/episodes/163-self-referential-association?view=asciicast)
+ [Bi-Directional and Self-Referential Associations in Rails](http://collectiveidea.com/blog/archives/2015/07/30/bi-directional-and-self-referential-associations-in-rails/)
