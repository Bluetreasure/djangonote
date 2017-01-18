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

+   在一對一關係時,我們可以適用OneToOneField去針對某個Model做額外資訊Table的應用
```
from django.db import models

class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

    def __str__(self):              # __unicode__ on Python 2
        return "%s the place" % self.name

class Restaurant(models.Model):
    place = models.OneToOneField(
        Place,
        on_delete=models.CASCADE,
        primary_key=True,
    )
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)

    def __str__(self):              # __unicode__ on Python 2
        return "%s the restaurant" % self.place.name

class Waiter(models.Model):
    restaurant = models.ForeignKey(Restaurant, on_delete=models.CASCADE)
    name = models.CharField(max_length=50)

    def __str__(self):              # __unicode__ on Python 2
        return "%s the waiter at %s" % (self.name, self.restaurant)
```

+   在模型欄位的定義不可以使用pass以及多個下劃線___

+   要客製化FieldType可以透過以下網址去實現[Writing custom model fields.](https://docs.djangoproject.com/en/1.10/howto/custom-model-fields/)

+   模型可以定義Meta選項，可以包含ordering,verbosename等等 實際可以參考[model option reference.](https://docs.djangoproject.com/en/1.10/ref/models/options/)

+   在Model 裡可以自行定義不同方法，例如根據不同資料新增的狀況取得不同訊息
```
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    birth_date = models.DateField()

    def baby_boomer_status(self):
        "Returns the person's baby-boomer status."
        import datetime
        if self.birth_date < datetime.date(1945, 8, 1):
            return "Pre-boomer"
        elif self.birth_date < datetime.date(1965, 1, 1):
            return "Baby boomer"
        else:
            return "Post-boomer"

    def _get_full_name(self):
        "Returns the person's full name."
        return '%s %s' % (self.first_name, self.last_name)
    full_name = property(_get_full_name)
```

+   Models也有一些自定義的Method ex: __str__(), __unicode__()

+   當我們希望改寫Models自定義的Method時，可以透過以下例子複寫
```
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, *args, **kwargs):
        if self.name == "Yoko Ono's blog":
            return # Yoko shall never have her own blog!
        else:
            super(Blog, self).save(*args, **kwargs) # Call the "real" save() method.
```

+   abstract class 可以避免一直重複寫入某欄位
```
from django.db import models

class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
```

+   Meta繼承的擴充時可以透過以下方式進行
```
from django.db import models

class CommonInfo(models.Model):
    # ...
    class Meta:
        abstract = True
        ordering = ['name']

class Student(CommonInfo):
    # ...
    class Meta(CommonInfo.Meta):
        db_table = 'student_info'
```

##Making Query

+   Model create
+   透過以下方式可以做到model新增

```
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```
+   要記得異動完要save()才會記錄在database
+   Model Update

```
>>> b5.name = 'New name'
>>> b5.save()
```
+   透過直接更新attr ,save()即可
