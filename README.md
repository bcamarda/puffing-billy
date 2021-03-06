# Puffing Billy

A rewriting web proxy for testing interactions between your browser and
external sites. Works with ruby + rspec.

Puffing Billy is like [webmock](https://github.com/bblimke/webmock), but for
your browser.

![](http://upload.wikimedia.org/wikipedia/commons/0/01/Puffing_Billy_1862.jpg)

## Overview

Billy spawns an EventMachine-based proxy server, which it uses to intercept
requests sent by your browser. It has a simple API for configuring which
requests need stubbing and what they should return.

Billy lets you test against known, repeatable data.  It also allows you to
test for failure cases.  Does your twitter (or facebook/google/etc)
integration degrade gracefully when the API starts returning 500s?  Well now
you can test it!

```ruby
it 'should stub google' do
  proxy.stub('http://www.google.com/').and_return(:text => "I'm not Google!")
  visit 'http://www.google.com/'
  page.should have_content("I'm not Google!")
end
```

## Installation

Add this line to your application's Gemfile:

    gem 'puffing-billy', :require => 'billy'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install puffing-billy

## Usage

In your `spec_helper.rb`:

```ruby
require 'billy/rspec'

# select a driver for your chosen browser environment
Capybara.javascript_driver = :selenium_billy
# Capybara.javascript_driver = :webkit_billy
# Capybara.javascript_driver = :poltergeist_billy
```

In your tests:

```ruby
# Stub and return text, json, jsonp (or anything else)
proxy.stub('http://example.com/text/').and_return(:text => 'Foobar')
proxy.stub('http://example.com/json/').and_return(:json => { :foo => 'bar' })
proxy.stub('http://example.com/jsonp/').and_return(:jsonp => { :foo => 'bar' })
proxy.stub('http://example.com/wtf/').and_return(:body => 'WTF!?', :content_type => 'text/wtf')

# Stub redirections and other return codes
proxy.stub('http://example.com/redirect/').and_return(:redirect_to => 'http://example.com/other')
proxy.stub('http://example.com/missing/').and_return(:code => 404, :body => 'Not found')

# Even stub HTTPS!
proxy.stub('https://example.com:443/secure/').and_return(:text => 'secrets!!1!')

# Pass a Proc (or Proc-style object) to create dynamic responses.
#
# The proc will be called with the following arguments:
#   params:  Query string parameters hash, CGI::escape-style
#   headers: Headers hash
#   body:    Request body string
#
proxy.stub('https://example.com/proc/').and_return(Proc.new { |params, headers, body|
  { :text => "Hello, #{params['name'][0]}"}
})
```

Stubs are reset between tests.  Any requests that are not stubbed will be
proxied to the remote server.

## Customising the javascript driver

If you use a customised Capybara driver, remember to set the proxy address
and tell it to ignore SSL certificate warnings. See
[lib/billy/rspec.rb](https://github.com/oesmith/puffing-billy/blob/master/lib/billy/rspec.rb)
to see how Billy's default drivers are configured.

## FAQ

1. Why name it after a train?

   Trains are *cool*.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## TODO

1. Integration for test frameworks other than rspec.
2. Caching (for super awesome improved test performance).
3. Show errors from the EventMachine reactor loop in the test output.

