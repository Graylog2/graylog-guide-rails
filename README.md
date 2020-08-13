### Logging from a Ruby on Rails application to a Graylog server

This guide briefly describes the steps required to send logs from your Ruby on Rails application to a Graylog server. **The Rails version used in this guide is v4.2.1.** but should work with different versions, too.

#### Overview

We'll be using [the official Graylog GELF gem](https://github.com/Graylog2/gelf-rb) to transport messages formatted by [the lograge gem](https://github.com/roidrage/lograge). The reason to use lograge is that Rails has a default logger that is very noisy and splits up messages over multiple lines. Lograge does a great job taming the format and provides a `key=value` formatter that we can use to extract fields in Graylog later.

#### Configure Graylog

We provide a Graylog content pack that starts the correct GELF input type together with extractors (parser rules) to make the configuration really easy. [Just apply this content pack using your Graylog Web Interface](https://marketplace.graylog.org/addons/0a1caed3-92a5-4f86-840b-2c61421d73dc) and follow the next steps in this guide.

#### Configuring your Rails application

Add the two gems to your `Gemfile`:

    gem 'gelf'
    gem 'lograge'
    
In your `config/environment/development.rb` (and `production.rb`) set up Rails to log to your Graylog server:

    config.logger = GELF::Logger.new("graylog.example.org", 12219, "WAN", { :facility => "YOUR_APP_NAME" })

> You may need to manually include the silencer module to your GELF logger to avoid `NoMethodError (undefined method 'silence_logger' for #<GELF::Logger:0x0000000007f5f170>)`:
> ```ruby
>   GELF::Logger.send :include, ActiveRecord::SessionStore::Extension::LoggerSilencer
>   config.logger = GELF::Logger.new(...)
> ```
> 
> This silencer is being used to silence the logger and not leaking private information into the log, and it is required for security reason.


Create a new initializer (`config/initializers/lograge.rb`) to configure the lograge formatting:

    Rails.application.configure do
      config.lograge.enabled = true
      config.lograge.formatter = Lograge::Formatters::KeyValue.new
    end
    
Your Rails application should now log into the Graylog input the content pack launched and already apply proper parsing of the message content into fields.
    
#### Custom events

The above steps forward the standard built-in Rails logs to Graylog in realtime. Any explicit call to `logger` will also be sending to Graylog, but without the lograge formatting applied:

    logger.info("A new user with id <#{user.id}> signed up.")
