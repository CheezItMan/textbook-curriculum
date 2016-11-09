# Advanced Model Tests & as_json
## Learning Goals
- Identify places we can use helper methods to DRY up our Model Testing
- Override the default as_json method to hide by default attributes from json serialization.  

## Model Testing

In model tests you generally want to tests to accomplish these things:

1.  Verify that all validations are in place.
1.  Verify that custom methods work properly
1.  Verify that relationships, especially `belongs_to` are working correctly.


### Getting Started Testing

Assume we have a model with the following schema:

```ruby
class CreateContacts < ActiveRecord::Migration
  def change
    create_table :contacts do |t|
      t.string :name
      t.string :tel
      t.string :email
      t.string :avatar

      t.timestamps null: false
    end
  end
end
```

We want all Contacts to have a name, telephone and email and the e-mail and telephone number must be unique.  So when you get started you will often get started testing like this:

```ruby
require 'test_helper'

class ContactTest < ActiveSupport::TestCase
  def setup
    @contact = Contact.new(name: "Tyrion Lannister", tel: "333-333-3333", email: "Tyrion@donttellmysister.com", avatar: "http://placecage.com/300/300")
  end

  test "The name attribute is required." do
    @contact.name = nil
    refute @contact.valid?
    refute @contact.save
    assert @contact.errors.messages.include? :name
    assert @contact.errors.messages[:name].include? "can't be blank"
  end


  test "The email attribute is required." do
    @contact.email = nil
    refute @contact.valid?
    refute @contact.save
    assert @contact.errors.messages.include? :email
    assert @contact.errors.messages[:email].include? "can't be blank"
  end


  
	... lots and lots of validation tests

end
```

So we end up with a lot of tests for validations, lots and lots of copy and paste.  There must be a better way!  Life is too short.

There is.  The basic validations we're using all have a few things in common:  
*  The list of validation errors are stored in model.errors.messages
*  They all cause `.valid?` and `.save` to fail.  

When we run into problems like this, the first place to turn is a loop.

```ruby
  test "Required Attributes are genuinely required" do
    contact = Contact.new

    required_fields =  %w(name email tel)
    refute contact.valid?
    refute contact.save
    required_fields.each do |field|
      assert contact.errors.messages.include? field.to_sym
      assert contact.errors.messages[field.to_sym].include? "can't be blank"
    end
  end
```

That's **MUCH** DRYer, but it has a couple of disadvantages.    

1.  This test fails for any validation error and it's not specific about *which* validation failed.
2.  It only works for required fields, nothing about fields that have to be unique or other validations.  

We can do better.  

First we can generalize this test to handle all kinds of validations as a helper method.

```ruby
  test "Required Attributes are genuinely required" do
    assert_validations Contact.new, %w(name email tel), "can't be blank"
  end

  test "Unique Attributes must be unique" do
    assert_validations Contact.new(name: @contact.name, email: @contact.email, tel: @contact.tel), %w(email tel), "has already been taken"
  end

  private

  def assert_validations (model, fields, message)
    # We need to save or run .valid? to check errors
    refute model.valid?
    refute model.save
    
    fields.each do |field|
      assert model.errors.messages.include?(field.to_sym), "Failure the field #{field} #{message}"
      assert model.errors.messages[field.to_sym].include?(message), "Failure the field #{field} #{message}"
    end
  end

```

This does a pretty good job, we've managed to reduce our validation testing code with our handy `assert_validations` helper method, and the assert's error message will indicate which field failed the validations test.  


## Overriding as_json

Your Model classes have inherited a method `as_json(options = nil)` from `ActiveRecord::Base`.  We've seen this to select the fields rendered as Json in the controller.

```ruby
render json: @post.as_json(
      only: [:id, :content, :created_at],
      include: { user: { only: [:id, :name, :tel, :avatar, :email] } }
    ), status: :ok
```

However that's a PAIN to do every time you want to render a model, especially if  the fields remain the same.  Instead you can ovrrride your model's `as_json` method to set up a default method to render your model as Json.

```ruby
  def as_json(options = nil)
    super({ only: [:id, :name, :tel, :avatar, :email]}.merge(options || {}))
  end
```

## Resources
- [Better Serialization, less as_json](https://robots.thoughtbot.com/better-serialization-less-as-json)
- [Minitest Github](https://github.com/seattlerb/minitest)