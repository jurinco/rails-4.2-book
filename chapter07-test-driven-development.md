Tests
=====

Introduction
------------

I have been programming for 30 years and most of the time I have managed
quite well without test-driven development (TDD). I am not going to be
mad at IT dinosaurs if they decide to just skip this chapter. You can
create Rails applications without tests and are not likely to get any
bad karma as a result (at least, I hope not - but you can never be
entirely sure with the whole karma thing).

But if you should decide to go for TDD, then I can promise you that it
is an enlightenment. The basic idea of TDD is that you write a test for
each programming function to check this function. In the pure TDD
teaching, this test is written before the actual programming. Yes, you
will have a lot more to do initially. But later, you can run all the
tests and see that the application works exactly as you wanted it to.
The read advantage only becomes apparent after a few weeks or months,
when you look at the project again and write an extension or new
variation. Then you can safely change the code and check it still works
properly by running the tests. This avoids a situation where you find
yourself saying "oops, that went a bit wrong, I just didn't think of
this particular problem".

Often, the advantage of TDD already becomes evident when writing a
program. Tests can reveal many careless mistakes that you would
otherwise only have stumbled across much later on.

This chapter is a brief overview of the topic test-driven development
with Rails. If you have tasted blood and want to find out more, you can
dive into the official Rails documentation at
<http://guides.rubyonrails.org/testing.html>.

> **Note**
>
> TDD is just like driving a car. The only way to learn it is by doing
> it.

Example for a User in a Web Shop
--------------------------------

Let's start with a user scaffold in an imaginary web shop:

    $
      [...]
    $
    $
          [...]
          invoke    test_unit
          create      test/models/user_test.rb
          create      test/fixtures/users.yml
          [...]
          invoke    test_unit
          create      test/controllers/users_controller_test.rb
          invoke    helper
          [...]
          invoke      test_unit
          create        test/helpers/users_helper_test.rb
          [...]
    $
          [...]
    $

You already know all about scaffolds (if not, please go and read ?
first) so you know what the application we have just created does. The
scaffold created a few tests (they are easy to recognise because the
word `test` is in the file name).

The complete test suite of a Rails project is processed with the command
`rake test`. Let's have a go and see what a test produces at this stage
of development:

    $
    Run options: --seed 23117

    # Running tests:

    .......

    Finished tests in 0.696922s, 10.0442 tests/s, 18.6535 assertions/s.

    7 tests, 13 assertions, 0 failures, 0 errors, 0 skips
    $

The output "`7 tests, 13 assertions, 0 failures, 0 errors, 0
    skips`" looks good. By default, a test will run correctly in a
standard scaffold.

Let's now edit the `app/models/user.rb` and insert a few validations (if
these are not entirely clear to you, please read ?):

    class User < ActiveRecord::Base
      validates :login_name,
                presence: true,
                length: { minimum: 10 }

      validates :last_name,
                presence: true
    end

Then we execute `rake test` again:

    $
    Run options: --seed 51265

    # Running tests:

    F.....F

    Finished tests in 0.178619s, 39.1896 tests/s, 55.9851 assertions/s.

      1) Failure:
    UsersControllerTest#test_should_create_user [/Users/stefan/webshop/test/controllers/users_controller_test.rb:20]:
    "User.count" didn't change by 1.
    Expected: 3
      Actual: 2

      2) Failure:
    UsersControllerTest#test_should_update_user [/Users/stefan/webshop/test/controllers/users_controller_test.rb:39]:
    Expected response to be a <redirect>, but was <200>

    7 tests, 10 assertions, 2 failures, 0 errors, 0 skips
    $

Boom! This time we have "`2 failures`". The error happens in the
"`should create user`" and the "`should update user`". The explanation
for this is in our validation. The example data created by the scaffold
generator went through in the first `rake test` (without validation).
The errors only occurred the second time (with validation).

This example data is created as *fixtures*tests fixturestixtures tests
in YAML format in the directory `test/fixtures/`. Let's have a look at
the example data for User in the file `test/fixtures/users.yml`:

    one:
      login_name: MyString
      first_name: MyString
      last_name: MyString
      birthday: 2013-07-17

    two:
      login_name: MyString
      first_name: MyString
      last_name: MyString
      birthday: 2013-07-17

There are two example records there that do not fulfil the requirements
of our validation. The login\_name should have a length of at least 10.
Let's change the `login_name` in `test/fixtures/users.yml` accordingly:

    one:
      login_name: MyString12
      first_name: MyString
      last_name: MyString
      birthday: 2013-07-17

    two:
      login_name: MyString12
      first_name: MyString
      last_name: MyString
      birthday: 2013-07-17

Now, a `rake test` completes without any errors again:

    $
    Run options: --seed 2058

    # Running tests:

    .......

    Finished tests in 0.150927s, 46.3800 tests/s, 86.1344 assertions/s.

    7 tests, 13 assertions, 0 failures, 0 errors, 0 skips
    $

We now know that valid data has to be contained in the
`test/fixtures/users.yml` so that the standard test created via scaffold
will succeed. But nothing more. We now change the
`test/fixtures/users.yml` to a minimum (for example, we do not need a
`first_name`):

    one:
      login_name: MyString12
      last_name: Obama

    two:
      login_name: MyString12
      last_name: Bush

To be on the safe side, let's do another `rake
    test` after making our changes (you really can't do that often
enough):

    $
    Run options: --seed 1554

    # Running tests:

    .......

    Finished tests in 0.141682s, 49.4064 tests/s, 91.7548 assertions/s.

    7 tests, 13 assertions, 0 failures, 0 errors, 0 skips
    $

> **Important**
>
> All fixtures are loaded into the database when a test is started. You
> need to keep this in mind for your test, especially if you use
> uniqueness in your validation.

### Functional Tests

tests
functional
functional tests
tests
Let's take a closer look at the point where the original errors
occurred:

      1) Failure:
    UsersControllerTest#test_should_create_user [/Users/stefan/webshop/test/controllers/users_controller_test.rb:20]:
    "User.count" didn't change by 1.
    Expected: 3
      Actual: 2

      2) Failure:
    UsersControllerTest#test_should_update_user [/Users/stefan/webshop/test/controllers/users_controller_test.rb:39]:
    Expected response to be a <redirect>, but was <200>

In the `UsersControllerTest` the User could not be created nor changed.
The controller tests are located in the directory `test/functional/`.
Let's now take a good look at the file
`test/controllers/users_controller_test.rb`

    require 'test_helper'

    class UsersControllerTest < ActionController::TestCase
      setup do
        @user = users(:one)
      end

      test "should get index" do
        get :index
        assert_response :success
        assert_not_nil assigns(:users)
      end

      test "should get new" do
        get :new
        assert_response :success
      end

      test "should create user" do
        assert_difference('User.count') do
          post :create, user: { birthday: @user.birthday, first_name: @user.first_name, last_name: @user.last_name, login_name: @user.login_name }
        end

        assert_redirected_to user_path(assigns(:user))
      end

      test "should show user" do
        get :show, id: @user
        assert_response :success
      end

      test "should get edit" do
        get :edit, id: @user
        assert_response :success
      end

      test "should update user" do
        patch :update, id: @user, user: { birthday: @user.birthday, first_name: @user.first_name, last_name: @user.last_name, login_name: @user.login_name }
        assert_redirected_to user_path(assigns(:user))
      end

      test "should destroy user" do
        assert_difference('User.count', -1) do
          delete :destroy, id: @user
        end

        assert_redirected_to users_path
      end
    end

At the beginning, we find a `setup` instruction:

    setup do
      @user = users(:one)
    end

These three lines of code mean that for the start of each individual
test, an instance `@user` with the data of the item `one` from the file
`test/fixtures/users.yml` is created. setup is a predefined callback
that - if present - is started by Rails before each test. The opposite
of setup is teardown. A teardown - if present - is called automatically
after each test. tests functional setuptests functional teardown

> **Note**
>
> For every test (in other words, at each run of `rake
>         test`), a fresh and therefore empty test database is created
> automatically. This is a different database than the one that you
> access by default via `rails console` (that is the development
> database). The databases are defined in the configuration file
> `config/database.yml`. If you want to do debugging, you can access the
> test database with `rails
>         console test`.

This functional test then tests various web page functions. First,
accessing the index page:

    test "should get index" do
      get :index
      assert_response :success
      assert_not_nil assigns(:users)
    end

The command `get :index` accesses the page [/users](/users).
`assert_response :success` means that the page was delivered. The line
`assert_not_nil assigns(:users)` ensures that the controller does not
pass the instance variable `@users` to the view with the value `nil`
(setup ensures that there is already an entry in the database).

> **Note**
>
> The symbol `:users` is used here to make sure that `@users` in the
> controller class to be tested is used, not `@users` in the test class
> itself.

Let's look more closely at the two problems from earlier. First,
`should create user`:

    test "should create user" do
      assert_difference('User.count') do
        post :create, user: { birthday: @user.birthday, first_name: @user.first_name, last_name: @user.last_name, login_name: @user.login_name }
      end

      assert_redirected_to user_path(assigns(:user))
    end

The block `assert_difference('User.count') do ... end` expects a change
by the code contained within it. `User.count` after should result in +1.

The last line `assert_redirected_to
      user_path(assigns(:user))` checks if after the newly created
record the redirection to the corresponding view `show` occurs.

The second error occurred with `should update
      user`:

    test "should update user" do
      patch :update, id: @user, user: { birthday: @user.birthday, first_name: @user.first_name, last_name: @user.last_name, login_name: @user.login_name }
      assert_redirected_to user_path(assigns(:user))
    end

Here, the record with the `id` of the `@user` record was supposed to be
updated with the attributes of the `@user` record. Then, the `show` view
for this record was again supposed to be displayed. Logically, this test
could not work either, because a) the `@user` record did not exist in
the database and b) it could not be updated as it was not valid.

Without commenting each individual functional test line by line, it is
becoming clear what these tests do: they execute real queries to the Web
interface (or actually to the controllers) and so they can be used for
testing the controllers.

> **Tip**
>
> With `rake test:functionals` you can also run just the functional
> tests in the directory `test/functional/`.
>
>     $
>     Run options: --seed 59879
>
>     # Running tests:
>
>     .......
>
>     Finished tests in 0.152887s, 45.7854 tests/s, 85.0301 assertions/s.
>
>     7 tests, 13 assertions, 0 failures, 0 errors, 0 skips
>     $

### Unit Tests

tests
unit
unit tests
tests
assert
tests
For testing the validations that we have entered in
`app/models/user.rb`, units tests are more suitable. Unlike the
functional tests, these test only the model, not the controller's work.

> **Tip**
>
> With `rake test`, all tests present in the Rails project are executed.
> With `rake test:units`, only the unit tests in the directory
> `test/models/` are executed.

The unit tests are located in the directory `test/models/`. But a look
into the file `test/models/user_test.rb` is rather sobering:

    require 'test_helper'

    class UserTest < ActiveSupport::TestCase
      # test "the truth" do
      #   assert true
      # end
    end

By default, scaffold only writes a commented-out dummy test. That is why
`rake test:units` runs through without any content:

    $
    Run options: --seed 30150

    # Running tests:



    Finished tests in 0.002333s, 0.0000 tests/s, 0.0000 assertions/s.

    0 tests, 0 assertions, 0 failures, 0 errors, 0 skips
    $

A unit test always consists of the following structure:

    test "an assertion" do
      assert something_is_true_or_false
    end

The word `assert`assert already indicates that we are dealing with an
assertion in this context. If this assertion is `true`, the test will
complete and all is well. If this assertion is `false`, the test fails
and we have an error in the program (you can specify the output of the
error as string at the end of the assert line).

> **Note**
>
> If you have a look around at
> <http://guides.rubyonrails.org/testing.html> then you will see that
> there are some other `assert` variations. Here are a few examples:
>
> -   `assert( ,
>                    )`
>
> -   `assert_equal( ,
>                   ,
>                    )`
>
> -   `assert_not_equal( ,
>                   ,
>                    )`
>
> -   `assert_same( ,
>                   ,
>                    )`
>
> -   `assert_not_same( ,
>                   ,
>                    )`
>
> -   `assert_nil( ,
>                    )`
>
> -   `assert_not_nil( ,
>                    )`
>
> -   `assert_match( ,
>                   ,
>                    )`
>
> -   `assert_no_match( ,
>                   ,
>                    )`
>
Let's breathe some life into the first test in the
`test/unit/user_test.rb`:

    require 'test_helper'

    class UserTest < ActiveSupport::TestCase
      test 'a user with no attributes is not valid' do
        user = User.new
        assert !user.save, 'Saved a user with no attributes.'
      end
    end

This test checks if a newly created User that does not contain any data
is valid (it should not). As `assert` only reacts to `true`, I placed
a"`!`" before `User.new.valid?` to turn the `false` into a `true`, as an
empty user cannot be valid.

So a `rake test:units` then completes immediately:

    $
    Run options: --seed 43622

    # Running tests:

    .

    Finished tests in 0.051971s, 19.2415 tests/s, 19.2415 assertions/s.

    1 tests, 1 assertions, 0 failures, 0 errors, 0 skips
    $

Now we integrate two asserts in a test to check if the two fixture
entries in the `test/fixtures/users.yml` are really valid:

    require 'test_helper'

    class UserTest < ActiveSupport::TestCase
      test 'an empty user is not valid' do
        assert !User.new.valid?, 'Saved an empty user.'
      end

      test "the two fixture users are valid" do
        assert User.new(last_name: users(:one).last_name, login_name: users(:one).login_name ).valid?, 'First fixture is not valid.'
        assert User.new(last_name: users(:two).last_name, login_name: users(:two).login_name ).valid?, 'Second fixture is not valid.'
      end
    end

Then once more a `rake test:units`:

    $
    Run options: --seed 10228

    # Running tests:

    ..

    Finished tests in 0.054056s, 36.9987 tests/s, 55.4980 assertions/s.

    2 tests, 3 assertions, 0 failures, 0 errors, 0 skips
    $

Fixtures
--------

tests
fixtures
fixtures
tests
With *fixtures* you can generate example data for tests. The default
format for this is YAML. The files for this can be found in the
directory `test/fixtures/` and are automatically created with
`rails generate scaffold`. But of course you can also define your own
files. All fixtures are loaded anew into the test database by default
with every test.

> **Note**
>
> Examples for alternative formats (e.g. CSV) can be found at
> <http://api.rubyonrails.org/classes/ActiveRecord/Fixtures.html>.

### Static Fixtures

fixtures
static
The simplest variant for fixtures is static data. The fixture for `User`
used in ? statically looks as follows:

    one:
      login_name: barak.obama
      last_name: Obama

    two:
      login_name: george.w.bush
      last_name: Bush

You simple write the data in YAML format into the corresponding file.

### Fixtures with ERB

fixtures
with ERB
Static YAML fixtures are sometimes too unintelligent. In these cases,
you can work with ERB (see ?).

If we want to dynamically enter today's day 20 years ago for the
birthdays, then we can simply do it with ERB in
`test/fixtures/users.yml`

    one:
      login_name: barak.obama
      last_name: Obama
      birthday: <%= 20.years.ago.to_s(:db) %>

    two:
      login_name: george.w.bush
      last_name: Bush
      birthday: <%= 20.years.ago.to_s(:db) %>

Integration Tests
-----------------

tests
integration
integration tests
tests
Integration tests are tests that work like functional tests but can go
over several controllers and additionally analyze the content of a
generated view. So you can use them to recreate complex workflows within
the Rails application. As an example, we will write an integration test
that tries to create a new user via the Web GUI, but omits the
`login_name` and consequently gets corresponding flash error messages.

A `rake generate scaffold` generates unit and functional tests, but not
integration tests. You can either do this manually in the directory
`test/integration/` or more comfortably with
`rails generate integration_test`. So let's create an integration test:

    $
          invoke  test_unit
          create    test/integration/invalid_new_user_workflow_test.rb
    $

We now populate this file
`test/integration/invalid_new_user_workflow_test.rb` with the following
test:

    require 'test_helper'

    class InvalidNewUserWorkflowTest < ActionDispatch::IntegrationTest
      fixtures :all

      test 'try to create a new empty user and check for flash error messages' do
        get '/users/new'
        assert_response :success

        post_via_redirect "/users", user: {:last_name => users(:one).last_name}
        assert_equal '/users', path
        assert_select 'li', "Login name can&#39;t be blank"
        assert_select 'li', "Login name is too short (minimum is 10 characters)"
      end
    end

The magic of the integration test lies amongst others in the method
post\_via\_redirect, with which you can carry on after a post in the
test. This method is only available within an integration test.

All integration tests can be executed with `rake
    test:integration`. Let's have a go:

    $
    Run options: --seed 61457

    # Running tests:

    .

    Finished tests in 0.146213s, 6.8393 tests/s, 27.3573 assertions/s.

    1 tests, 4 assertions, 0 failures, 0 errors, 0 skips
    $

The example clearly shows that you can program much without manually
using a web browser to try it out. Once you have written a test for the
corresponding workflow, you can rely in future on the fact that it will
run through and you don't have to try it out manually in the browser as
well.

rake stats
----------

rake stats
With `rake stats` you get an overview of your Rails project. For our
example, it looks like this:

    $
    +----------------------+-------+-------+---------+---------+-----+-------+
    | Name                 | Lines |   LOC | Classes | Methods | M/C | LOC/M |
    +----------------------+-------+-------+---------+---------+-----+-------+
    | Controllers          |    79 |    53 |       2 |       9 |   4 |     3 |
    | Helpers              |     4 |     4 |       0 |       0 |   0 |     0 |
    | Models               |     8 |     7 |       1 |       0 |   0 |     0 |
    | Mailers              |     0 |     0 |       0 |       0 |   0 |     0 |
    | Javascripts          |    19 |     0 |       0 |       0 |   0 |     0 |
    | Libraries            |     0 |     0 |       0 |       0 |   0 |     0 |
    | Controller tests     |    49 |    39 |       1 |       0 |   0 |     0 |
    | Helper tests         |     4 |     3 |       1 |       0 |   0 |     0 |
    | Model tests          |    13 |    11 |       1 |       0 |   0 |     0 |
    | Mailer tests         |     0 |     0 |       0 |       0 |   0 |     0 |
    | Integration tests    |    15 |    12 |       1 |       0 |   0 |     0 |
    +----------------------+-------+-------+---------+---------+-----+-------+
    | Total                |   191 |   129 |       7 |       9 |   1 |    12 |
    +----------------------+-------+-------+---------+---------+-----+-------+
      Code LOC: 64     Test LOC: 65     Code to Test Ratio: 1:1.0

    $

In this project, we have a total of 64 LOC (Lines Of Code) in the
controllers, helpers and models. Plus we have a total of 65 LOC for
tests. This gives us a test relation of 1:1.0, which should be the
principal objective. Logically, this does not say anything about the
quality of tests.

More on Testing
---------------

The most important link on the topic testing is surely the URL
<http://guides.rubyonrails.org/testing.html>. There you will also find
several good examples on this topic. Otherwise, Railscasts
(<http://railscasts.com/episodes?utf8=%E2%9C%93&search=test>) offers a
few good screencasts on this topic.

No other topic is the subject of much discussion in the Rails community
as the topic testing. There are very many alternative test tools. One
very popular one is RSpec (see <http://rspec.info/>). I am deliberately
not going to discuss these alternatives here, because this book is
mainly about helping you understand Rails, not the thousands of extra
tools with which you can build your personal Rails development
environment.