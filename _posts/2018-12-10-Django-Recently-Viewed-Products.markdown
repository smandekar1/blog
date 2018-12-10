---
layout: post
title:  "Adding User Data About Viewed Products"
date:   2018-10-22 14:58:58 -0500
categories: code
excerpt_separator: <!--more-->
---
I wanted to add a feature to my recent ecommerce app and retain information about products that the customer has viewed.  After saving the objects that the customer has used to the database, I want display those products on a products page so that they might be inclined to visit those pages again and purchase those products possibly.  There were two routes that I worked with in implementing this.  One method was to save the products viewed to a sepereate list in the views.py file.  Another method that I used was to save the objects to the postgres database itself and have the objects retrieved from the database.  Product objects would be bound to the user using a foreign key.  Retrieving the objects from the database should allow for more flexibility as the the list would be cleared out every time the server is restarted.  If this app were to scaled up, I did not want to have a lot of product object data for each user, so I have created some code that deletes the product objects when the number of them gets to large.  

Goals going forward:

1. Put the logic of creating a product object after the user has viewed the product into the models.py file taking it out of the views.py file.  This goes along with the fat model methodology. 

2. Put the logic of limiting the number of product objects kept into models.py file.  

This is how things currently look in the views.py and models.py file:      



{% highlight ruby %}
#Portion of views.py


class ProductDetailSlugView(DetailView):
    queryset = Product.objects.all()
    template_name = "products/detail.html"

    def get_context_data(self, *args, **kwargs):
        context = super(ProductDetailSlugView, self).get_context_data(*args, **kwargs)
        cart_obj, new_obj = Cart.objects.new_or_get(self. request)
        context['cart'] = cart_obj

        return context

    def get_object(self, *args, **kwargs):
        request = self.request
        slug = self.kwargs.get('slug')
        # item_id = Product.objects.get(pk=pk, active=True)
        # instance = get_object(Product, slug=slug, active=True)
        try:
            instance = Product.objects.get(slug=slug, active=True)
        except Product.DoesNotExist:
            raise Http404("Not Foud..")
        except Product.MultipleObjectsReturned:
            qs = Product.objects.filter(slug=slug, active=True)
            instance = qs.first()
        except:
            raise Http404("Uhhmmm")
        # print(instance)
        if len(recently_viewed) > 3:
            del recently_viewed[-1]
        # global recently_viewed
        if instance not in recently_viewed:
            recently_viewed.insert(0,instance)

        # print('recently_viewed')
        # print(recently_viewed)

        # get or create session 

        viewed_product_obj, new_obj = Viewed_Product.objects.new_or_get(self.request)


        product_id = self.kwargs.get('slug')
        print('product_id is: ', product_id)

        Viewed_Product_Manager.foo()
       
        # product_id = request.POST.get('product_id')
        if product_id is not None:
            try:
                product_obj = Product.objects.get(slug=slug, active=True)
            except Product.DoesNotExist:
                print("Show message to user, product is gone?")
                return redirect("viewed_product:home")
            # if product_obj not in viewed_product_obj.products.all():
            # print('prod_obj was not in viewed_prod_objects...')
            

            Viewed_Product_Object.objects.create(user=viewed_product_obj, products=product_obj)
            product_objects = Viewed_Product_Object.objects.filter(user=viewed_product_obj).values()
            # product_objects = product_objects.product_objects_set.all()
            # for product in product_objects:
            #     products = products
            #     print(product_objects.products)
            print(product_objects)

            deleting_objects1 = Viewed_Product_Object.objects.filter(user=viewed_product_obj)[0:3]
            # values_list('pk', flat=True)[:2])
            
            # CreditPerson.objects.filter(pk__in=CreditPerson.objects.filter(name=person.name).values_list('pk', flat=True)[1:]).delete()
            print('prod objects sliced: ', deleting_objects1)

            if len(product_objects) > 5:

                Viewed_Product_Object.objects.filter(pk__in=Viewed_Product_Object.objects.filter(user=viewed_product_obj).values_list('pk',flat=True)[0:3]).delete()





            deleting = Viewed_Product_Object.objects.filter(user=viewed_product_obj, id=20).values()
            print('deleting: ', deleting)
            deleting_object = Viewed_Product_Object.objects.filter(user=viewed_product_obj, id=20)
            deleting_object.delete()
            # viewed_objects = viewed_product_obj.products.last()
            # print('this viewed objects: ', viewed_objects)

            # global prod_two
            # if prod_two is not []:
            #     b = get_object_or_404(user=request.user, products=product_obj).delete()
            #     b = Viewed_Product.objects.filter(user=request.user, products=prod_two)
            #     del b
            global prod_one, prod_two

            if product_obj != prod_one and product_obj != prod_two:
                global prod_three 
                # if prod_three is not []:
                #     Viewed_Product.objects.remove(prod_three)
      
                prod_three = []

                # global prod_two
                prod_three = prod_two

                # global prod_one
                prod_two = prod_one    

                prod_one = product_obj

                print('prod_one: ', prod_one)
                print('prod_two: ', prod_two)
                print('prod_three: ', prod_three)                


        if product_id is None:
            print('product_id is None')        
            request.session['viewed_product_items'] = viewed_product_obj.products.count()
        print(' 7  - PORDUCT CoUNT')    
        # print(len(viewed_product_obj.products))    
        # print(slug)
        # self.object = self.get_object() 
        # context = super(ProductDetailSlugView, self).get_context_data(*args, **kwargs)
        # print(context)
        return instance


    def viewed_product_home(request):
        request = self.request   
        print('viewedeeee_product_obj')
        
        viewed_product_obj, new_obj = Viewed_Product.objects.new_or_get(self.request)

        # return render(request, "viewed_products/home.html", {"viewed_product": viewed_product_obj})
        print('viewed_product_obj')
        print(viewed_product_obj)

        return viewed_product_obj, new_obj


#models.py file 

import uuid

from django.conf import settings
from django.db import models
# from django.db.models.signals import pre_save, post_save, m2m_changed

from products.models import Product


User = settings.AUTH_USER_MODEL

class Viewed_Product_Manager(models.Manager):
    def new_or_get(self, request):
        viewed_product_id = request.session.get("viewed_product_id", None)
        qs = self.get_queryset().filter(id=viewed_product_id)
        if qs.count() == 1:
            new_obj = False
            viewed_product_obj = qs.first()
            if request.user.is_authenticated and viewed_product_obj.user is None:
                viewed_product_obj.user = request.user
                viewed_product_obj.save()
        else:
            viewed_product_obj = Viewed_Product.objects.new(user=request.user)
            new_obj = True
            request.session['viewed_product_id'] = viewed_product_obj.id
        return viewed_product_obj, new_obj

    def new(self, user=None):
        user_obj = None
        if user is not None:
            if user.is_authenticated:
                user_obj = user
        return self.model.objects.create(user=user_obj)

    def foo():
    	print('this is from the Viewed_Product model!!')

class Viewed_Product(models.Model):
    user        = models.ForeignKey(User,  on_delete=models.CASCADE, null=True, blank=True)
    # products    = models.SlugField(Product, blank=True)
    # products    = models.CharField(max_length=200)
    # products    = models.SlugField(Product, blank=True)

    

	# slug 		= models.SlugField(blank=True, unique=True)

    # subtotal    = models.DecimalField(default=0.00, max_digits=100, decimal_places=2)
    # total       = models.DecimalField(default=0.00, max_digits=100, decimal_places=2)
    # updated     = models.DateTimeField(auto_now=True)
    # timestamp   = models.DateTimeField(auto_now_add=True)

    objects = Viewed_Product_Manager()

    # def __str__(self):
    #     return str(self.id)

class Viewed_Product_Object(models.Model):
    # id          = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    user        = models.ForeignKey(Viewed_Product,  on_delete=models.CASCADE)
    products    = models.CharField(max_length=200)

    # def __str__(self):
    #     return '%s %s' % (self.user, self.products)


{% endhighlight %}


