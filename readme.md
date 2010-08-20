# Exception2db #

I love hoptoad. I especially like the ability to catch exceptions in rake tasks and have those exceptions logged at hoptoad using hoptoad_notifier.

Currently I am working on a government project where hoptoad can not be used. Rather than writing code that captures all the information about exception, I am letting hoptoad_notifier do all the heavy lifting.

Instead of sending the information to hoptoad this plugin ensures that information is written to a database.

This plugin has been tested with Rails 2.3.5 .

## I am using Rails 3 ##

If you are using Rails 3 then follow the instructions mentioned below. If you are using Rails 2.x  then follow the instructions [mentioned here]() .

## Add necessary gems  to Gem file ##

    gem 'exception2db'
    gem 'hoptoad_notifier'


## Setup configuration ## 

Create a new file called <tt>~/config/intializers/exception2db.rb</tt> and following text to that file.

    HoptoadNotifier.configure do |config|
    end

    module HoptoadNotifier
      def self.send_notice(data)
        E2db.create(:exception => data.to_xml)
      end
    end

## Create a migration file ##

Generate migration <pre>rails g migration add_exception2db_table</pre> 

Open the migration file and add following text

    class AddException2dbTable < ActiveRecord::Migration
      def self.up
        create_table :exception2dbs do |t|
          t.text :exception, :null => false

          t.datetime :created_at, :null => false
        end
      end

      def self.down
        drop_table :exception2dbs
      end
    end

Run migration <pre>rake db:migrate</pre> 


h2. How to use it

Whenever there is an exception that exception will be logged to database. Remember all the hoptoad rules apply. It means no excpetion will be logged in development environment.

To view exceptions visit "http://localhost:3000/exception2db":http://localhost:3000/exception2db .

In order to test this feature first add following line in <tt>application_controller.rb</tt> .

    def local_request?
     false
    end

Then start the server in *non development* environment. Once again please start your server in production
or some other environment. Exceptions in development mode are ignored by hoptoad. 

Go to a url that raises exception.

Then visit "http://localhost:3000/exception2db":http://localhost:3000/exception2db . If you are not seeing *not authorized* message then read the next section.

If you are not seeing the logging of exception then please read "this article":http://neeraj.name/2010/04/23/I-am-not-seeing-hoptoad-messages.html .

h2. Configuring security

By default no security check is performed while visiting "http://localhost:3000/exception2db":http://localhost:3000/exception2db in development mode. In other environment a security check is done. At the bottom of <tt>config/initializers/exception2db.rb</tt> add something like this to configure security settings.

    Exception2dbConfig.set = {
      :is_allowed_to_view => lambda {|controller| controller.send('admin_logged_in?') }
    }

Above code assumes that you have <tt>admin_logged_in?</tt> method defined in <tt>application_controller.rb</tt> . The <tt>is_allowed_to_view</tt> key accepts a proc and controller is provided to you. So you can call any controller method.

h2. How to run tests for this plugin

h3. Running rspec tests

Stand in the current directory and run following command. I assume that you have rspec already installed.

    spec spec

h3. Running cucumber test

Stand in the current directory and run following command. I assume you have cucumber already installed.

    cucumber


Copyright (c) 2010 neerajdotname, released under the MIT license