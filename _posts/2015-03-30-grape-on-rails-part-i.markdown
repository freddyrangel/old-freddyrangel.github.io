---
layout: post
title: "Grape on Rails: Part I"
author: freddyrangel
modified:
categories:
comments: true
excerpt: "Setting up a Grape App inside Rails"
tags: []
image:
  feature:
date: 2015-03-30T20:19:03-07:00
---


<p>
  Over the last few months I've had to build a number of JSON APIs for mobile
  and Javascript apps. Unfortunately, Rails' conventions really let us down
  when it comes to building APIs. What is needed is something that gives us
  better control over our API, while at the same time not giving up some of the
  useful Rails conventions that boost productivity.
</p>

<p>
  Enter Grape. It's a Ruby micro-framework for APIs, which can either be
  run on Rack or can be mounted inside of another framework like Rails or Sinatra.
  Its DSL is really clean and intuitive compared to Sinatra, and it's just a
  pleasure to work with. Plus, since it can be mounted inside of Rails, we can
  easily reach for Rails when we need it, or use libraries designed specifically
  to work with Rails. It requires a little setup, but we will walk through the
  basics, including writing a simple RSpec request spec to make sure everything
  is set up properly.
</p>

<p>
  Here I'm going to use an example Rails app, but consider using
  <a href="https://github.com/rails-api/rails-api">Rails::API</a>
  For now, let's just talk about just Grape.
</p>

### Setup

<p>
* Note: source code for all examples can be found here: <a
  href="https://github.com/freddyrangel/grape-on-rails-tutorial">Grape on Rails
Tutorial</a>
</p>

<p>
  We're going to fire up a new Rails app:
</p>

{% highlight ruby %}
  rails new example_app -T
{% endhighlight %}

<p>
  Next let's update the Gemfile:
</p>

{% highlight ruby %}
gem 'grape'
gem 'hashie-forbidden_attributes' # disables strong_params for Grape

group :development, :test do
  gem 'rspec-rails'
end
{% endhighlight %}

<p>
  Don't forget the usual:
</p>

{% highlight bash %}
bundle install
rake db:migrate
{% endhighlight %}

<p>
  Next let's set up RSpec:
</p>

{% highlight bash %}
rails g rspec:install
{% endhighlight %}

<p>
  Now let's write our spec:
</p>

{% highlight ruby %}
#spec/api/example_app/api_spec.rb
require 'rails_helper'

describe ExampleApp::API do
  describe "GET /api/v1" do
    before(:each) do
      ### Send get request with proper HTTP headers
      get '/api/v1',
        { 'Accept' => 'application/json', 'Content-Type' => 'application/json' }
    end

    it { expect(response.status).to eq(200) }
  end
end
{% endhighlight %}
<p>
  Now we should be getting:
</p>

{% highlight bash %}
uninitialized constant ExampleApp::API (NameError)
{% endhighlight %}
<p>
  Good, now we're getting the right kind of error. Let's create that class:
</p>

{% highlight ruby %}
#/app/api/example_app/api.rb
class ExampleApp::API < Grape::API
  format :json
  prefix :api

  version 'v1', using: :path , vendor: 'example_app' do
    before { Rails.logger.info "Request Body: #{request.body.read}" }

    get '/' do
      status 200
    end
  end
end
{% endhighlight %}
<p>
  Grape won't log anything by default, so we're adding a before block to make at
  least log the request body. You can log whatever you'd like. Now we're getting:
</p>

{% highlight bash %}
Randomized with seed 20573
F

Failures:

  1) ExampleApp::API GET /api/v1
    Failure/Error: get '/api/v1',
    ActionController::RoutingError:
      No route matches [GET] "/api/v1"
{% endhighlight %}
<p>
  So let's mount our Grape app onto Rails. First let's update our <code>routes.rb</code>
file:
</p>

{% highlight ruby %}
#/config/routes.rb
Rails.application.routes.draw do
  mount ExampleApp::API => '/'
end
{% endhighlight %}
<p>
  Now let's update <code>application.rb</code>:
</p>

{% highlight ruby %}
#config/application.rb
config.paths.add File.join('app', 'api'), glob: File.join('**', '*.rb')
config.autoload_paths += Dir[Rails.root.join('app', 'api', '*')]
{% endhighlight %}
<p>
  Grape does not automatically reload changes in development like Rails, so you
  have to create an initializer that will do that for us:
</p>

{% highlight ruby %}
#config/initializers/reload_api.rb
if Rails.env.development?
  ActiveSupport::Dependencies.explicitly_unloadable_constants << "ExampleApp::API"

  api_files = Dir[Rails.root.join('app', 'api', '**', '*.rb')]
  api_reloader = ActiveSupport::FileUpdateChecker.new(api_files) do
    Rails.application.reload_routes!
  end
  ActionDispatch::Callbacks.to_prepare do
    api_reloader.execute_if_updated
  end
end
{% endhighlight %}
<p>
  Now we run our spec and we're all green.
</p>

### CORS
<p>
  Chances are you're going to want to configure your app for CORS. Luckily this is
  really straight forward in Grape + Rails.
</p>

<p>
  Let's add <code>rack-cors</code> to our Gemfile
</p>

{% highlight ruby %}
gem 'rack-cors', require: 'rack/cors'
{% endhighlight %}

<p>
  And now lets use the middleware in <code>config.rb</code>
</p>

{% highlight ruby %}
require 'rack/cors'

use Rack::Cors do
  allow do
    origins '*'
    resource '*', headers: :any, methods: :get
  end
end

run ExampleApp::API
{% endhighlight %}
