!SLIDE
# Compound Model

!SLIDE
# It all started...
### Me: "having a model create other models in after\_create seems like a code smell. What pattern should I use?"
### Russ Olsen (Design Patterns in Ruby): "A model. Just because it's a model doesn't mean it has to be a database table."
### Me: !

!SLIDE
# Send someone a Shortmail
## A user sees it as a single entity

!SLIDE bullets
# Shortmail in a Relational DB
* n Addresses
* 1 Conversation
* 1 Post
* n Personalization (archived, etc)
* 0 "Shortmails"

!SLIDE
# a Shortmail = Compound Model
    @@@ ruby
    Shortmail.create(
      :to => 'Dave',
      :from => 'Nick',
      :subject => 'Hey Dave',
      :text => 'How are things?'
    )
    => true

!SLIDE
# Part 1: Transactional Creation
    @@@ ruby
    def save
      Conversation.transaction do
        normalize_addresses
        if existing_conversation
          create_post
        else
          create_conversation
          create_post
        end
        true # rollback returns false
      end
    end

!SLIDE
# Individual methods for sub-objects
    @@@ ruby
    def create_conversation
      @conversation = Conversation.new(
        :subject => @subject,
        :is_public => @public
      )
      raise ActiveRecord::Rollback unless @conversation.save
      @conversation.addresses = addresses
    end

!SLIDE
# Errors from multiple models
    @@@ ruby
    def errors
      # base errors
      @errors.merge(
        # conversation errors
        @conversation.errors
      ).merge(
        # post errors
        @post.errors
      )
    end

!SLIDE
# Example of a base error
    @@@ ruby
    begin
      # parse email addresses
    rescue Mail::Field::ParseError
      @errors.add :recipients, "are not all valid email addresses"
    end

!SLIDE
# Great places for high-level helpers
    @@@ ruby
    def self.convert_to_address(address)
      # handle address as a:
      # * String
      # * Address (AR Model)
      # * Mail::Address
      # And find or create a corresponding Address object
      # or raise a Shortmail::InvalidAddress exception
    end

!SLIDE bullets
# Advantages of a compound model
* Clean external DSL
* Individual models don't concern themselves with each other
* Individual models don't know about higher-order business logic
* Easy to unit test
