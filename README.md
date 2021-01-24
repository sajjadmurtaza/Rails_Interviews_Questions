# Rails Interview Questions



#### Interviewees

* Don't study to the test.  Not all of these questions will be used, and others will be asked that aren't covered here.  Know your stuff.
* Rails skills != CS skills
* Know whether you are interviewing for a full-stack vs. backend-focused position.
* Have code up on Github/Bitbucket/wherever - the more side projects you have to talk about, the better.
* Be comfortable with *some* version control system.
* Community awareness and resourcefulness can be as important as technical knowledge.

## The Questions

#### General

* What is a tree?  What is a DAG?
* Pseudo-code depth-first and breadth-first search.
* Given a sorted array, what is the fastest way to find a element?  What is the Big-O time to do so?
* Explain how ajax works, hitting all parts of the stack.

#### Ruby
---
* **What is the difference between a lambda, a block and a proc?**
  * BLOCKS
    * A block is a colleection of code enclosed in `do / end` statement or between bracket `{}`.
    * ``` 
      # single line blocks
      [1,2,3].each { _1 } # [1, 2, 3].each { |num| puts num }
      
      # multi line blocks
      [1, 2, 3].each do |num|
        puts num
      end
      ```
    * What is `yield`?
      * it's a Ruby keyword that calls a block when you use it.
      * WHen you use `yield` the code inside the block will run.
      * ```
        def fun_with_yield
          p 'Before Yield'
          yield
          p 'After yield'
        end
        
        fun_with_yield { p 'I am Yield' }
        
        # "Before Yield"
        # "I am Yield"
        # "After yield"
        ```
      * Yield can be user multiple time and with arguments as well
      * `block_given?` check if has been passed.
  * Procs
    * Procs are objects, blocks are not
    * ```
      p = Proc.new { puts "Hello World" }
      # #<Proc:0x00007f903bbd5998 (irb):64>
      p.call

      # Hello World

      # you can pass multiple procs to methods

      def multiple_procs(proc1, proc2)
        proc1.call
        proc2.call
      end

      a = Proc.new { puts "First proc" }
      b = Proc.new { puts "Second proc" }

      # First proc
      # Second proc
      ```
   * lambda
      * Proc and lambda both are Proc objects.
      * ```
        proc = Proc.new { puts "Hello world" }
        lam = lambda { puts "Hello World" }

        proc.class # returns 'Proc'
        lam.class  # returns 'Proc'
        ```
      * little difference when return lambda object `#<Proc:0x00007f903b8e6ac0 (irb):60 (lambda)>`
      * lamda check the number of arguments, while proc does not
        ```
        lam = lambda { |x| puts x }  

        lam.call(2)                    #  2
        lam.call                       # ArgumentError (wrong number of arguments (given 0, expected 1))
        lam.call(1,2,3)                # ArgumentError (wrong number of arguments (given 3, expected 1))

        # and in case of proc

        proc = Proc.new { |x| puts x } 
        proc.call(2)                   # 2
        proc.call                      # nil
        proc.call(1,2,3)               # 1
        ```
      * Lambdas and procs treat the ‘return’ keyword differently
        ```
        def proc_foo
             p = Proc.new { return 'I will get returned by the method :o' }
             puts "You will see me"
             p.call
             puts "You will not see me because the method returned :("
        end

        proc_foo
        # You will see me
        # => "I will get returned by the method :o" 

        # and that is not true when you use lambda

        def lambda_foo
             l = lambda { return puts 'I will not get returned by the method. I will get printed by the method!' }
             puts "You will see me"
             l.call
             puts "You will see me too because the method haven't returned :)"
        end

        lambda_foo
        # You will see me
        # I will not get returned by the method. I will get printed by the method!
        # You will see me too because the method haven't returned :)
        ```
   * lambda, proc and block related ref: [ref:1](http://awaxman11.github.io/blog/2013/08/05/what-is-the-difference-between-a-block/), [ref:2](http://stackoverflow.com/questions/626/when-to-use-lambda-when-to-use-proc-new), [ref:3](http://techspry.com/ruby_and_rails/proc-and-lambda-in-ruby/) 

---
* **How do you sort an Array of objects by a particular attribute?  What is a better way to do sorting with ActiveRecord?**
 ```
 # ruby sort by
 fruits = ["banana", "apple", "orange", "grapers"]
 
 fruits.sort! { |a, b|  a <=> b }
 # => ["apple", "banana", "grapers", "orange"]

 fruits.sort! { |a, b|  a.length <=> b.length }
 # => ["apple", "banana", "orange", "grapers"]

 User.all.sort_by { |u| u.name }
 
 User.all.sort_by &:name
 
 # in rails
 User.order(:name)
 ```
 [ref:1 <=>](https://www.youtube.com/watch?v=f4NItw7r33E), [ref: 2](https://medium.com/@albert.s.chun/ruby-the-spaceship-operator-101-717b42566971)
 
---
* **What are some of your favorite gems?  What are their alternatives?** [Ref: Ruby awesome gems](https://github.com/markets/awesome-ruby)

---
* **In Ruby, which is generally the better option: a recursive function or an iterative one?**
```
def iterative_factorial(n)
  (1..n).inject(:*)      # same (1..n).inject{ |sum, number| sum * number }
end

def recursive_factorial(n)
  return 1 if n <= 1

  n * recursive_factorial(n-1)
end
```

[ref: 1](https://www.rubyguides.com/2015/08/ruby-recursion-and-memoization/)

---
* **What are `#method_missing` and `#send`?  Why are they useful?**

method_missing is a method that ruby gives you access inside of your objects 
to handle situations when calling method that doesn't exist. 
It's sort of like a Begin/Rescue, but for method calls. 
It gives you one last chance to deal with that method call before an exception is raised.

```
class Cow
 def walk
  p "Let's walk"
 end
end

# c = Cow.new
# c.walk     # "Let's walk"
# c.fly      # NoMethodError (becasue cow can't fly :D)
```
Now instead of the NoMethodError exception, we make it clear to anyone that cows do not fly with the message.
```
class Cow
 def walk
  p "Let's walk"
 end
 
 def method_missing(m, *args, &block)
  return p "Sorry! Cow can't fly! She is very heavy :D" if m.to_s == 'fly'
  
  super
 end
end

# c = Cow.new
# c.walk     # "Let's walk"
# c.fly      # Sorry! Cow can't fly! She is very heavy :D
```
and send is a ruby method allowing to invoke another method by name.
```
 class Klass
     def hello(*args)
       "Hello"
     end
   end
   
   k = Klass.new
   k.send :hello     # => "Hello"
```
Note: `Klass.instance_methods(false)` will return all instance methods lists.

[ref: method_missing](https://www.leighhalliday.com/ruby-metaprogramming-method-missing), [ref: att_accessor...](https://stackoverflow.com/questions/20018784/attr-accessor-vs-attr-reader-instance-variables)

---
* **What is the difference between a string and a symbol?**
  * "string"
    * text between quotation marks "" i.e. "I am strring"
    * Strings are mutable.
    * if you repeat the assignment then object in memory location will change.
     ```
     irb(main):254:0> "foo".object_id
     # => 21540
     
     irb(main):255:0> "foo".object_id
     # => 21560
     
     irb(main):256:0> "foo".object_id
     => 21580

     ```
  * :symbol
    * symbol begins with ccolon : i.e. :symbol
    * symbols are immutable
      * immutable objects that can't be changed
      * object_id remains same (one object per symbol)
     ```
      # :foo will have same object
      
      irb(main):251:0> :foo.object_id
      # => 11652508
      
      irb(main):252:0> :foo.object_id
      # => 11652508
      
      irb(main):253:0> :foo.object_id
      # => 11652508

     ```
      
Note: String are about 1.7 times slower than symbols!
[ref](https://www.rubyguides.com/2018/02/ruby-symbols/)

---
* **What are the various Ruby runtimes, and how are they different?**[ref](https://www.rrtutors.com/site/answer/What-are-the-various-Ruby-runtimes#)

#### Rails

* What is the general history of Rails?
* What is new in Rails 4?
* Explain the different pieces of Rails.
* Walk through the flow of a request through Rails.
* What are the different server options for running a Rails/Rack app?
* Explain CSRF and how Rails combats it.
* Explain mass-assignment vulnerability.
* Define "DHH".
* Define "tenderlove".
* Why do some people say "Rails can't scale"?
* What is Rack?
* What is middleware? How does it compare to controller filters/actions?
* Explain various forms of caching available in Rails.
* What are some Ruby web server options?
* [# 1](http://adamjonas.com/blog/interview-prep/)
* [# 2](http://www.hub4tech.com/interview/ruby-on-rails)

## Other Questions

##### Why we need Asset Pipeline in rails.?  
[#Answer 1](http://guides.rubyonrails.org/asset_pipeline.html) -- [#Answer 2](http://coderberry.me/blog/2012/04/24/asset-pipeline-for-dummies/) -- [#Answer 3 *](http://stackoverflow.com/questions/14719788/how-can-i-determine-the-md5-digest-of-a-given-asset-in-the-rails-asset-pipeline)  

#####  Include and join ?  
[include(EAGER LOADING) -- join(LAZY LOADING)](#)  
[#Answer 1](http://tomdallimore.com/blog/includes-vs-joins-in-rails-when-and-where/) -- [#Answer 2](http://blog.bigbinary.com/2013/07/01/preload-vs-eager-load-vs-joins-vs-includes.html)  --  [#Answer 3](http://railscasts.com/episodes/181-include-vs-joins)   

#####  Inheritence Questions?  
[#Answer 1](http://stackoverflow.com/questions/9359948/multiple-inheritance-whats-a-good-example) -- [#Answer 2](http://beginnersbook.com/2013/05/java-inheritance-types/) -- [#Answer 3](http://www.dotnet-tricks.com/Tutorial/oops/JaIO211013-Understanding-Inheritance-and-Different-Types-of-Inheritance.html)

#####  Multiple table inheritence.?  
[#Answer 1](http://techspry.com/ruby_and_rails/multiple-table-inheritance-in-rails-3/)

#####  Multiple Inheritence in rails.  
[#Answer 1](http://www.nishantnigam.in/2012/01/simulating-multiple-inheritance-with.html) -- [#Answer 2](http://stackoverflow.com/questions/1282864/ruby-inheritance-vs-mixins) -- [#Answer 3 **](http://tutorials.jumpstartlab.com/topics/models/modules.html) -- [# In object-oriented programming languages, a mixin is a class that contains a combination of methods from other classes.](https://railskey.wordpress.com/2013/02/07/multiple-inheritance-ruby/)  

#####  If you have 3 objects(Extend)  
obj1  
obj2  
obj3  
you want "test" method only for obje3  
obj3.extend Abc  
obj3.test  
[# answer](http://saturnflyer.com/blog/jim/2011/09/28/4-simple-steps-extending-ruby-objects-the-tip-of-the-iceberg-with-dci/)  
[#Answer 1](https://www.digitalocean.com/community/tutorials/sqlite-vs-mysql-vs-postgresql-a-comparison-of-relational-database-management-systems)  --  [#Answer 2](http://www.railstips.org/blog/archives/2009/05/15/include-vs-extend-in-ruby/)  

#####  CarrierWave vs. Paperclip vs. Dragonfly comparison  
[#Answer 1](http://bcjordan.com/posts/rails-image-uploading-framework-comparison/) -- [# carrierwave is more flexible, complex image processing](http://praaveenvr.blogspot.fr/2014/03/compare-carrierwave-paperclip-and.html)  

##### Observer  
* Observer are not in rails 4  
[#Answer 1 *](http://codebrahma.com/ruby/2014/07/30/observers-in-rails.html) -- [#Answer 2](http://samuelmullen.com/2013/05/the-problem-with-rails-callbacks/) -- [#Answer 3](http://stackoverflow.com/questions/15165260/rails-observer-alternatives-for-4-0)  -- [#Answer 4](http://ablogaboutcode.com/2011/10/24/using-rails-observers-to-write-faster-tests-and-simpler-models/)  

##### rails before_action and before_filter
[#Answer 1 *](http://stackoverflow.com/questions/16519828/rails-4-before-filter-vs-before-action) -- [#Answer 2](http://stackoverflow.com/questions/20811029/difference-between-actions-and-filters-in-rails) -- [#Answer 3](https://twitter.com/rails/status/277121523649740800)  

##### class vs module
[#Answer 1](http://stackoverflow.com/questions/151505/difference-between-a-class-and-a-module) -- [#Answer 2](https://www.youtube.com/watch?v=taaIIYj1jFA&list=PL7A85FD7803A8CB1F&index=5)

##### Try / Dealing with Nil
[#Answer 1](http://apidock.com/rails/Object/try) -- [#Answer 2](https://www.youtube.com/watch?v=v8d5UthX9gw&index=6&list=PL7A85FD7803A8CB1F) -- [#Answer 3](http://pramodtech.quora.com/nil-empty-blank-present-in-Ruby-on-Rails)

##### Save Something after_save callback
[#Answer 1](http://stackoverflow.com/questions/5777172/rails-how-to-update-a-column-after-saving) -- [If you have to change any value, always try before_save](http://stackoverflow.com/questions/12835029/how-does-after-save-work-when-saving-an-object)

##### Initialization
[#Answer 1](http://www.zlu.me/rails/ruby/2013/05/29/initialization-is-an-interesting-thing.html)

##### Ruby on Rails Dup vs Clone
[#Answer 1](http://www.jvanbaarsen.com/blog/2014/07/01/ruby-on-rails-dup-vs-clone) -- [#Answer 2](https://coderwall.com/p/1zflyg/ruby-the-differences-between-dup-clone) -- [#Answer 3*](http://sachinchoudhary.blogspot.com/2014/01/ruby-clone-dup-vs-rails-deepdup.html)

#####  SQLite vs MySQL vs PostgreSQL: A Comparison Of Relational Database Management Systems  


------------ 

##### helper & helper_method
[#Answer 1](http://stackoverflow.com/questions/3992659/in-rails-what-exactly-do-helper-and-helper-method-do)

##### Background jobs
[#Answer 1](http://railscasts.com/episodes/271-resque?view=asciicast) -- [#Answer 2](http://railscasts.com/episodes/366-sidekiq) -- [#Answer 3](http://railscasts.com/episodes/171-delayed-job)

##### ElasticSearch
[#Answer 1**](http://www.sitepoint.com/full-text-search-rails-elasticsearch/) -- [#Answer 2](http://www.codinginthecrease.com/news_article/show/409843?referrer_id=948927) -- [#Answer 3*](http://aaronvb.com/articles/intro-to-elasticsearch-ruby-on-rails-part-1.html) -- [#Answer 4](http://stackoverflow.com/questions/25258121/elasticsearch-rails-setting-a-custom-analyzer) -- [#Answer 5](http://stackoverflow.com/questions/12409438/when-do-you-start-additional-elasticsearch-nodes/12414123#12414123) -- [#Answer 6](https://www.elastic.co/guide/en/elasticsearch/client/ruby-api/current/index.html)

##### Session & Cookies
[#Answer 1](http://www.theodinproject.com/ruby-on-rails/sessions-cookies-and-authentication) -- [#Answer 2](http://railscasts.com/episodes/270-authentication-in-rails-3-1) -- [#Answer 3](http://railscasts.com/episodes/250-authentication-from-scratch?autoplay=true) -- [#Answer 4](http://pothibo.com/2013/09/sessions-and-cookies-in-ruby-on-rails/) -- [#Answer 5](http://www.justinweiss.com/blog/2015/03/17/how-rails-sessions-work/)

##### Gemfile and Gemfile.lock
[#Answer 1](http://stackoverflow.com/questions/6927442/what-is-the-difference-between-gemfile-and-gemfile-lock-in-ruby-on-rails) -- [#Answer 2](https://www.youtube.com/watch?v=ZE-8rqPI3B4&list=PL7A85FD7803A8CB1F&index=9) -- [#Answer 3](http://stackoverflow.com/questions/6588674/what-does-bundle-exec-rake-mean)

##### Delegate
[#Answer 1](http://pivotallabs.com/rails-delegates-are-even-more-useful-than-i-knew/) -- [#Answer 2](http://wyeworks.com/blog/2009/6/4/rails-delegate-method) -- [#Answer 3](http://apidock.com/rails/Module/delegate)

##### Object Relational Mapping(ORM)
[#Answer 1](http://guides.rubyonrails.org/active_record_basics.html) -- [#Answer 2](http://stackoverflow.com/questions/2194915/what-is-orm-as-related-to-ruby-on-rails)

##### Layout + rails render and redirect
[#Answer 1](http://railscasts.com/episodes/8-layouts-and-content-for) -- [#Answer 2](http://brettu.com/rails-daily-ruby-tip-28-whats-the-difference-between-redirect_to-and-render-in-rails/)


##### & symbol in map
[#Answer 1](http://stackoverflow.com/questions/9468564/what-does-post-all-mapid-mean) -- [#Answer 2](http://stackoverflow.com/questions/12084507/what-does-the-map-method-do-in-ruby) -- [#Answer 3](http://apidock.com/ruby/Enumerable/map)

##### Active Record: Preventing SQL Injection Attacks
[#Answer 1](http://guides.rubyonrails.org/security.html#sql-injection) -- [#Answer 2](https://blog.8thlight.com/adam-gooch/2013/01/04/protect-yourself-from-sql-injection.html) -- [#Answer 3](https://blog.8thlight.com/adam-gooch/2013/01/04/protect-yourself-from-sql-injection.html) -- [#Answer 4](http://rails-sqli.org/) -- [#Answer 5](http://stackoverflow.com/questions/1582511/rails-active-record-nuances-and-protecting-against-injection-attacks)

------------ 
##### Encapsulation
[#Answer 1](http://samurails.com/interview/ruby-inheritance-encapsulation-polymorphism/)

##### send
[#Answer 1](http://stackoverflow.com/questions/7895253/rails-100-newb-issue-send-method)

#### serialize
[#Answer 1](http://ruby-journal.com/how-to-write-custom-serializer-for-activerecord-number-serialize/) -- [#Answer 2](http://stackoverflow.com/questions/2959661/rails-serializing-objects-in-a-database)

##### What is http
[#Answer 1](https://www.youtube.com/watch?v=kGOpY2J31pI&list=PL7A85FD7803A8CB1F&index=39) - [#Answer 2](http://stackoverflow.com/questions/2441962/what-is-restful-routing)

##### What is rake
[#Answer 1](http://stackoverflow.com/questions/18737696/what-is-rake-and-how-it-is-used-in-rails) - [#Answer 2](http://www.tutorialspoint.com/ruby-on-rails/rails-and-rake.htm) 

#### What is RACK
[#Rack is basically specification of these two things => what the server should send to the app and what the app should return to the server](http://blog.gauravchande.com/what-is-rack-in-ruby-rails)  


[#Answer 1](https://www.amberbit.com/blog/2011/07/13/introduction-to-rack-middleware/) - [_2_](http://southdesign.de/blog/rack.html) - [_3_](https://www.amberbit.com/blog/2011/07/13/introduction-to-rack-middleware/) - [_4*_](https://blog.engineyard.com/2015/understanding-rack-apps-and-middleware) - [_5_](http://stackoverflow.com/questions/2256569/what-is-rack-middleware) - [_6_](http://rack.github.io/) - [_7_railscast](http://railscasts.com/episodes/151-rack-middleware?view=asciicast)
------------ 

##### What is noSQL

[#Answer 1](http://www.zdnet.com/article/what-is-nosql-and-why-do-you-need-it/) - [#Answer 2*](http://www.slideshare.net/Leesy/an-introduction-to-nosql-mongodb) - [#Answer 3](http://www.slideshare.net/harrikauhanen/nosql-3376398) - [#Answer 4](https://www.youtube.com/watch?v=KSq6tMMXZ8s)


[Rails Hot Topic](http://techspry.com/ruby_and_rails) -- (http://www.skilledup.com/articles/ruby-on-rails-interview-questions-answers) -- [__@_](http://blog.reskill.me/ruby-on-rails-interview-questions-that-will-land-you-the-job/?utm_source=content&utm_medium=fb&utm_campaign=ruby_questions)
[_*_Questions_](https://gist.github.com/ryansobol/5252653) == [_2_](http://www.careerride.com/ruby-on-rails-interview-questions.aspx)

------------  

##### Search File
* [#Answer 1](http://www.howtogeek.com/112674/how-to-find-files-and-folders-in-linux-using-the-command-line/)  

##### ERB 2 SLIM
* [#Answer 1](http://stackoverflow.com/questions/10347572/convert-erb-template-to-slim)

https://www.javatpoint.com/ruby-interview-questions
https://www.educba.com/ruby-interview-questions/
https://medium.com/better-programming/53-ruby-on-rails-interview-questions-and-answers-eb99eed1aeb7
