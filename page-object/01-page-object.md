!SLIDE
# Page Object

!SLIDE
# Problem
## Repeating html/css in tests != DRY

!SLIDE
# Solution
## Cucumber!

!SLIDE
# But even in Cucumber
    @@@ cucumber
    Then there (is|are) (n) Widget[s]
<span/>
    @@@ ruby
      assert_equal n, find('.widgets').size
<br/>
    @@@ cucumber
    When I delete a Widget[ named "X"]
<span/>
    @@@ ruby
      find('.widgets').detect do |w|
        w.find('.name').value == X
      end.find('.delete').click
<br/>
    @@@ cucumber
    Then there should exist a widget named "X"
<span/>
    @@@ ruby
      refute_nil find('.widgets').detect do |w|
        w.find('.name').value == X
      end

!SLIDE
# Still not DRY across steps

!SLIDE
# Page Object
## From selenium
### [http://code.google.com/p/selenium/wiki/PageObjects](http://code.google.com/p/selenium/wiki/PageObjects)
    @@@ java
    public class LoginPage { // oh god it's so verbose
      public HomePage loginAs(String username, String password) {
        // ... clever magic happens here
      }
    }

!SLIDE
# This is Ruby!
## We can totally one-up this

!SLIDE
# Initial Iteration
    @@@ ruby
    class Dom::Widget
      extend Enumerable
      def initialize(node)
        @node = node
      end
      def self.all
        find('.widgets').map{|node| new(node)}
      end
      def self.each
        all.each{|widget| yield widget }
      end
      def name
        @node.find('.name').value
      end
      def delete
        @node.find('.delete').click
      end
    end

!SLIDE
# Cucumber revisited
    @@@ cucumber
    Then there (is|are) (n) Widget[s]
<span/>
    @@@ ruby
      assert_equal n, Dom::Widget.count
<br/>
    @@@ cucumber
    When I delete a Widget[ named "X"]
<span/>
    @@@ ruby
      Dom::Widget.detect{|w| w.name == X }.delete
<br/>
    @@@ cucumber
    Then there should exist a widget named "X"
<span/>
    @@@ ruby
      refute_nil Dom::Widget.detect{|w| w.name == X }
## Where's my markup???

!SLIDE
# !Gratuitous Gem Sales Pitch!
### The follow slide contains gratuitous amounts of pandering and pleading on behalf of the gem author to use his/her gem. If you or anyone on your project have a history of bloated gem files or are bundling or are expecting to bundle in the near future, do not install this gem. If you have heart or back problems, please leave the room and re-enter when the gem sales pitch is complete. 
### Thank you and enjoy the ride.

!SLIDE
# Second Interation: Domino
## ActiveRecord inspired Page Object
    @@@ ruby
    class Dom::Widget < Domino
      selector '.widgets .widget'
      attribute :name # infers .name
      def delete
        @node.find('.delete').click
      end
    end
    Dom::Widget.find_by_name('alpha').delete

!SLIDE bullets
# Page Object advantages
* DRY up integration test code
* Convenience of Enumerable methods
* Decouple your view code (easy to modify markup)
