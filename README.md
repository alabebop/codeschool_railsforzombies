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

4. count the number of entries

	```ruby
	Tweet.count
	```

5. order the found entries by a certain key

	```ruby
	Tweet.order(:zombie)
	```

6. return the first 10 entries

	```ruby
	Tweet.limit(10)
	```

7. find all entries with a certain key-value pair

	```ruby
	Tweet.where(zombie: "ash")
	```

8. chaining them together
	
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














