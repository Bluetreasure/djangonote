+ Django Csrf 在Django 1.10 的問題
    Django 1.10 版本的時候再csrf尚有比較大的限制
    之前使用{% csrf_token %} + render_context 的方法已經沒辦法通過csrf    
    之後改成以下寫法:
        return render(request, 'login.html',locals())
    才成功通過驗證

    http://www.cnblogs.com/Rocky_/p/6140362.html
    之後可以參考這篇
+ 透過django-registration 可以快速做到基本註冊跟認證
    但因為registration的form 跟 view 是寫死的 
    要客製化的話需要繼承class 
    或是直接import 重新寫
    這樣子速度不會比叫快
    但是因為有認證信功能可以達成不錯的認證效果

+ Django + Nginx + Gunicorn
    目前卡關在Gunicorn
    
+ Wsgi 的了解
+ Model 在應用時當需要用到一個class使用兩個相同model的foreign key 時，會需要用related_name去做區隔
    但會發生一個狀況，當我將共用class 區隔到外面時
    例如放在 utils 會導致在同一個app裡面的models 如果有兩個以上共用到這個class
    會發生reverse 問題 也就是related name重複
    這時候解決方法可以如下
```
class CommonInfo(models.Model):
    # 新增日期
    create_date = models.DateTimeField(editable=False, auto_now_add=True)
    # 資料新增者
    create_user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='%(class)s_create_user'
    )
    # 資料修改者
    modify_user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='%(class)s_modify_user'
    )
    # 資料修改日期
    modify_date = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True
```
可以透過%(class)s去做到把related_name特殊化
    
這樣做之後產生的model會變成
```
    ('create_user', models.ForeignKey(on_delete=django.db.models.deletion.CASCADE, related_name='slide_bar_pic_create_user', to=settings.AUTH_USER_MODEL)),
    ('modify_user', models.ForeignKey(on_delete=django.db.models.deletion.CASCADE, related_name='slide_bar_pic_modify_user', to=settings.AUTH_USER_MODEL)),
    ('pic_slide_bar', models.ForeignKey(on_delete=django.db.models.deletion.CASCADE, related_name='pic_slide_bar', to='slide_bar.Slide_Bar')),
```
