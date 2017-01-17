
#app的模式
+   盡量將一個專案分割成多個應用
+   是否使用第三方建構好的app
    
#模型

+   分割models.py,透過新增一個models資料夾model分割成多個檔案,在 __init__.py 維護from  import 
+   設計models須將模型正規化
    正規化大致分為三個
    (1)避免同一個欄位出現多個值，每個表都要有Key
    (2)去除掉部分功能相依，例如欄位裡面有廠商+廠商地址,因為是獨立的所以需要拉出來另外成一個表
    (3)欄位與主要索引沒有遞移相依的狀況，
+   django 不支援組合key,只能使用 Meta類中指定unique_together屬性 + ForeignKey
+   有時過多的規範化不是個好選擇，在需要考慮效能的時候往往會因為串聯過多表讓查詢時間過久
+   當有某些欄位需要在很多個classs裡面重複新增時，可以考慮增加abstract class讓其他class 可以繼承.
    例如當遇到需要在每個model增加修改時間跟新增時間以及新增人員的時候，可以透過下面方式去達成
```python
class Postable(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    modified = modified.DateTimeField(auto_now=True)
    message = models.TextField(max_length=500)
    class Meta:
        abstract = True
class Post(Postable):
    ...
```
+   在建立models.py時可以把models.py 改完一個dir models
+   透過init 把每一個model獨立寫成一個py檔
```python
'''__init__.py'''
    from postable import Postable
    from post import Post
    from commnet import Comment
```
+   針對django 內建 user model ,當想要擴展時會使用one_to_one_去擴展
+   例如：
```python
class Profile(models.Model):
    user = models.OnToOneField(settings.AUTH_USER_MODEL, primary_key=True)
```
+   針對user model 的延伸，往往會需要因為user 的新增或刪除，去對延伸的Table做異動
+   此時可以使用signal根據model傳出的信號，去做異動．

+   在使用user model 延伸時，在admin通常使用inline方式去實現，讓用戶profile與user放在一起
```python
from django.contrib import admin
from .models import Profile
from django.contrib.auth.models import User


class UserProfileInline(admin.StackedInline):
    model = Profile


class UserAdmin(admin.UserAdmin):
    inlines = [UserProfileInline]


admin.site.unregister(User)
admin.site.register(User, UserAdmin)
```

+   當同一用戶有不同帳戶類型時
+   例如：男生/女生 分別要紀錄不同的資料
+   可以使用抽象class達成用戶類型的共用
```python
class BaseProfile(models.Model):
    USER_TYPES = (
        (0, 'Ordinary'),
        (1, 'SuperHero'),
    )

    user = models.OnToOneField(settings.AUTH_USER_MODEL, primary_key=True)
    user_type = models.IntegerField(max_length=1, null=True,    choices=USER_TYPES)
    bio = models.CharField(max_length=200, blank=True, null=True)

    def __str__(self):
        return "{}:{:.20}".format(self.user, self.bio or "")

    class Meta:
        abstract = True


class SuperHeroProfile(models.Model):
    origin = models.CharField(max_length=100, blank=True, null=True)

    class Meta:
        abstract = True


class OrdinaryProfile(models.Model):
    address = models.CharField(max_length=200, blank=True, null=True)

    class Meta:
        abstract = True


class Profile(SuperHeroProfile, OrdinaryProfile, BaseProfile):
    pass
```
