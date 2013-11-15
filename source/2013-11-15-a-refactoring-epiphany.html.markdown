---
title: A refactoring epiphany
date: 2013-11-15 15:27 UTC
tags: refactoring, ruby
---

There are tons of resources out there that list ideals for ruby code and
methods to learn about refactoring your
code([an example](http://ghendry.net/refactor.html)) if you didn't follow those
ideals.  All of these ideas sound great but implementing them while you're in
the middle of dealing with bugs or tricky features can be tough.  This past week
I commited to refactoring a messy before\_action I wrote for
a controller.  Our project was [Dinner
Dash](http://tutorials.jumpstartlab.com/projects/dinner_dash.html).

####A few lessons learned from this experience:

- A solid test harness allows you to aggressively refactor.  A weak or
  non-existent test harness restricts your ability and desire to change existing
  code.

- Many shorter methods in place of fewer longer methods allow you to better
  understand and therefore more easily update and extend functionality.

- Refactoring is fun.

####An example

We have a controller that sits on top of a join table (order\_items) and the
order\_items#create action has some complex before\_actions due to it having to
perform different logic depending on the status of the current\_user. Prior to
refactoring, the load\_order before\_action looked like this:

#####The Original

```
class OrderItemsController < ApplicationController

...

private

    def load_order
      unless current_user
        @user = User.new_guest
        if @user.save
          session[:user_id] = @user.id
        end
      end
      @order = Order.find_or_initialize_by_id(session[:order_id], status: "unsubmitted")
      @order.user_id = current_user.id
      if @order.new_record?
        @order.save!
        session[:order_id] = @order.id
      end
    end
end
```
This action is doing way too much.  It is:

- checking for a current\_user and if there isn't one, making a new\_guest and
  setting that new user's session.
- `finding\_or\_initializing a new order
- setting the order's user to the current\_user
- saving the new order and updating the session in some way

Way too many things going on at once.  As the requirements changed (ie - we
found bugs in our code) we needed to update this logic so that it would maintain
a guest user's cart through sign up and logging in again (by using the session).
The first few glances at this method didn't show exactly what needed to happen so
relying heavily on our feature tests I set about refactoring the large
load\_order method into smaller methods.  We ended up with the below code.

#####An Update
```
    def load_order
      find_or_create_cart
    end

    def find_or_create_cart
      create_and_log_in_guest_user unless current_user
      @order = find_or_create_order
      save_order_and_set_session if @order.new_record?
    end

    def create_and_log_in_guest_user
      session[:user_id] = User.new_guest_user_id
    end

    def save_order_and_set_session
      @order.save!
      session[:order_id] = @order.id
    end

    def find_or_create_order
      order = Order.find_or_initialize_by_id(session[:order_id], status: "unsubmitted")
      order.user_id = current_user.id
      order
    end
```
While certainly far from perfect or complete, this next iteration of the code:

- maintains the same functionality as before
- has the added benefit of pushing some logic down into the model layer (see the new\_guest\_user\_id method)
- makes debugging easier
- allows for easier implementation of additional features

Once the long method was broken out into these smaller methods it was significanly less intimidating to make updates.  Instead of having to think about updating some lines in a large method you can now think about adding a new small method or updaing just one small method.

Refactoring can seem like a waste of time when your user stories are piling deeper and deeper but so far I've found that investing the time to refactor confusing sections of code pays dividends quickly.

For some more fun check out Katrina Owen's
[talk](http://confreaks.com/videos/1071-cascadiaruby2012-therapeutic-refactoring).
