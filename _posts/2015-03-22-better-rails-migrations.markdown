---
layout: post
title: "Better Rails Migrations"
author: freddyrangel
modified:
categories: 
excerpt: "Rails Migrations Guidelines & Tips"
tags: []
image:
  feature:
date: 2015-03-22T19:20:06-07:00
---

<p>
  When working on a smaller Rails app, I've been able to get away from having to
  put too much thought into migrations. Once I started working on more established
  apps, I started to feel the some pain points that shouldn't have been happening.
  Here's some guidelines I recommend.
</p>

### Don't modify an existing migration

<p>
  If you've worked on any Rails app that went into production, you probably
  already know this. If you're working on your own branch and you haven't pushed
  your code yet, you <i>could</i> get away with it, but it's almost always a bad idea.
  If you pushed your code to another environment like Heroku, those changes are
  just going to be ignored because those migrations have already been run.
</p>

<p>
  There are exceptions to every rule though. Once, I had a case where I was taking
  over a legacy project where we had to migrate the app from a hosted server to
  Heroku, and once of the prior migrations violated a guideline I'll bring up
  later, so I had to go in and modify the prior migration.
</p>

### Don't interact directly with models in migrations

{% highlight ruby %}
  class AddDescriptionToPosts < ActiveRecord::Migration
    def up
      add_column :posts, :description, :text
      Post.find_each { |post| post.update_attribute(:description, 'NA') }
      change_column :posts, :description, :text, null: false
    end
  
    def down
      remove_column :posts, :description
    end
  end
{% endhighlight %}

<p>
  Here were adding a <code>description</code> column to <code>posts</code> and then making sure it's
  never null. The problem with this code is with </code>Post</code>. Later, the name of the
  model can change. Sad things will occur. How do we fix this?
</p>

### Do use custom SQL statements

{% highlight ruby %}
  class AddDescriptionToPosts < ActiveRecord::Migration
    def up
      add_column :posts, :description, :text
      db.execute "UPDATE posts SET description = 'NA'"
      change_column :posts, :description, :text, null: false
    end

    def down
      remove_column :posts, description:
    end

    private

    def db
      ActiveRecord::Base.connection
    end
  end
{% endhighlight %}

### Do use migration models

<p>
  This is my preferred method because it involves pure Ruby, and if the migration
  is complicated then custom SQL can get a little hairy.
</p>

{% highlight ruby %}
  class AddDescriptionToPosts < ActiveRecord::Migration
    class Post < ActiveRecord::Base; end
    def up
      add_column :posts, :description, :text
      Post.find_each { |post| post.update_attribute(:description, 'NA') }
      change_column :posts, :description, :text, null: false
      Post.reset_column_information
    end

    def down
      remove_column :posts, :description
    end
  end
{% endhighlight %}
