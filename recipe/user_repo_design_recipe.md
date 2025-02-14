# USER Model and Repository Classes Design Recipe

_Copy this recipe template to design and implement Model and Repository classes for a database table._

## 1. Design and create the Table

If the table is already created in the database, you can skip this step.

Otherwise, [follow this recipe to design and create the SQL schema for your table](./single_table_design_recipe_template.md).

*In this template, we'll use an example table `users`*

```
# EXAMPLE

Table: users

Columns:
id | email | password | name | username
```

## 2. Create Test SQL seeds

Your tests will depend on data stored in PostgreSQL to run.

If seed data is provided (or you already created it), you can skip this step.

```sql
-- EXAMPLE
-- (file: spec/seeds.sql)

-- Write your SQL seed here. 

-- First, you'd need to truncate the table - this is so our table is emptied between each test run,
-- so we can start with a fresh state.
-- (RESTART IDENTITY resets the primary key)

TRUNCATE TABLE users RESTART IDENTITY; -- replace with your own table name.

-- Below this line there should only be `INSERT` statements.
-- Replace these statements with your own seed data.

INSERT INTO users (email,password,name, username) VALUES ('david@mail.com', '123','David','superdavid');
INSERT INTO users (email,password,name, username) VALUES ('anna@mail.com','123','Anna',' superanna');
```

Run this SQL file on the database to truncate (empty) the table, and insert the seed data. Be mindful of the fact any existing records in the table will be deleted.

```bash
psql -h 127.0.0.1 your_database_name < seeds_{table_name}.sql
```

## 3. Define the class names

Usually, the Model class name will be the capitalised table name (single instead of plural). The same name is then suffixed by `Repository` for the Repository class name.

```ruby
# EXAMPLE
# Table name: users

# Model class
# (in lib/user.rb)
class User
end

# Repository class
# (in lib/user_repository.rb)
class UserRepository
end
```

## 4. Implement the Model class

Define the attributes of your Model class. You can usually map the table columns to the attributes of the class, including primary and foreign keys.

```ruby
# EXAMPLE
# Table name: users

# Model class
# (in lib/user.rb)

class User

  # Replace the attributes by your own columns.
  attr_accessor :id, :email, :password,:name, :username
end

# The keyword attr_accessor is a special Ruby feature
# which allows us to set and get attributes on an object,
# here's an example:
#
# student = Student.new
# student.name = 'Jo'
# student.name
```

*You may choose to test-drive this class, but unless it contains any more logic than the example above, it is probably not needed.*

## 5. Define the Repository Class interface

Your Repository class will need to implement methods for each "read" or "write" operation you'd like to run against the database.

Using comments, define the method signatures (arguments and return value) and what they do - write up the SQL queries that will be used by each method.

```ruby
# EXAMPLE
# Table name: users

# Repository class
# (in lib/user_repository.rb)

class UserRepository

  # Selecting all records
  # No arguments
  def all
    Executes the SQL query:
    SELECT id, email, name, username FROM users;

    Returns an array of User objects.
  end

  # Gets a single record by its ID
  # One argument: the id (number)
  # def find(id)
    # Executes the SQL query:
    # SELECT id, name, cohort_name FROM students WHERE id = $1;

    # Returns a single Student object.
  # end

  # Add more methods below for each operation you'd like to implement.

  def create(user)
  Executes the SQL query:
  INSERT INTO users (email, password, name, username) VALUES ($1, $2, $3, $4) 
  Create an user
  Return nil
  end

  # def update(student)
  # end

  # def delete(student)
  # end
end
```

## 6. Write Test Examples

Write Ruby code that defines the expected behaviour of the Repository class, following your design from the table written in step 5.

These examples will later be encoded as RSpec tests.

```ruby
# EXAMPLES

# 1
# Get all users

repo = UserRepository.new

users = repo.all

users.length # =>  2

users[0].id # =>  1
users[0].name # =>  'David'
users[0].email # =>  'david@mail.com'

users[1].id # =>  2
users[1].name # =>  'Anna'
users[1].email # =>  'anna@mail.com'

# 2
# create a single user
repo = UserRepository.new

user = User.new
user.email = 'harry@mail.com'
user.password = '123'
user.name = 'Harry'
user.username = 'superharry'

repo.create(user)

users = repo.all

last_user = users.last
expect(last_user.name).to eq 'Harry'
expect(last_user.email).to eq 'harry@mail.com'

# 3
# Get an error if create an exist username 

repo = UserRepository.new

user = User.new
user.email = 'david02@mail.com'
user.password = '123'
user.name = 'David02'
user.username = 'superdavid'

expect{repo.create(user)}.to raise_error "The username is already exist"

# 4
# Get an error if create an exist email address 

repo = UserRepository.new

user = User.new
user.email = 'david@mail.com'
user.password = '123'
user.name = 'David02'
user.username = 'superdavid02'

expect{repo.create(user)}.to raise_error "The Email address is already exist"

```

Encode this example as a test.

## 7. Reload the SQL seeds before each test run

Running the SQL code present in the seed file will empty the table and re-insert the seed data.

This is so you get a fresh table contents every time you run the test suite.

```ruby
# EXAMPLE

# file: spec/user_repository_spec.rb

def reset_users_table
  seed_sql = File.read('spec/seeds.sql')
  if ENV["PG_password"] 
    connection = PG.connect({ host: '127.0.0.1', dbname: 'chitter_test', password: ENV["PG_password"] })
  else
    connection = PG.connect({ host: '127.0.0.1', dbname: 'chitter_test' })
  end
  connection.exec(seed_sql)
end

describe UserRepository do
  before(:each) do 
    reset_users_table
  end

  # (your tests will go here).
end
```

## 8. Test-drive and implement the Repository class behaviour

_After each test you write, follow the test-driving process of red, green, refactor to implement the behaviour._
