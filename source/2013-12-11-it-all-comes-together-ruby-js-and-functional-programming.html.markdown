---
title: "It all comes together: Ruby, JS, and functional programming"
date: 2013-12-11 15:10 UTC
tags: ruby, javascript, clojure, functional programming
---

Whenever you learn a new word, you inevitably see that word in the newspaper
a day later.  You've been seeing that word all along you just didn't know it was
there.  I had a similar experience with programming yesterday when Ruby,
Javascript, and [functional
programming](http://en.wikipedia.org/wiki/Functional_programming) concepts (by way of Clojure and Ruby
lambdas) all came together.  It was an exciting day...

###Javascript and Ajax
I took my first deep dive into javascript a couple weeks ago doing exercises on
[exercism.io](http://exercism.io/) and [Javascript
Patterns](http://www.amazon.com/JavaScript-Patterns-Stoyan-Stefanov/dp/0596806752).
I am proud to say the curly braces no longer scare me.  Javascript logically led
me to Ajax in Rails.  We spent some time writing simple Ajax calls using JQuery
similar to those below:

```
$.ajax({
  type: 'POST',
  url: '/articles',
  success: function(data){
    alert("ok");
    $("#title").html(data);
  }
});
```

The concept of the success callback function was new but seemed reasonable
enough.

###Functional Programming and Clojure
In parallel over the past few weeks I started working my way through
[Functional Programming for the Object-Oriented Programmer](https://leanpub.com/fp-oo),
which is the [Ruby Rogues](http://rubyrogues.com/) next book club book.  This
book is based in Clojure and its concepts were new but digestable because of my
work with Javascript and my dabbling in Ruby's tokenization in [Ruby Under
a Microscope](http://www.amazon.com/Ruby-Under-Microscope-Illustrated-Internals/dp/1593275277).

```
(apply + [1 4 9 16])
```

While a 'fun' mental exercise for me, I considered reading about Clojure to be
a type of procrastination from doing my Rails project work.  I was wrong.  The
fp concepts were almost immediately helpful.

###Refactoring Rails Controllers using lambdas
Back to rails.  I wrote [a post](http://www.simontaranto.com/2013/11/15/a-refactoring-epiphany.html)
recently about a refactoring epiphany I had when breaking down a large
controller action.  In our [current
project](http://tutorials.jumpstartlab.com/projects/fourth_meal.html) we were
again faced with refactoring a bloated controller action.  The controller action
looked like this to start:

```
  def create
    @transaction = current_order.build_transaction(transaction_params)
    @transaction.update(:order_id => current_order.id)
    if @transaction.save
      @transaction.pay!
      if current_user
        current_order.update(:user_id => current_user.id, :status => "paid")
      else
        current_order.update(:status => "paid")
      end
      session[:current_order] = nil
      OrderMailer.order_confirmation(@transaction).deliver
      flash[:notice] = "Successfully created your order!"
      redirect_to transaction_path(@transaction)
    else
      flash[:notice] = "There was a problem creating your order!"
      render :new
    end
  end
```

We took the same approach I wrote about in my other post and broke down the
action into smaller, more manageable methods and we ended up with the below:

```
  def create
    @transaction = current_order.build_transaction(updated_params)
    if @transaction.save
      transaction_successfully_saved(@transaction)
      flash[:notice] = "Successfully created your order!"
      redirect_to transaction_path(@transaction)
    else
      flash[:notice] = "There was a problem creating your order!"
      render :new
    end
  end

  private

    # some supporting methods aka garbage in the closet
```

This refactor was certainly an improvement as it made the create action more readable and easier to update.
I then spoke with my mentor about how we could further improve this code and
lambdas ensued.  The create action became this:

```
  def create
    CompletePurchase.new(current_user, current_order).create(transaction_params,
      ->(transaction) {
        session[:current_order] = nil
        flash[:notice] = "Successfully created your order!"
        redirect_to transaction_path(transaction)
      },
      -> {
        flash[:notice] = "There was a problem creating your order!"
        render :new
    })
  end
```

along with a supporting class that housed all the actual work.

```
class CompletePurchase

  def initialize(user, order, mailer=OrderMailer)
    @user = user
    @order = order
    @mailer = mailer
  end

  def create(params, success, failure)
    ActiveRecord::Base.transaction do
      transaction = @order.build_transaction(params)
      if transaction.save
        transaction_successfully_saved(transaction)
        success.call(transaction)
      else
        failure.call
      end
    end
  end

  def transaction_successfully_saved(trans)
    trans.pay!
    @order.update(:user_id => @user.id) if @user
    @mailer.order_confirmation(trans).deliver
  end

end
```

While the lambda syntax with its stabby arrows looked odd at first, combining
my recent learnings on javascript callback functions and Clojure's function passing,
it was a small leap to see how this pattern works and why it's really nice.

Using this pattern the 'logic' of the action is pushed down to this CompletePurchase
class and all the controller does is

1) pass off context (current\_user, current\_order, transaction\_params) and

2) respond (manipulate session state, flash notices, and redirects) depending on the success or failure of the action.

Generally, this pattern is good to reach for when you have a controller action
that is talking to more than one model.

Letting all these new concepts settle in can take some time but it's really
motivating when seemily disparate concepts all coalesce and result in understanding.

Keep reading!
