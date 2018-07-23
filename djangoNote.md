# Django初級


## 1. 

---

1. 建立虛擬環境

    \> python -m venv v01      

    \> cd v01
    
    \> Scripts\activate.bat      

    安裝Django

    \(v01)> pip install Django==1.11.4

    查看安裝的django版本

    1. \(v01)> python -m django --version

        1.11.4

        或者

    2. \(v01)> python

        \>>> import django

        \>>> print(django.get_version())

        1.11.4

---

2. 建立Django專案

    建立Django專案, 名為mySite

    (v01)> django-admin runserver mySite 

    啟動後,架構如下
    ```
    mySite/                     專案的Container名稱
        manage.py               django專案底下, 引用到的套件都在這
        mySite/                 專案中實際的Python套件, 我稱它為 主專案/外層/專案層
            __init__.py         (未知)
            settings.py         組態、設定檔
            urls.py             專案層級的路由(紀錄各個服務的位置)
            wsgi.py             Web Server Gateway Interface; WSGI相容伺服器設定檔案
    ```

    (v01)> cd mySite

    啟動Django Server, 開放8000 port 給全世界
    
    (v01)\mySite> python manage.py runserver 0:8000
    ```
    You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
    Run 'python manage.py migrate' to apply them.
    August 27, 2017 - 12:33:33
    Django version 1.11.4, using settings 'mySite.settings'
    Starting development server at http://0:8000/
    Quit the server with CTRL-BREAK.
    ```

    上方區塊講了2件事情

    1. 目前專案仍沒有跟資料庫做串接

    2. 可透過網頁「127.0.0.1:8000」來存取

---

3. 建立專案底下的app (以新增靜態網頁為例)

    (v01)\mySite> python manage.py startapp blog
    
    建立後,架構如下
    ```
    mySite/                     整個專案的外殼(可以任意改名字)
        manage.py           
        mySite/                 核心專案套件 (專案層/外層)
            __init__.py     
            settings.py     
            urls.py         
            wsgi.py         
        blog/                  應用程式套件 (app層/內層)
            __init__.py
            admin.py	        後台管理介面 ($ python manage.py createsuperuser )
            apps.py             (未知)
            models.py	        app要用到的資料模型ORM
            tests.py	        (未知)
            views.py	        紀錄要render的網頁及其回傳之參數
            urls.py             為自己建立,建議將urls由個別app自己掌控要導向的views, 而專案曾urls負責導向app層級的urls
    ```

    blog為附加上去的app, 瀏覽器透過 urls(路由), 幫忙導向 Views(Controller), 分發相關Templates(回傳xx.html)以及要存取哪個Models(藉由ORM存取Database)

    為了讓**主專案**擁有blog這個服務, 我們需要讓主專案認識blog, 再來, 讓主專案知道blog在哪邊, 再來, 若有人想使用blog時, 讓主專案知道該怎麼回應

    - 讓主專案認識blog

        修改 mySite/mySite/settings.py (增加其中一行到此程式碼裡面)
        ```  
        # Application definition

        INSTALLED_APPS = [
            'blog',                    # 加入此行
            'django.contrib.admin',
            'django.contrib.auth',
            'django.contrib.contenttypes',
            'django.contrib.sessions',
            'django.contrib.messages',
            'django.contrib.staticfiles',
        ]
        ```
    - 讓主專案知道blog在哪邊

        修改 mySite/mySite/urls.py (增加其中幾行到此程式碼裡面)
        ```
        from django.conf.urls import url
        from django.conf.urls import include  # 加入此行
        from django.contrib import admin

        urlpatterns = [
            url(r'^admin/', admin.site.urls),

            # 讓blog這個app的路由, 重新導向blog app底下的urls
            # 若將來人家瀏覽 xxx/blog/ 則到blog.urls底下去找相關檔案
            url(r'^blog/', include('blog.urls')), # 加入此行
        ]
        ```

        新增 mySite/blog.urls.py (新增之內容如下)
        ```
        from django.conf.urls import url
        from . import views # 到同資料夾的views找相關資源

        urlpatterns = [
            # 以views.post_list裡面記載的方法來做處理
            url(r'^$', views.post_list, name='post_list')
        ]
        ```

    - 讓主專案知道該怎麼回應

        1. 修改 mySite/blog/views.py (將內容改成下面那樣)

        ```
        from django.shortcuts import render

        # 增加一個名為post_list的方法, 此方法回傳blog/post_list.html的頁面
        def post_list(request):
            return render(request, 'blog/post_list.html', {})
        ```

        2. 新增 mySite/blog/templates/blog/post_list.html

        ```
        <html>
            <head>
                <title>Tony's blog</title>
            </head>
            <body>
                <h1><a href="">Tony's Blog</a></h1>
                <div>
                    <h2>第1篇po文</h2>
                    <p>發佈時間: 08.27.2017, 17:07</p>
                    <p>這是Django寫起來的靜態網頁所模擬的Blog, 所以整個網頁是寫死的!!</p>
                </div>

                <div>
                    <h2>第2篇po文</h2>
                    <p>發佈時間: 08.27.2017, 17:10</p>
                    <p>沒想到寫這份說明文件那麼廢時= =</p>
                </div>
            </body>
        </html>
        ```

(v01)\mySite>python manage.py startproject 0:8000

到時候瀏覽網頁時, 127.0.0.1:8000/blog ,就可以看到剛剛的post_list.html了~~

---

4. 串接後端資料庫 (以sqlite為例)

    因為新增的blog, 將來會「有人」留言, 這就會牽扯到, 誰可以來留言? 當然是有註冊的會員囉!!  所以還得新增會員註冊系統
    
    另外要有資料庫紀載什麼人留言、留些什麼、哪時候留言, 接著要有顆資料庫把他們存下來，
    
    再來告訴主專案, 讓django可以把資料庫的東西show出來到網頁上(資料庫與網頁串接)

    - 新增會員註冊系統

        (v01)\mySite>python manage.py createsuperuser

        接著進入網頁「127.0.0.1:8000/admin」

    - 修改 mySite/blog/models.py 為下面程式碼
        ```
        from django.db import models
        from django.utils import timezone

        class Post(models.Model):
            author=models.ForeignKey('auth.User')                       # 留言的人
            title=models.CharField(max_length=32)                       # 留言標題
            text=models.TextField()                                     # 留言內容
            publish_datetime = models.DateTimeField(auto_now_add=True)  # 首次發佈日期時間
            modify_datetime = models.DateTimeField(auto_now=True)       # 最近修改日期時間

            def __str__(self):
                return self.title

            # 此 model 制定一個類別, 紀錄日後留言訪客的一些流言資訊
        ```
    - 讓django幫你紀錄目前 models.py 所寫下將來期望的 table 樣式(只是做版本控管, 資料庫並未有所改變)

        (v01)\mySite>python manage.py makemigrations blog

    - 串接資料庫(資料庫會依照 models.py 來做相對應的改變)

        (v01)\mySite>python manage.py migrate

    - 


● 開始寫後端網頁!!  在polls/views.py輸入(別把原本的東西蓋掉...)
----------------------------------------------------------------------
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
----------------------------------------------------------------------

以上是建立一個view (人看的到的網頁), 但是需要有東西記錄它在哪裡(描述URN位置)

$ touch urls.py (polls/views.py)
建立一個urls.py檔案, 裡面準備紀錄相關網址資訊
from django.conf.urls import url
from . import views
urlpatterns = [
    url(r'^$', views.index, name='index'),
]

============================================================================================
●告訴整個專案, 我剛剛新增了polls專案, 而底下也註冊好相對應的網址了
修改mysite/urls.py
改成下面的樣子
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^polls/', include('polls.urls')),
    url(r'^admin/', admin.site.urls),
]
↑新增一行而已...
重新啟動django
127.0.0.1:8000/polls   <- 就可以進入剛剛的頁面了!!


使用django,並串連database
https://docs.djangoproject.com/en/1.11/intro/tutorial02/

●選擇要串連那種資料庫?
開啟mysite/settings.py, 找到「DATABASES = 」那段
例如要改用MySQL, 則
註解這行「'ENGINE': 'django.db.backends.sqlite3'」
加入這行「'ENGINE': 'django.db.backends.mysql'」

而'NAME'那行, 就是要使用的Database名稱
本部分因教學, 故不動作


●執行串接資料庫
$ python manage.py migrate
此指令會去尋找 mysite/settings.py底下的INSTALLED_APPS，並且建立相對的table於
「...\django\mysite\」就會出現「db.sqlite3」



●新增pop app的models
於polls/models.py內容如下

from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

藉此，django即可
1.建立相關的table schema
2.建立Python database-access API來存取Question和Choice
但我們需要告知我們的project, polls app已經安裝好了

故, mysite/settings.py的部分，需要新增'polls.apps.PollsConfig'
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]


●測試整合
$ python manage.py sqlmigration polls 0001
會出現一堆
BEGIN;
--
-- Create model Choice
--
CREATE TABLE "polls_choice" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "choice_text" varchar(200) NOT NULL, "votes" integer N
OT NULL);
--
-- Create model Question
--
CREATE TABLE "polls_question" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "question_text" varchar(200) NOT NULL, "pub_date" da
tetime NOT NULL);
--
-- Add field question to choice
--
ALTER TABLE "polls_choice" RENAME TO "polls_choice__old";
CREATE TABLE "polls_choice" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "choice_text" varchar(200) NOT NULL, "votes" integer N
OT NULL, "question_id" integer NOT NULL REFERENCES "polls_question" ("id"));
INSERT INTO "polls_choice" ("id", "choice_text", "votes", "question_id") SELECT "id", "choice_text", "votes", NULL FROM "polls_choic
e__old";
DROP TABLE "polls_choice__old";
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");
COMMIT;

此指令只是測試整合後,讓他印出來告知說做了什麼,可以用來做事前check用

用來測試專案建置的models是否有問題
$ python manage.py check

●更新models
$ python manage.py makemigrations polls
用來告訴django,我已經對models做了改變
(改變的部分被放到了polls/migrations/0001_initial.py)

●做整合
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK

●簡言之,
1.更改models(models.py)

2.建立migrations紀錄變更的動作
$ python manage.py makemigrations 

3.將更新的部分套用至database
$ python manage.py migrate


●執行python互動模式
$ python manage.py shell



●建立admin
$ python manage.py createsuperuse
建立一個可以登入的pu
  cool21540125@hotmail.com
  tony
  no1



瀏覽網頁登入資訊
http://127.0.0.1:8000/admin/


============================================================================================
● 註冊model到admin
project/app1/admin.py
----------------------------------------------------------------------
from django.contrib import admin

# Register your models here.
from django.contrib import admin
from musics.models import Music

admin.site.register(Music)
----------------------------------------------------------------------
註冊後的好處是, 可以透過後台(網頁)來控管db

============================================================================================
●告知admin說Question物件有admin interface
於polls/admin.py修改
from django.contrib import admin

from .models import Question

admin.site.register(Question)


重新整理網頁之後, 就會出現
POLLS
 Questions
 的app了

