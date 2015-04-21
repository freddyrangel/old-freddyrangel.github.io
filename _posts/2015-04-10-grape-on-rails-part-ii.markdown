---
layout: post
title: "Grape on Rails: Part II"
author: freddyrangel
modified:
categories:
comments: true
excerpt: "Authentication for a Grape JSON API"
tags: []
comments: true
image:
  feature:
date: 2015-04-10T21:57:03-07:00
---

<p>
  In the first part of this tutorial, we talked about how to initially set up a
  Grape application inside of a Rails app. In this post, we're going to go over
  how to get a simple authentication solution setup in a Grape app.
</p>

<p>
  If you're not used to working with JSON APIs, then authentication for an API is
  going to be a little foreign to you. An API should be truly stateless, so we're
  not doing anything involving cookies or sessions. There are many ways to
  handle authorization for an API. For now, we're going to hangle authorization
  by generating an authorization token and requiring that token for every
  request.
</p>

<p>
* Note: source code for all examples can be found here: <a
  href="https://github.com/freddyrangel/grape-on-rails-tutorial">Grape on Rails
Tutorial</a>
</p>

<p>
  We're going to take a BDD approach to testing by writing a higher level test
  first before diving down to unit tests. Here's an example request spec.
</p>

{% highlight ruby %}
#/spec/api/example_app/login_request_spec.rb
require 'rails_helper'

RSpec.describe 'Login request', type: :request do
  let!(:user) { create(:user, :with_token) }
  let!(:uri)  { '/api/v1/login' }

  describe 'User login' do
    let(:request) do
      post uri, request_params, mime_json
    end

    before(:each) { request }

    context 'happy path' do
      let(:request_params) { { user: { email: user.email, password: user.password } }.to_json }
      it { expect(response.status).to eq(201) }

      context 'request body data' do
        it { expect(parse_json(response.body)[:user][:authentication_token]).to_not be_nil }
        it { expect(parse_json(response.body)[:user][:email]).to eq(user.email) }
      end
    end

    context 'wrong email or password' do
      let(:request_params) { { user: { email: 'lolol', password: 'lmao' } }.to_json }
      it { expect(response.status).to eq(400) }
    end
  end
end
{% endhighlight %}

<p>
  OK, now we have a benchmark letting us know if we set all this up correctly. Let's start adding some code.
</p>

{% highlight ruby %}
#Gemfile
gem 'bcrypt', '~> 3.1.7'

group :test do
  gem 'factory_girl_rails'
  gem 'shoulda-matchers', require: false
end
{% endhighlight %}

<p>
  Let's install these gems:
</p>

{% highlight bash %}
bundle install
{% endhighlight %}

<p>
  We're going to user a spec helper to make our request specs less verbose.
</p>

{% highlight ruby %}
#spec/support/helpers/request_helpers.rb
module RequestHelpers
  def parse_json(body)
    JSON.parse(body, symbolize_names: true)
  end

  def mime_json
    { 'Accept' => Mime::JSON.to_s, 'Content-Type' => Mime::JSON.to_s }
  end

  def authorization_header(token)
    header = mime_json
    header['Authorization'] = token
    header
  end
end
{% endhighlight %}

<p>
  Now we need to configure our rails_helper to play nice with our specs.
</p>

{% highlight ruby %}
#spec/rails_helper.rb
require 'shoulda/matchers'
Dir[Rails.root.join('spec/support/**/*.rb')].each { |f| require f }

RSpec.configure do |config|
  config.include FactoryGirl::Syntax::Methods
  config.include RequestHelpers
end
{% endhighlight %}

<p>
  We're going to need a user factory.
</p>

{% highlight ruby %}
#spec/factories/users.rb
FactoryGirl.define do
  factory :user do
    sequence(:email) { |u| "user_#{u}@example.com" }
    password 'password'
    password_confirmation 'password'

    trait :with_token do
      sequence(:authentication_token) { |u| "random_token_#{u}" }
    end
  end
end
{% endhighlight %}

<p>
  At this point we're ready to set up our user model.
</p>

{% highlight bash %}
rails g model user email password_digest authentication_token
rake db:migrate
{% endhighlight %}


<p>
  Let's update that user spec.
</p>

{% highlight ruby %}
#spec/models/user_spec.rb
require 'rails_helper'

RSpec.describe User, type: :model do
  describe 'Active Record' do

    context 'validations' do
      it { is_expected.to have_secure_password }
      it { is_expected.to validate_presence_of(:email) }
      it { is_expected.to validate_uniqueness_of(:email) }
    end
  end

  describe '#generate_authentication_token' do
    let!(:user) { create(:user) }

    it 'creates an authentication token' do
      user.generate_authentication_token!
      expect(user.authentication_token).to_not be_nil
    end

    it 'should create new token each time' do
      user.generate_authentication_token!
      first_token = user.reload.authentication_token
      user.generate_authentication_token!
      second_token = user.reload.authentication_token

      expect(second_token == first_token).to eq(false)
    end
  end
end
{% endhighlight %}

<p>
  Now we're ready to add some code to make our specs pass.
</p>


{% highlight ruby %}
#app/models/user.rb
class User < ActiveRecord::Base
  validates :email, presence: true, uniqueness: true

  has_secure_password

  def self.authorize!(env)
    token = sanitize_token(env)
    token.present? ? find_by_authentication_token(token) : false
  end

  def generate_authentication_token!
    begin
      self.authentication_token = SecureRandom.hex
    end while self.class.exists?(authentication_token: authentication_token)
    self.save!
  end

  def login_json_info
    as_json(only: [:id, :name, :email, :authentication_token])
  end

  private

  def self.sanitize_token(env)
    token = env['HTTP_AUTHORIZATION']
    token.gsub!(/\A"|"\Z/, '') if token.present?
    token
  end
end
{% endhighlight %}


{% highlight ruby %}
#app/api/example_app/api.rb
class ExampleApp::API < Grape::API
  format :json
  prefix :api

  version 'v1', using: :path , vendor: 'skyrise' do
    before { Rails.logger.info "Request Body: #{request.body.read}" }

    get '/' do
      status 200
    end

    post '/login' do
      user = User.find_by_email(params[:user][:email])
      if user && user.authenticate(params[:user][:password])
        user.generate_authentication_token!
        { user: user.login_json_info }
      else
        status 400
      end
    end
  end
end
{% endhighlight %}
