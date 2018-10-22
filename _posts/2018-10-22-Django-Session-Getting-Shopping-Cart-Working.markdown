---
layout: post
title:  "Django Session, Getting a Shpping Cart Working"
date:   2018-10-22 14:58:58 -0500
categories: code
excerpt_separator: <!--more-->
---
For the past few days, I've been working through a shopping cart tutorial, and figuring out how to create a shopping cart in an app using Session.  I often thought about session before I used it here, and the different ways a session can work.  A session stays in place, before a user logs in and can stay in place after a user logs out.  The duration of the session in my case will be based on time not whether the user is logged in or not.  This allows a guest user to add items to their shopping cart and not lose those items just because they were not logged in when they added them.  Once the customer does log in, their previous items are available for them to purchase in their cart.  

Here's the beginning of the session cart:  



{% highlight ruby %}
#Session Sample


from django.shortcuts import render

from .models import Cart

def cart_create(user=None):
	cart_obj = Cart.objects.create(user=None)
	print('New Cart created')
	return cart_obj

def cart_home(request):
	request.session['cart_id'] = "12"
	cart_id = request.session.get("cart_id", None)
	qs = Cart.objects.filter(id=cart_id)
	if qs.count() == 1:
		print('Cart ID exists')
		cart_obj = qs.first()
	else:
		cart_obj = cart_create()
		request.session['cart_id'] = cart_obj.id
	return render(request, "carts/home.html", {})


{% endhighlight %}


