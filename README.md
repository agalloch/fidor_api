# FidorApi

[![Gem Version](https://badge.fury.io/rb/fidor_api.svg)](https://badge.fury.io/rb/fidor_api)
[![Build Status](https://travis-ci.org/fidor/fidor_api.svg?branch=v2)](https://travis-ci.org/fidor/fidor_api)

Ruby client library to work with Fidor APIs.

💡 This branch contains the `2.x.x` version. For previous versions see `v1` branch.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'fidor_api', '>= 2'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install fidor_api

## Usage

### 0. Configure

```ruby
client = FidorApi::Client.new do |config|
  # config.environment = FidorApi::Environment::FidorDE::Sandbox
  config.client_id     = 'your-client-id'
  config.client_secret = 'your-client-secret'

  # optional
  config.faraday = lambda do |faraday|
    faraday.use MyApp::CustomFaradayLogger, Rails.logger
  end
end
```

#### Environments

By default this gem ships with different supported environments:

`FidorApi::Environment::FidorDE::Sandbox` (default)<br>
Connects against the Fidor Germany Sandbox API (`*.sandbox.fidor.com`).<br>
Supports only oauth2 authorization code flow and the classic transfer APIs.

`FidorApi::Environment::FidorDE::Production`<br>
Connects against the Fidor Germany Production API (`*.fidor.de`).<br>
Supports only oauth2 authorization code flow and the classic transfer APIs.

`FidorApi::Environment::Future`<br>
Connects against a fictional API version which represents a future version of the Fidor Standard API (`*.example.com`).<br>
Supports more oauth2 based authorization flows and the generic transfer API.<br>
It is used for testing purposes only.

In some cases you'll need to connect to a custom environment (e.g. when using mocked APIs or if you're working with internal test-servers).<br>
For this you can simply implement your own environment definition (for example inside your application code):

```ruby
module FidorApi
  module Environment
    class Custom < Base
      def api_host
        'http://api.custom.example.com'
      end

      def auth_host
        api_host
      end

      def auth_redirect_host
        'http://auth.custom.example.com'
      end

      def auth_methods
        %i[authorization_code resource_owner_password_credentials client_credentials].freeze
      end

      def transfers_api
        :generic
      end
    end
  end
end

Rails.application.config.x.tap do |config|
  # [...]
  config.fidor_api.environment = FidorApi::Environment::Custom.new
  # [...]
end
```

Methods required to be implemented:

| Method Name          | Expected to Return  | Usage                                                             |
| -                    | -                   | -                                                                 |
| `api_host`           | `String`            | Used as endpoint for the normal API calls                         |
| `auth_host`          | `String`            | Used for the oAuth2 calls                                         |
| `auth_redirect_host` | `String`            | Used for the web-based redirect flow URLs                         |
| `auth_methods`       | `Array` of `Symbol` | Defines the supported oAuth2 authentication methods               |
| `transfers_api`      | `Symbol`            | Defines which transfer API is supported. `:generic` or `:classic` |

### 1. oAuth (Rails)

Redirect the user to the authorize URL:

```ruby
redirect_to client.authorize_start(
  redirect_uri: 'https://localhost:3000/callback'
)
```

Use code passed to the callback URL and fetch the access token:

```ruby
session[:api_token] = client.authorize_complete(
  redirect_uri: 'https://localhost:3000/callback',
  code:         params[:code]
)
```

### 2. Fetching data

```ruby
client.token = FidorApi::Token.new(session[:api_token])

user = client.user
# => FidorApi::Model::User

user = client.transactions
# => FidorApi::Collection

transaction = transactions.first
# => FidorApi::Model::Transaction
```

### 3. Creating transfers

```ruby
client.token = FidorApi::Token.new(session[:api_token])

transfer = client.create_internal_transfer(
  account_id:   875,
  receiver:     'kycfull@fidor.de',
  external_uid: '4279762F5',
  subject:      'Money for you',
  amount:       1000
)
# => FidorApi::Model::Transfer::Classic::Internal
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`.

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/fidor/fidor_api.

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
