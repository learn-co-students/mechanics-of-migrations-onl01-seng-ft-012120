


**Setting Up Your Migration**

Create a directory called db at the top level of the lesson's directory. 
Then, within the db directory, create a migrate directory. 
<!-- The mkdir command is the appropriate tool to use here. -->
In the db/migrate directory, create a file called 01_create_artists.rb (we'll talk about why we added the 01 later).
---------
With the file created, we'll need to add in the migration code:
<!-- # db/migrate/01_create_artists.rb -->
 
<class CreateArtists < ActiveRecord::Migration[5.2]
  def up
  end>
 
  <def down
  end
end>

**********************Active Record 5.x Migration Syntax Update
IMPORTANT: Active Record is primarily used in Rails applications and as of Active Record 5.x, 
we must specify which version of Rails the migration was written for, even in situations like this lab where we do not have Rails configured.
This lesson was originally created with gem versions that support Rails 5.2, so we need to make have our CreateArtist migration inherit from ActiveRecord::Migration[5.2].************************

Don't worry too much about this until you get to the Rails section. Until then, if you encounter an error like this...

<Caused by:
StandardError: Directly inheriting from ActiveRecord::Migration is not supported. Please specify the Rails release the migration was written for:
   class CreateArtists < ActiveRecord::Migration[5.2]
...simply add [5.2] or whatever number is displayed to the end of ActiveRecord::Migration in your migration file, exactly as the error message instructs.>

***Active Record Migration Methods: up, down, change***

Here we're creating a class called CreateArtists that inherits from ActiveRecord's ActiveRecord::Migration module. 
Within the class, we have an up method to define the code to execute when the migration is run and a down method to define the code to execute when the migration is rolled back. 
Think of it like "do" and "undo."

Another method is available to use besides up and down: change, which is more common for basic migrations.

<!-- # db/migrate/01_create_artists.rb -->
 
<class CreateArtists < ActiveRecord::Migration[5.2]
  def change
  end
end>

>>>>>From the Active Record Migrations RailsGuide:<<<<<

                    # The change method is the primary way of writing migrations. 
                    # It works for the majority of cases, where Active Record knows how to reverse the migration automatically

Let's take a look at how to finish off our CreateArtists migration, which will generate our artists table with the appropriate columns.

***Creating a Table***

Remember how we created a table using SQL with ActiveRecord?

<!-- NOTE: Recall, we can do this with IRB: irb -r active_record -->

First, we connect to a database, then write the necessary SQL to create the table. So, first, we'd have to connect to a database:

># ActiveRecord::Base.establish_connection(
  ># :adapter => "sqlite3",
  ># :database => "db/artists.sqlite"
># )

Then write some SQL to create the table:

<sql = <<-SQL
  CREATE TABLE IF NOT EXISTS artists (
  id INTEGER PRIMARY KEY,
  name TEXT,
  genre TEXT,
  age INTEGER,
  hometown TEXT
  )
SQL>
 
<ActiveRecord::Base.connection.execute(sql)>

Using migrations, we will still need establish Active Record's connection to the database, 
but we no longer need the SQL! Instead of dealing with SQL directly, we provide the migrations body (in Ruby) and Active Record takes care of creating complex SQL commands. This is less error-prone and much easier to read.

As a bonus, migrations, when paired with version control (git), create a record of changes to the database. 
If we just ran a command on the database, there's no undo, no opportunity for peer review and pull request behaviors. Migrations help programmers think about purpose not syntax.

Since we still need to connect to the database, let's make the connection inside config/environment.rb:
------------------------------------------------
<!-- # config/environment.rb -->
require 'rake'
require 'active_record'
require 'yaml/store'
require 'ostruct'
require 'date'
 
require 'bundler/setup'
Bundler.require
                                                  <
>ActiveRecord::Base.establish_connection(       <
  >:adapter => "sqlite3",                     < < < < < < < < < < < < < 
  >:database => "db/artists.sqlite"             <
>)                                                <
 
require_relative "../artist.rb"
-------------------------------------------------

****************************Reminder: The environment.rb file commonly contains things we want to read and make available to our Ruby application when it starts. 
It isn't necessary that you totally grasp all the parts of this file, but looking through it with this in mind, 
you might be able to gather what is happening: some gems, including active_record are required; something happens with Bundler; our database connection is established; the artist.rb file is read.**************************

With the connection to the database configured, we should have access to ActiveRecord::Migration, and can create tables using only Ruby!

<!-- # db/migrate/01_create_artists.rb -->
<def change
  create_table :artists do |t|
  end
end>

Here we've added the create_table method and passed the name of the table we want to create as a symbol. 
Pretty simple, right? Other methods we can use here are things like remove_table, rename_table, remove_column, add_column and others. 
See this list for more. https://edgeguides.rubyonrails.org/active_record_migrations.html#writing-a-migration

No point in having a table that has no columns in it, so let us add a few:

# db/migrate/01_create_artists.rb
 
<class CreateArtists < ActiveRecord::Migration[5.2]
  def change
    create_table :artists do |t|
      t.string :name
      t.string :genre
      t.integer :age
      t.string :hometown
    end
  end
end>

Looks a little familiar? On the left we've given the data type we'd like to cast the column as, and on the right, we've given the name we'd like to give the column. 
The only thing that we're missing is the primary key. Active Record will generate that column for us, and for each row added, a key will be auto-incremented.

And that's it! You've created your first Active Record migration. Next, we're going to see it in action!
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

***Running Migrations***

The simplest way is to run our migrations through a Rake task that we're given through the activerecord gem. How do we access these?

Run rake -T to see the list of commands we have.

****************************Note: If you get an error when trying to run rake commands, you may have a newer version of rake already installed compared to this lesson, causing a conflict. 
To avoid this error, run bundle exec rake -T. Adding bundle exec indicates that you want rake to run within the context of this lesson's bundle (defined in the Gemfile),
not the default version of rake you have installed globally on your computer.**************************

Let's look at the Rakefile. The commands listed when running rake -T are made available as Rake tasks through require 'sinatra/activerecord/rake'.

Now take a look again at environment.rb, which our Rakefile also requires:

<!-- # config/environment.rb -->
 
<require 'bundler/setup'
Bundler.require>
 
>ActiveRecord::Base.establish_connection(
  >:adapter => "sqlite3",
  >:database => "db/artists.sqlite"
>)

This file is requiring the gems in our Gemfile and giving our program access to them. 
We're using the establish_connection method from ActiveRecord::Base to connect to our artists database, which will be created in the migration via SQLite3 (the adapter).

After we've added the above code to config/environment.rb, it's time to run rake db:migrate.

Take a look at artist.rb. Let's create an Artist class.

<!-- # artist.rb -->
 
class Artist
end
Next, we'll extend the class with ActiveRecord::Base

<!-- # artist.rb -->
 
class Artist < ActiveRecord::Base
end
To test our newly-created class out, let's use the rake task rake console (or bundle exec rake console), which we've created in the Rakefile.

------Try Out The Following:--------

Check that the class exists:

Artist
<!-- # => Artist (call 'Artist.connection' to establish a connection) -->
"
---------------------------
View the columns in its corresponding table in the database:

Artist.column_names
<!-- # => ["id", "name", "genre", "age", "hometown"] -->
"
--------------------------

Instantiate a new Artist named Jon, set his age to 30, and save him to the database:

a = Artist.new(name: 'Jon')
<!-- # => #<Artist id: nil, name: "Jon", genre: nil, age: nil, hometown: nil> -->
 
a.age = 30
<!-- # => 30 -->
 
a.save
<!-- # => true -->
The .new method creates a new instance in memory, but for that instance to persist, we need to save it.
---------------------------

If we want to create a new instance and save it all in one go, we can use .create.

Artist.create(name: 'Kelly')
<!-- # => #<Artist id: 2, name: "Kelly", genre: nil, age: nil, hometown: nil> -->
"
---------------------------
Return an array of all Artists from the database:

Artist.all
<!-- # => [#<Artist id: 1, name: "Jon", genre: nil, age: 30, hometown: nil>, -->
 <!-- #<Artist id: 2, name: "Kelly", genre: nil, age: nil, hometown: nil>] -->
"
----------------------------
Find an Artist by name:

Artist.find_by(name: 'Jon')
# => #<Artist id: 1, name: "Jon", genre: nil, age: 30, hometown: nil>

There are several methods you can now use to create, retrieve, update, and delete data from your database, and a whole lot more.

Take a look at these CRUD methods, and play around with them. https://guides.rubyonrails.org/active_record_basics.html#crud-reading-and-writing-data