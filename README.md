### Logging from a Ruby on Rails application to a Graylog server

This guide briefly describes the steps required to send logs from your Ruby on Rails application to a Graylog server. **The Rails version used in this guide is v4.2.1.** but should work with different versions, too.

#### Overview

We'll be using [the official Graylog GELF gem](https://github.com/Graylog2/gelf-rb) to transport messages formatted by [the lograge gem](https://github.com/roidrage/lograge). The reason to use lograge is that Rails has a default logger that his very noisy and splits up messages over multiple lines. Lograge does a great job taming the format and provides a `key=value` formatter that we can use to extract fields in Graylog later.

#### Configuring your Rails application

Add the two gems to your `Gemfile`:

    gem 'gelf'
    gem 'lograge'
    
In your `config/environment/development.rb` (and `production.rb`) set up Rails to log to your Graylog server:

    config.logger = GELF::Logger.new("graylog.example.org", 12219, "WAN", { :facility => "YOUR_APP_NAME" })
    
Make sure to start a GELF UDP input on the configured port (12219 in this case) in your Graylog server.
    
Create a new initializer (`config/initializers/lograge.rb`) to configure the lograge formatting:

    Rails.application.configure do
      config.lograge.enabled = true
      config.lograge.formatter = Lograge::Formatters::KeyValue.new
    end
    
Your Rails application should now log into Graylog. In the next step we'll improve the parsing on the Graylog side.
    
#### Content pack
    

    
#### Custom events
