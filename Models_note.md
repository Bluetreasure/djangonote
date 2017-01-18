##Django Model 閱讀筆記

+   Field有null,blank,choices,default,help_text,primary_key,unique 等參數可以調整
+   主要要注意的為primary_key通常建議不要為True，django內建會設一組readonly id為primary_key

+   當建立多對一的關係時會使用Foreign_key

```
from django.db import models

class Manufacturer(models.Model):
    # ...
    pass

class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
    # ...
```

+   on_delete=models.CASCADE 代表當Foreignkey指向的目標刪除時,一併刪除
+   ManytoManyField 可以定義多對多的關係
```
from django.db import models

class Topping(models.Model):
    # ...
    pass

class Pizza(models.Model):
    # ...
    toppings = models.ManyToManyField(Topping)
```
+   ManytoManyField 在定義多對多關係時 若需要針對這個關係做額外的紀錄時
+   可以透過through
```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Membership(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)
```

+   上面的例子就是透過Membership紀錄他們之間關係的原因
