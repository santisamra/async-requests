# AsyncRequest
[![Gem Version](https://badge.fury.io/rb/async-requests.svg)](https://badge.fury.io/rb/async-requests)
[![Build Status](https://travis-ci.org/Wolox/async-requests.svg?branch=master)](https://travis-ci.org/Wolox/async-requests)
[![Code Climate](https://codeclimate.com/github/mdesanti/async-requests/badges/gpa.svg)](https://codeclimate.com/github/mdesanti/async-requests)
[![Test Coverage](https://codeclimate.com/github/mdesanti/async-requests/badges/coverage.svg)](https://codeclimate.com/github/mdesanti/async-requests/coverage)

### Summary
At [Wolox](http://www.wolox.com.ar) we build Rails Apps. Some of them need heavy computing when a request is received. In order to make the App scalable, we perform those heavy actions in background. We return a job-id and the client asks for it's state a few seconds after.

`async_request` gives us the possibility of handling these type of requests in a simple way.

### Installation

Add `gem 'async_request'` to your gemfile

Run `rails g async_request`

### Usage

Simply call `async_request(your_worker, *args)` like this:

``` ruby
class SomeController < ApplicationController
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  def my_compute_heavy_endpoint_option_1
    _job, token = AsyncRequest::Job.execute_async(MyComputeHeavyWorker, { some: 'args' }, 'another arg')
    render json: { token: token, url: async_request.job_url }, status: 202
  end

  def my_compute_heavy_endpoint_option_2
    _job, token = AsyncRequest::Job.execute_async(MyComputeHeavyWorker, { some: 'args' }, 'another arg')
    render json: { token: token }, status: 202, location: async_request.job_url
  end
end

class MyComputeHeavyWorker
  def execute(*_params)
    # Perform heavy task
    [200, { message: 'success' }]
  end
end
```

This will enqueue a Sidekiq task that will call `MyComputeHeavyWorker.new.execute` with the args passed as parameters to `execute_async`.

The client can then make a GET request to the returned URL sending the `token` in a `X-JOB-AUTHORIZATION` header (configurable). If the job's completed, it will get a response code and a json body. Otherwise, a `202` code will be returned.

`MyComputeHeavyWorker` must return an array with two components. The first one must be the status code, the second one must be the response that will be sent to the client. Take into account that the response must respond to `to_json`.

### Configuration

There are some aspects of the gem you can configure:

``` ruby
# config/initializers/async_request.rb
AsyncRequest.configure do |config|
  config.sign_algorithm = 'HS256' # This is the default, valid algorithms: HS256 and RS256
  config.request_header_key = 'X-JOB-AUTHORIZATION' # This is the default
  config.encode_key = Rails.application.secrets.secret_key_base
  config.decode_key = Rails.application.secrets.secret_key_base
  config.token_expiration = Time.zone.now + 1.day
end

```

## About ##

This project is maintained by [Matías De Santi](https://github.com/mdesanti) and it was written by [Wolox](http://www.wolox.com.ar).

![Wolox](https://raw.githubusercontent.com/Wolox/press-kit/master/logos/logo_banner.png)

## License

**AsyncRequest** is available under the MIT [license](https://raw.githubusercontent.com/mdesanti/async-requests/master/LICENSE).
