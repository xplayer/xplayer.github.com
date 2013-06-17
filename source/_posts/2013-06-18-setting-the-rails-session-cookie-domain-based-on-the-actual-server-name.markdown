---
layout: post
title: "Setting the Rails session cookie domain based on the actual server name"
date: 2013-06-18 00:08
comments: true
published: false
sharing: true
categories: [Ruby, Rails]
---
Say you have single Rails codebase publishing different apps with different domain names, all sharing a common higher-level domain name, for example:

    store.mysite.com
    www.mysite.com
    read.mysite.com

and you want all these apps to share the same Rails session cookie.

What you should do is set a proper domain in the session cookie, so that this domain is based on the current server name.

We needed just this for our e-commerce platform, which supports multiple stores (and thus multiple domain names) on a single Rails app.

This is the solution we came up with (tested on Rails 2.3.4): it's a simple middleware.

{% codeblock %}
class SetCookieDomain
  def initialize(app)
    @app = app
  end

  def call(env)
    subdomain = remove_innermost_domain_from(env["SERVER_NAME"])
    env["rack.session.options"][:domain] = subdomain
    log "forced cookie domain to #{subdomain}"

    @app.call(env)
  end

  private

  def remove_innermost_domain_from(server_name)
    server_name.gsub(/^[^.]*/, '')
  end

  def log(message)
    RAILS_DEFAULT_LOGGER.info("========")
    RAILS_DEFAULT_LOGGER.info(message)
    RAILS_DEFAULT_LOGGER.info("========")
  end
end
{% endcodeblock %}

And this is a spec:

{% codeblock %}
describe SetCookieDomain do

  let(:app) { stub("app", :call => "") }
  let(:set_cookie_domain) { SetCookieDomain.new(app) }

  before(:each) do
    @env = {
      "SERVER_NAME" => "store.any.it", "SERVER_PORT" => "3000",
      "HTTP_HOST"   => "store.any.it:3000",
      "REQUEST_URI" => "/",
      "rack.session.options" => { :path => "/", :key => "_session_id", :expire_after => nil, :httponly => true, :domain => nil, :id => "any_id" },
    }
  end

  it "forces the session cookie domain to be the second-level domain of the full domain name" do
    @env["SERVER_NAME"] = "store.xpeppers.com"

    @env["rack.session.options"][:domain].should be_nil

    set_cookie_domain.call(@env)

    @env["rack.session.options"][:domain].should == ".xpeppers.com"
  end
end
{% endcodeblock %}

To enable the middleware, you've to put this line in your Rails startup files (e.g. environment.rb)

{% codeblock %}
config.middleware.use "SetCookieDomain"
{% endcodeblock %}

