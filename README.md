# Rails for Zombies Play through
Notes from the code school workshop [rails for zombies](http://railsforzombies.org/)

## Level 1 DEEP IN THE CRUD

### Syntax

* Hash

	```ruby
	b = {
		id: 3,
		status: "I ate some brains",
		zombie: "Tim"
	}
	```

	* Accessing value with keys using dot syntax

		```ruby
		b.id
			=> 3
		```

		or the alternative hash (`[:key]`) syntax

		```ruby
		b[:id]
			=> 3
		```

* (useful) print string

	```ruby
	puts b[:id]
	```

### Dealing with the database:

The zombie database structure:

#### tweets

id | status | zombie
---|--------|-------
1  | Where can I get a good bite to eat? | Ash
2  | My left arm is missing, but I don't care | Bob
3  | I just ate some delicious brains | Jim
4  | OMG, my fingers turned green. #FML | Ash 

#### I. CREATE

Three ways to create a new zombie tweet entry:

1. verbose way

	```ruby 
	t = Tweet.new
	t.status = "I <3 brains."
	t.save
	```

2. less verbose way

	```ruby
	t = Tweet.new(
		status: "I <3 brains.",
		zombie: "Jim"
	)
	t.save
	```

3. compact way

	```ruby
	Tweet.create(
		status: "I <3 brains.",
		zombie: "Jim"
	)
	```

#### II. READ

1. Find

	1. find by id

		```ruby
		Tweet.find(2)
		```

	2. find multiple by id

		```ruby
		Tweet.find(2, 3, 4)
		```

	3. find the first, the last, find all

		```ruby
		Tweet.first
		```

		```ruby
		Tweet.last
		```

		```ruby
		Tweet.all
		```

2. count the number of entries

	```ruby
	Tweet.count
	```

3. order entries by a certain key value

	```ruby
	Tweet.order(:zombie)
	```

4. return the first 10 entries

	```ruby
	Tweet.limit(10)
	```

5. find all entries with a certain key-value pair

	```ruby
	Tweet.where(zombie: "ash")
	```

6. chaining them together
	
	* To find the first zombie tweet by the zombie "ash":

	```ruby
	Tweet.where(zombie: "ash").first
	```

	* To find Ash's first ten tweet ordered by status:

	```ruby
	Tweet.where(zombie: "ash").order(:status).limit(10)
	```

#### III. UPDATE

1. verbose way

	```ruby
	t = Tweet.find(3)
	t.zombie = "EyeballChomper"
	t.save
	```

2. less verbose way

	```ruby
	t = Tweet.find(2)
	t.attributes = {
		status: "Can I munch your eyeballs?",
		zombie: "EyeballChomper"
	}
	```

3. compact way

	```ruby
	t = Tweet.find(2)
	t.update(
		status: "Can I munch your eyeballs?",
		zombie: "EyeballChomper"
	)
	```

#### IV. DELETE

1. delete one

	```ruby
	t = Tweet.find(2)
	t.destroy
	```

2. delete all

	```ruby
	Tweet.destroy_all
	```

## Level 2 MODELS TASTE LIKE CHICKEN

### I. Model 
is the way how your Rails application communicates with a data store.

1. To define a Model class

	```ruby
	class Tweet < ActiveRecord::Base

	end
	```

	`<` is for `extends`

	This `Tweet` class then maps to the `tweets` table

	* tweets

		id | status | zombie
		---|--------|--------
		1  | Where can I get a good bite to | Ash
		2  | My left arm is missing, but I  | Bob
		3  | I just ate some delicious      | Jim
		4  | OMG, my fingers turned green.  | Ash


2. Validations

	We don't want empty records to be saved in the database, so we add validation code in our model class:

	```ruby
	class Tweet < ActiveRecord::Base
		validates_presence_of :status
	end
	```

	* Now trying to save an empty record will give you an error

		```
		>> t = Tweet.new
			=> #<Tweet id: nil, status: nil, zombie: nil>
		>> t.save
			=> false
		>> t.errors.messages
			=> {status: ["can't be blank"]}
		>> t.errors[:status][0]
			=> "can't be blank"
		```	

	* Other validation possibilities:
		* presence, numeric, unique:

			```ruby
			validates_presence_of :status
			validates_numericality_of :fingers
			validates_uniqueness_of :toothmarks
			```
		
		* special pattern: confirmation pairs
		
			```ruby		
			validates_confirmation_of :password		# to validate if two input fields match
													# (one of which is a confirmation field 
													# like in the case of a password or email) 
			```

			An example usage

			in `Model`:

			```ruby
			class Person < ActiveRecord::Base
				validates_confirmation_of :password, message: "should match"
				validates_presence_of :password_confirmation, if: :password_changed?
			end
			```

			in `View`:

			```ruby
			<%= password_field "person", "password" %>
			<%= password_field "person", "password_confirmation" %>
			```

		* requires a checkbox to be checked

			```ruby
			validates_acceptance_of :zombification	# to validate if a checkbox is checked
			```

		* validate length

			```ruby
			validates_length_of :password, minimum: 3
			```

		* validates format

			```ruby	
			validates_format_of :email, with: /regex/i  # provide a regular expression 
														# to validate email
			```

		* validates number range
			```ruby
			validates_inclusion_of :age, in: 21..99
			validates_exclusion_of :age, in: 0...21, 
				message: "Sorry, you must be over 21"
			```

	* Compact alternate syntax of validation

		```ruby
		validates :status, 
					presence: true, 
					length: { minimum: 3 }
		```

		Additional options

		```ruby
				presence: true
				uniqueness: true
				numericality: true
				length: { minimum: 0, maximum: 2000 }
				format: { with: /.*/ }
				acceptance: true
				confirmation: true
		```

### II. Relationships

1. Relational Database

	We have the tweets table, now we store zombies in their own table with unique ids and graveyard information. The two tables now have a _relationship_:

	> each zombie `has_many` tweets

	and

	> each tweet `belongs_to` a zombie

	Now the zombies show up in the tweets table with their unique id *zombie_id*, with which we can find the very zombie in the zombies table.

	* zombies

		id | name | graveyard
		---|------|-----------
		1 | Ash | Glen Haven Memorial Cemetery
		2 | Bob | Chapel Hill Cemetery
		3 | Jim | My Father's Basement

		```ruby
		class Zombie < ActiveRecord::Base
			has_many :tweets	# plural!
		end
		```

	* tweets

		id | status | zombie_id
		---|--------|-----------
		1 | Where can I get a good bite to eat? | 1
		2 | My left arm is missing, but I don't care | 2
		3 | I ust ate some delicious brains. | 3
		4 | OMG, my fingers turned green. #FML | 1

		```ruby
		class Tweet < ActiveRecord::Base
			belongs_to :zombie 		# singular!
		end
		```

2. Using relationships

	* Pass a zombie when creating a new tweet
 
		```ruby
		> ash = Zombie.find(1)
			=> #<Zombie id: 1, name: "Ash", graveyard: "Glen Haven Memorial Cemetery">
		> ash.tweets.count
			=> 2
		> t = Tweet.create(status: "Your eyelids taste like bacon.", zombie: ash)
			=> #<Tweet id: 5, status: "Your eyelids taste like bacon.", zombie_id: 1>
		> ash.tweets
			=> [
				#<Tweet id: 1, status "Where can I get a good bite to eat?", zombie_id: 1>,
				#<Tweet id: 4, status "OMG, my fingers turned green. #FML", zombie_id: 1>,
				#<Tweet id: 5, status "Your eyelids taste like bacon.", zombie_id: 1>
			]
		```
	* Get the zombie from an existing tweet

		```ruby
		> t = Tweet.find(5)
			=> #<Tweet id: 5, status: "Your eyelid tastes like bacon.", zombie_id: 1>
		> t.zombie
			=> #<Zombie id: 1, name: "Ash", graveyard: "Glen Haven Memorial Cemetery">
		> t.zombie.name
			=> "Ash"
		```

## Level 3 THE VIEWS AIN'T ALWAYS PRETTY

### I. View

View is the visual representation of an application. 

1. Rails view folder structure

	```
	zombie_twitter
		|_ app
			|_ views
				|_ layouts
				|	|_ application.html.erb 	# reused template
				|_ zombies
				|_ tweets
					|_ index.html.erb 	# list all tweets
					|_ show.html.erb 	# show a single tweet
	```	

2. ERB templating (.html.erb files)
	
	* `erb` stands for **E**mbedded **R**u**b**y

	* tag markers
		
		* `<% ... %>`: 
			Tags without the equals sign denote that the enclosed code is a *scriptlet*. Scriplets are caught and executed, the result can be used in following part of the template. Nothing will be rendered on the html page.

			```html
			<% tweet = Tweet.find(5) %>
			```
		
		* `<%= ... %>`:
			Tags with the equals sign indicates that enclosed is an *expression*, and that the result of the code as a string will be rendered on the page.

			```html
			<p>Posted by zombie <%= tweet.zombie %></p>
			```

### II. ERB Templates in a Rails Application

1. the HTML `application` (a reused html wrapper) in the `layout` folder

	`/app/views/layouts/application.html.erb`
	
	```erb
	<!DOCTYPE html>
	<html>
		<head><title>Twiiter for Zombies</title></head>
		<body>
			<header>...</header>
			<%= yield %>
		</body>
	</html>
	```
	
	The `yield` denotes the place where the content defined in each page template will be inserted.

2. a single page template, e.g. the `tweets/show` template that defines the page showing one single tweet

	`/app/views/tweets/show.html.erb`

	```erb
	<% tweet = Tweet.find(1) %>
	<h1><%= tweet.status %></h1>
	<p>Posted by <%= tweet.zombie.name %></p>
	```

	This part goes to the place denoted with `yield` in the **application.html.erb** layout template

3. template helpers 

	* Loop: `[array].each do |[variable name to be used in the loop]|`

		E.g., to list all tweets:

		```erb
		<ul>
		<% Tweet.all.each do |tweet| %>
			<li><%= tweet.status %>
		<% end %>
		</ul>
		```

	* Conditional: `if`, `elsif`, `else`

		```erb
		<% if Tweet.all.size == 0 %>
			<p>There's no tweets yet.</p>
		<% elsif Tweet.all.size == 1 %>
			<p>There is 1 tweet.</p>
		<% else %>
			<p>There are <%= Tweet.all.size %> tweets.</p>
		<% end %>
		```

	* Build link: `link_to`

		The *link_to* helper creates a link with the following signature:

		```erb
		<%= link_to link_text, link_destination [, options] %>
		```

		Example:

		```erb
		<p>Posted by <%= link_to tweet.zombie.name, zombie_path(tweet.zombie) %></p>
	
		```

		or with alternate syntax:

		```erb
		<p>Posted by <%= link_to tweet.zombie.name, tweet.zombie %></p>
		```

		renders to

		```html
		<p>Posted by <a href="/zombies/1">Ash</a></p>
		```

		* Options for the `link_to` helper:

			* `:class =>` class name

				e.g.:

				```erb
				<%= link_to "Edit", edit_user_path(user), :class => "edit-button" %>
				```

				renders to:

				```html
				<a href="/users/1/edit" class="edit-button">Edit</a>
				```

			* `method:` symbol of HTTP verb - assign a HTTP method for the link

				e.g. to delete the tweet with id 1, which we found earlier in a tag marker without equals:

				```erb
				<%= link_to "Delete", tweet, :method => :delete %>
				```

				then the following html will be rendered to page:

				```html
				<a rel="nofollow" data-method="delete" href="tweets/1">Delete</a>
				```

				* supported verbs 

					> :post, :delete, :patch, :put


			* `:data =>` hash - adds custom data attributes to html, some can trigger JS functions automatically

				e.g. a confirm popup with message "Are you sure?" for the delete link:

				```erb
				<%= link_to "Delete", tweet, :method => :delete, 
					:data => {confirm: "Are you sure?"} %>
				```

				will be rendered as

				```html
				<a href="/tweets/1" data-method="delete" 
					data-confirm="Are you sure?">Delete</a>
				```

				And clicking the delete link will first popup a JS confirm box with the message "Are you sure?"

			* `remote: true` - use AJAX submit instead of follow the link 

		* Pre-cooked links we can use:

			With the tweets table as our example:

			action | code | link | template
			-------|------|------|----------
			list all tweets | `tweets_path` | "/tweets" | /app/views/tweets/index.html.erb
			create new tweet | `new_tweet_path` | "/tweets/new" | /app/views/tweets/new.html.erb


			With `tweet = Tweet.find(1)`:

			action | code | link | template
			-------|------|------|----------
			show a tweet | `tweet` | "/tweets/1" | /app/views/tweets/show.html.erb
			edit a tweet | `edit_tweet_path(tweet)` | "/tweets/1/edit" | /app/views/tweets/edit.html.erb
			delete a tweet | `tweet, :method => :delete` | /tweets/1 | none



## Level 4 CONTROLLERS MUST BE EATEN












