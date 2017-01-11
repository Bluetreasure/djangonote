
#app的模式

    * 盡量將一個專案分割成多個應用
    * 是否使用第三方建構好的app
    
#模型

    * 分割models.py,透過新增一個models資料夾model分割成多個檔案,在 __init__.py 維護from .... import ...
    * 設計models須將模型正規化
      正規化大致分為三個
      (1)避免同一個欄位出現多個值，每個表都要有Key
      (2)去除掉部分功能相依，例如欄位裡面有廠商+廠商地址,因為是獨立的所以需要拉出來另外成一個表
      (3)欄位與主要索引沒有遞移相依的狀況，
    * django 不支援組合key,只能使用 Meta類中指定unique_together屬性 + ForeignKey
    * 有時過多的規範化不是個好選擇，在需要考慮效能的時候往往會因為串聯過多表讓查詢時間過久
    * 當有某些欄位需要在很多個classs裡面重複新增時，可以考慮增加abstract class讓其他class 可以繼承.
      例如當遇到需要在每個model增加修改時間跟新增時間以及新增人員的時候，可以透過下面方式去達成
      class Postable(models.Model):
         created = models.DateTimeField(auto_now_add=True)
         modified = modified.DateTimeField(auto_now=True)
         message = models.TextField(max_length=500)

         class Meta:
            abstract = True

      class Post(Postable):
         ...
