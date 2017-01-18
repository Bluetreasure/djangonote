##Django Model 閱讀筆記

+   Field有null,blank,choices,default,help_text,primary_key,unique 等參數可以調整
+   主要要注意的為primary_key通常建議不要為True，django內建會設一組readonly id為primary_key

+   當建立多對一的關係時會使用Foreign_key

``
poll = models.ForeignKey(
    Poll,
    on_delete=models.CASCADE,
    verbose_name="the related poll",
)
sites = models.ManyToManyField(Site, verbose_name="list of sites")
place = models.OneToOneField(
    Place,
    on_delete=models.CASCADE,
    verbose_name="related place",
)
``
