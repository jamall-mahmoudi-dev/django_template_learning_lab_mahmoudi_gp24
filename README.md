# django_template_learning_lab

یک وبلاگ ساده برای یادگیری عملی   Template Tags، Template Filters، Custom Tag/Filter، Inclusion Tag، Sidebar و Pagination   در Django.

>   نسخه‌ها:   Python 3.10 تا 3.13 و Django 5.2 (نسخه LTS). فایل   requirements.txt   نسخه را قفل می‌کند.
>   وضعیت تست:   کد و تست‌ها نوشته شده‌اند، اما در محیط ساخت این پروژه امکان نصب Django وجود نداشت؛ پس تست‌ها   اجرا نشده‌اند  . لطفاً در مرحله ۱۴ خودتان اجرا کنید.

---

## ۱. هدف آموزشی
پس از اجرای پروژه باید بتوانید توضیح دهید هر ابزار چه می‌کند و چه فرقی با بقیه دارد.

## ۲. پیش‌نیازها
Python نصب‌شده (  python --version  ) و آشنایی ابتدایی با ترمینال.

## ۳. ساخت محیط مجازی
  bash
python -m venv venv
# Windows:
venv\Scripts\activate
# macOS / Linux:
source venv/bin/activate
  
محیط مجازی پکیج‌های این پروژه را از بقیه پروژه‌ها جدا نگه می‌دارد.

## ۴. نصب Django
  bash
pip install -r requirements.txt
  

## ۵. ایجاد پروژه و اپلیکیشن (برای ساخت از صفر)
این پروژه آماده است، اما ساختار آن با این دو دستور ساخته می‌شود:
  bash
django-admin startproject config .   # پوشه تنظیمات پروژه
python manage.py startapp blog       # اپلیکیشن blog
  

## ۶. تنظیم INSTALLED_APPS
در   config/settings.py   عبارت   "blog"   به   INSTALLED_APPS   اضافه شده است. بدون آن، Django قالب‌ها و   templatetags   اپ را پیدا نمی‌کند.

## ۷. Migrationها
  bash
python manage.py migrate
  
فایل   blog/migrations/0001_initial.py   جداول Category و Post را می‌سازد. اگر مدل‌ها را تغییر دادید:   python manage.py makemigrations   و دوباره   migrate  .
برای اطمینان از هم‌خوانی مدل‌ها با migration:   python manage.py makemigrations --check   (باید «No changes detected» بدهد).

## ۸. ساخت Superuser
  bash
python manage.py createsuperuser
  

## ۹. اجرای سرور
  bash
python manage.py runserver
  

## ۱۰. داده‌های آزمایشی
  bash
python manage.py seed_data
  
۳ دسته‌بندی و ۱۳ پست می‌سازد (Django: ۶، Python: ۴، Web: ۳). چندبار اجرا کنید، تکراری ساخته نمی‌شود، چون با   get_or_create   عنوان پست بررسی می‌شود.

## ۱۱. آدرس صفحات
| آدرس | چه چیزی را می‌بینید |
|---|---|
| http://127.0.0.1:8000/ | فهرست پست‌ها، Sidebar، Pagination (۳ صفحه) |
| http://127.0.0.1:8000/?page=2 | صفحه دوم |
| http://127.0.0.1:8000/?page=abc یا   ?page=999   | رفتار   get_page()   برای صفحه نامعتبر |
| http://127.0.0.1:8000/?category=1 | فیلتر دسته‌بندی (شناسه را از Sidebar ببینید) |
| http://127.0.0.1:8000/?category=1&page=2 | فیلتر + صفحه‌بندی |
| http://127.0.0.1:8000/post/1/ | جزئیات پست |
| http://127.0.0.1:8000/filters/ | جدول خروجی فیلترها |
| http://127.0.0.1:8000/subscribe/ | فرم POST با   csrf_token   |
| http://127.0.0.1:8000/admin/ | پنل مدیریت |

## ۱۲. فایل‌های مهم
  
manage.py
config/settings.py, urls.py
blog/
  models.py                  ← ساختار داده (Category, Post)
  views.py                   ← منطق: فیلتر، Paginator، ارسال داده به قالب
  urls.py                    ← نام‌گذاری آدرس‌ها (blog:post_list ...)
  admin.py                   ← پنل ادمین
  templatetags/blog_tags.py  ← filter و tag سفارشی + inclusion tag
  management/commands/seed_data.py
  tests.py
  static/blog/css/style.css
  templates/blog/
    base.html, post_list.html, post_detail.html, subscribe.html, filters_demo.html
    partials/_post_card.html, pagination.html, category_sidebar.html
  
  مسئولیت‌ها:   Model = ساختار و ذخیره داده · View = تصمیم‌گیری و آماده‌سازی داده · Template = فقط نمایش. منطق پیچیده را داخل Template ننویسید.

  رابطه Category و Post:   هر پست دقیقاً یک دسته دارد، ولی هر دسته چند پست دارد (ForeignKey، «یک به چند»).   on_delete=PROTECT   یعنی دسته‌ای که پست دارد حذف نمی‌شود، تا پست‌ها ناخواسته پاک نشوند. (  CASCADE   همراه دسته، پست‌ها را هم حذف می‌کند.)

---

## ۱۳. مفاهیم با مثال

### Template Tagهای داخلی
| تگ | مثال در پروژه | کاربرد |
|---|---|---|
|   if/else   | عنوان   post_list.html   | نمایش شرطی |
|   for/empty   | حلقه پست‌ها در   post_list.html   | تکرار؛   empty   وقتی فهرست خالی است |
|   extends/block   | همه قالب‌ها از   base.html   | ارث‌بری؛ ظاهر مشترک یک‌بار نوشته می‌شود |
|   include   |   _post_card.html  ,   pagination.html   | استفاده مجدد از یک تکه HTML |
|   url   | لینک‌ها مثل   {% url 'blog:post_detail' post.pk %}   | ساخت آدرس از روی نام؛ تغییر آدرس لینک‌ها را خراب نمی‌کند |
|   load static   /   static   | CSS در   base.html   | آدرس فایل‌های استاتیک |
|   csrf_token   |   subscribe.html   | محافظت از فرم POST (بدون آن خطای 403) |
|   with   |   _post_card.html   | متغیر موقت (تا   endwith  ) |

### Template Filterها (صفحه   /filters/  )
| فیلتر | ورودی | خروجی |
|---|---|---|
|   upper   |   "django learner"   |   DJANGO LEARNER   |
|   lower   |   "DJANGO"   |   django   |
|   title   |   "django learner"   |   Django Learner   |
|   length   |   ["django","python","html"]   |   3   |
|   truncatewords:5   | جمله ۱۲ کلمه‌ای | ۵ کلمه اول +   …   |
|   default:"ناشناس"   |   None   یا خالی |   ناشناس   |
|   date:"Y/m/d"   | تاریخ | مثلاً   2026/10/24   |
|   join:" - "   |   ["django","python","html"]   |   django - python - html   |

### فیلتر سفارشی
۱. پوشه   templatetags   با فایل خالی   __init__.py   داخل اپ بسازید.
۲. در   blog_tags.py  :
  python
from django import template
register = template.Library()

@register.filter
def word_count(value):
    return len(str(value).split()) if value else 0
  
۳. در قالب:   {% load blog_tags %}   و سپس   {{ post.content|word_count }}  .
۴. بعد از ساخت فایل جدید در   templatetags  ، سرور را یک‌بار Restart کنید.

  فیلتر در برابر تگ:   فیلتر یک مقدار را می‌گیرد و شکل دیگری از همان مقدار را برمی‌گرداند (  {{ x|upper }}  ). تگ می‌تواند منطق/ساختار اجرا کند (شرط، حلقه، کوئری، رندر قالب).

### Include، Simple Tag، Inclusion Tag
| |   include   | Simple Tag (  total_posts  ) | Inclusion Tag (  category_sidebar  ) |
|---|---|---|---|
| منطق پایتون | ندارد (از context موجود) | دارد | دارد |
| خروجی | HTML یک فایل دیگر | یک مقدار (عدد/متن) | HTML یک قالب مخصوص |
| داده از کجا؟ | context همان View | محاسبه داخل تابع | کوئری داخل تابع تگ |

مثال:   {% include "blog/partials/_post_card.html" %}   فقط HTML را درج می‌کند.   {% total_posts %}   عدد کل پست‌ها را از دیتابیس می‌دهد.   {% category_sidebar selected_category %}   خودش دسته‌ها را از دیتابیس می‌گیرد و   category_sidebar.html   را رندر می‌کند.
  چرا برای Sidebar مناسب است؟   Sidebar در همه صفحات هست؛ با Inclusion Tag لازم نیست هر View دسته‌ها را بفرستد. فقط   base.html   یک خط می‌نویسد.

  چرا   annotate()   و   Count()  ؟     Category.objects.annotate(post_count=Count("posts"))   با یک Query تعداد همه دسته‌ها را می‌دهد. اگر در حلقه از   category.posts.count   استفاده کنیم، برای هر دسته یک Query جدا اجرا می‌شود (مشکل N+1).

### Pagination
-   چرا بهتر است؟   نمایش همه پست‌ها صفحه را سنگین و کند می‌کند؛ با صفحه‌بندی فقط ۵ پست خوانده و ارسال می‌شود.
-     Paginator  :   کل مجموعه را به صفحه‌ها تقسیم می‌کند (تعداد کل صفحات:   paginator.num_pages  ).
-     Page   /   page_obj  :   یک صفحه مشخص (آیتم‌ها،   number  ،   has_next  ،   next_page_number   ...).   page_obj   نامی است که ما به Page داده و به قالب می‌فرستیم.
-     get_page()  :     ?page=abc   ← صفحه اول؛   ?page=999   (یا   0   و منفی) ← صفحه آخر؛ بدون پارامتر ← صفحه اول. خطا نمی‌دهد.
- لینک‌های «قبلی/بعدی» فقط با   has_previous   /   has_next   نمایش داده می‌شوند.

### فیلتر دسته و Pagination
در   views.py     اول     filter()   و   بعد     Paginator  . اگر برعکس باشد، تعداد صفحات و محتوای صفحه‌ها با کل پست‌ها محاسبه می‌شود نه پست‌های آن دسته. در   pagination.html   هر لینک   &category=<id>   را هم دارد تا فیلتر هنگام تغییر صفحه حفظ شود.

---

## ۱۴. بررسی عملکرد
  bash
python manage.py test
  
تست‌ها (  blog/tests.py  ):
-   PostListTests  : بارگذاری فهرست، ۵ پست در هر صفحه (۵ و ۵ و ۲)، صفحه نامعتبر، فیلتر دسته، حفظ فیلتر در لینک‌ها، ۴۰۴ برای دسته ناموجود، پیام خالی بودن
-   SidebarTests  : شمارش پست‌ها در Inclusion Tag، نمایش در صفحه، پیام نبودن دسته‌بندی
-   CustomFilterTests  :   word_count   (متن عادی، فاصله اضافه، خالی،   None  )

بررسی دستی در مرورگر: صفحات جدول مرحله ۱۱ را باز کنید و ببینید جمله‌های «بعدی»، «قبلی» و شمار Sidebar درست‌اند.

## ۱۵. خطاهای رایج مبتدیان
| خطا | علت و راه‌حل |
|---|---|
|   Invalid filter: 'word_count'   /   'blog_tags' is not a registered tag library   | فراموشی   {% load blog_tags %}   در همان قالب (در قالب فرزند ارث‌بری نمی‌شود)، نبودن   __init__.py   در   templatetags  ، یا Restart نکردن سرور |
|   TemplateDoesNotExist   | مسیر قالب باید   blog/templates/blog/...   باشد و   blog   در   INSTALLED_APPS   ثبت شده باشد |
|   NoReverseMatch   | نام URL یا آرگومان اشتباه؛ با   app_name   باید   blog:post_list   نوشت |
| CSS اعمال نمی‌شود |   {% load static %}   فراموش شده یا مسیر   blog/css/style.css   نادرست است |
| 403 CSRF در فرم |   {% csrf_token %}   داخل   <form method="post">   نیست |
|   no such table   |   python manage.py migrate   اجرا نشده |
|   UnorderedObjectListWarning   | Paginator روی queryset بدون   order_by  ؛ اینجا ترتیب مشخص شده |
| صفحه‌بندی بعد از فیلتر فیلتر را از دست می‌دهد | لینک‌ها باید   &category=...   را هم داشته باشند |
