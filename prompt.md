
for  django "Order" model , give me schema markup like below for this django model
- create proper enums for choices field

```py

class Order(models.Model):  
    
    is_reseller = models.BooleanField(default=False)

    full_name = models.CharField(max_length=255, verbose_name='Full Name')
    mobile_number = models.CharField(max_length=15, verbose_name='Mobile Number')
    city = models.CharField(max_length=255, verbose_name='City/Municipality',blank = True, null = True)
    area = models.CharField(max_length=255, verbose_name='Area')
    delivery_address = models.TextField(verbose_name='Delivery Address')
    delivery_option = models.CharField(max_length=255, choices=[('cash_in_delivery', 'Cash in Delivery'), ('advance_payment', 'Advance Payment')])

    # New fields
    total_points = models.IntegerField(default=0)
    subtotal = models.DecimalField(max_digits=10, decimal_places=2,default = 0, verbose_name='Delivery Discount', blank = True, null = True)
    items_total = models.IntegerField(default=0)
    discount = models.DecimalField(max_digits=10, decimal_places=2,default = 0, verbose_name='Delivery Discount', blank = True, null = True)
    total_payment = models.DecimalField(max_digits=10, decimal_places=2, verbose_name='Total Payment',default = 0, blank = True, null = True)


    delivery_status = models.CharField(max_length=10, choices=[('pending','Pending'),('processing','Processing'),('completed','Completed'),('cancel','cancel')], default = 'pending' , blank = True , null = True)
    payment_status = models.CharField(max_length=255, choices=[ ('paid','Paid'), ('unpaid', 'Unpaid')], default = 'unpaid', blank = True , null = True)
    user = models.ForeignKey(CustomUser, on_delete=models.SET_NULL, blank = True , null = True)

    used_amount_from_user = models.DecimalField(max_digits=10, decimal_places=2, default=0.0)
    special_discount = models.DecimalField(max_digits=10, decimal_places=2, default=0.0)

    #  the amount that was given to referral from this order
    referral_commision_amount = models.DecimalField(max_digits=10, decimal_places=2, default=0.0)

```



```schema
Enum "products_remark_enum" {
  "popular"
  "new"
  "top"
  "special"
  "trending"
  "regular"
}

Table "brands" {
  "id" INT [pk, increment]
  "brand_name" VARCHAR(50) [not null]
  "brand_img" VARCHAR(300) [not null]
  "created_at" TIMESTAMP [default: `CURRENT_TIMESTAMP`]
  "updated_at" TIMESTAMP [default: `CURRENT_TIMESTAMP`]
}

Table "categories" {
  "id" INT [pk, increment]
  "category_name" VARCHAR(50) [not null]
  "category_img" VARCHAR(300) [not null]
  "created_at" TIMESTAMP [default: `CURRENT_TIMESTAMP`]
  "updated_at" TIMESTAMP [default: `CURRENT_TIMESTAMP`]
}

Table "products" {
  "id" INT [pk, increment]
  "title" VARCHAR(200) [not null]
  "short_des" VARCHAR(500) [not null]
  "price" VARCHAR(50) [not null]
  "points" INT [not null]
  "discount" BOOLEAN [not null]
  "discount_price" VARCHAR(50) [not null]
  "image" VARCHAR(200) [not null]
  "stock" BOOLEAN [not null]
  "star" FLOAT [not null]
  "remark" products_remark_enum [not null]
  "category_id" INT [ref: > categories.id]
  "brand_id" INT [ref: > brands.id]
  "img1" VARCHAR(200) [not null]
  "img2" VARCHAR(200) [not null]
  "img3" VARCHAR(200) [not null]
  "img4" VARCHAR(200) [not null]
  "des" LONGTEXT [not null]
  "color" VARCHAR(200) [not null]
  "size" VARCHAR(200) [not null]
  "created_at" TIMESTAMP [default: `CURRENT_TIMESTAMP`]
  "updated_at" TIMESTAMP [default: `CURRENT_TIMESTAMP`]
}

```
