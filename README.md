![](https://apiflask.com/_assets/apiflask-logo.png)

# ترجمه فارسی مستندات فلسک

[![Netlify Status](https://api.netlify.com/api/v1/badges/456180b4-55f0-417c-9c4e-88e407a24011/deploy-status)](https://app.netlify.com/sites/fa-apiflask/deploys)

## روش کار ترجمه

برای مشارکت در این ترجمه خوش آمدید!

در اینجا برخی از راهکار های اصلی ترجمه آمده است.

* یک پول ریکوئست ایجاد کنید تا نام خود را به فصلی که میخواهید ترجمه کنید، اضافه کنید.
* این مخزن([apiflask/docs-fa](https://github.com/apiflask/docs-fa/fork)) را فورک کنید و آن را کلون کنید. (`{username}` را با نام کاربری گیت‌هاب خودتان جایگزین کنید.):

```
$ git clone https://github.com/{username}/docs-fa
$ cd docs-fa
$ git remote add upstream https://github.com/apiflask/docs-fa
```

* [راهنمای مشارکت](/contributing) را برای ساخت محیط توسعه بخوانید(برای فورک این مرحله را نادیده بگیرید)
<!--* Read the translation guide (WIP) to understand the basic requirements on translation and wording. -->
* یک برنچ جدید برای ترجمه فایل های داخل `docs_fa/` ایجاد کنید.
* دستور `mkdocs serve`  را برای پیشنمایش و بررسی مشکلات احتمالی اجرا کنید.
* یک پول ریکوئست ارسال کنید و منتظر بررسی باشید.

```bash
$ pip install -r requirements/docs.txt
$ pip install -e .
$ mkdocs serve
```

## فصل ها

* [X] Home (index.md) [@mmdbalkhi](https://github.com/mmdbalkhi)
* [X] Documentation Index (docs.md) [@mmdbalkhi](https://github.com/mmdbalkhi)
* [X] Migrating from Flask (migrating.md) [@mmdbalkhi](https://github.com/mmdbalkhi)
* [ ] Basic Usage (usage.md)
* [ ] Request Handling (request.md)
* [ ] Response Formatting (response.md)
* [ ] Data Schema (schema.md)
* [ ] Authentication (authentication.md)
* [ ] OpenAPI Generating (openapi.md)
* [ ] Swagger UI and Redoc (api-docs.md)
* [ ] Configuration (configuration.md)
* [ ] Error Handling (error-handling.md)
* [ ] Examples (examples.md)
* [ ] Comparison and Motivations (comparison.md)
* [ ] Authors (authors.md)
* [ ] Changelog (changelog.md)
* [ ] Contributing Guide (contributing.md)
* [ ] API Reference:
    - [ ] APIFlask (api/app.md)
    - [ ] APIBlueprint (api/blueprint.md)
    - [ ] Exceptions (api/exceptions.md)
    - [ ] OpenAPI (api/openapi.md)
    - [ ] Schemas (api/schemas.md)
    - [ ] Fields (api/fields.md)
    - [ ] Validators (api/validators.md)
    - [ ] Route (api/route.md)
    - [ ] Security (api/security.md)
    - [ ] Helpers (api/helpers.md)
    - [ ] Commands (api/commands.md)
# مقدمه

APIFlask یک web-api فریمورک سبک بر اساس [فلسک](https://github.com/pallets/flask) و [marshmallow-code](https://github.com/marshmallow-code) است. APIFlask در استفاده ساده، قابل شخصی سازی، استفاده از ORM/ODM-agnostic و ۱۰۰٪ سازگار با اکوسیستم فلسک است.

با APIFlask شما میتوانید:

* دیکریتور های بیشتری برای ویو-فانکشن ها(`@app.input()`,  `@app.output()`,  `@app.get()`,  `@app.post()` و بیشتر)
* اعتبار سنجی خودکار درخواست ها و سریال زدایی
* سریال سازی و فرمت خودکار پاسخ ها
* ساخت خودکار [OpenAPI Specification](https://github.com/OAI/OpenAPI-Specification) (OAS, formerly Swagger Specification) تولید مستندات
* ایجاد مستندات تعاملی خودکار برای API
* پشتیبانی از احراز هویت API(با [Flask-HTTPAuth](https://github.com/miguelgrinberg/flask-httpauth))
* پاسخ های خودکار JSON برای ارور های HTTP

## پیشنیاز ها

* پایتون 3.7+
* فلسک 1.1.0+

## روش نصب

در Linux و macOS:

```bash
$ pip3 install apiflask
```

در ویندوز:

```bash
> pip install apiflask
```

## لینک ها

* وبسایت: <https://fa.apiflask.com>
* مستندات: <https://fa.apiflask.com/docs>
* ریلیز های PyPI: <https://pypi.python.org/pypi/APIFlask>
* گزارش تغییرات: <https://fa.apiflask.com/changelog>
* کد منبع: <https://github.com/apiflask/apiflask>
* ردیاب مشکلات: <https://github.com/apiflask/apiflask/issues>
* گفت و گو: <https://github.com/apiflask/apiflask/discussions>
* توئیتر: <https://twitter.com/apiflask>

## مثال

```python
from apiflask import APIFlask, Schema, abort
from apiflask.fields import Integer, String
from apiflask.validators import Length, OneOf

app = APIFlask(__name__)

pets = [
    {'id': 0, 'name': 'Kitty', 'category': 'cat'},
    {'id': 1, 'name': 'Coco', 'category': 'dog'}
]

class PetIn(Schema):
    name = String(required=True, validate=Length(0, 10))
    category = String(required=True, validate=OneOf(['dog', 'cat']))

class PetOut(Schema):
    id = Integer()
    name = String()
    category = String()

@app.get('/')
def say_hello():
    # برگرداندن یک دیکشنری یا لیست برابر با استفاده از jsonify()
    return {'message': 'Hello!'}

@app.get('/pets/<int:pet_id>')
@app.output(PetOut)
def get_pet(pet_id):
    if pet_id > len(pets) - 1:
        abort(404)
    # همچنین می توانید یک نمونه کلاس مدل ORM/ODM را مستقیماً برگردانید
    # APIFlask آبجکت را به فرمت JSON سریال می کند
    return pets[pet_id]

@app.patch('/pets/<int:pet_id>')
@app.input(PetIn(partial=True))
@app.output(PetOut)
def update_pet(pet_id, data):
    # داده های ورودی تایید شده و تجزیه شده خواهد بود
    # به عنوان دیکشنری به تابع view تزریق شود
    if pet_id > len(pets) - 1:
        abort(404)
    for attr, value in data.items():
        pets[pet_id][attr] = value
    return pets[pet_id]
```

<details>
<summary>همچنین میتوانید از ویو های کلاس-بیس با <code>MethodView</code> استفاده کنید</summary>

```python
from apiflask import APIFlask, Schema, abort
from apiflask.fields import Integer, String
from apiflask.validators import Length, OneOf
from apiflask.views import MethodView

app = APIFlask(__name__)

pets = [
    {'id': 0, 'name': 'Kitty', 'category': 'cat'},
    {'id': 1, 'name': 'Coco', 'category': 'dog'}
]

class PetIn(Schema):
    name = String(required=True, validate=Length(0, 10))
    category = String(required=True, validate=OneOf(['dog', 'cat']))

class PetOut(Schema):
    id = Integer()
    name = String()
    category = String()

class Hello(MethodView):

    # از نام متود HTTP به عنوان نام متد کلاس استفاده کنید
    def get(self):
        return {'message': 'Hello!'}

class Pet(MethodView):

    @app.output(PetOut)
    def get(self, pet_id):
        """Get a pet"""
        if pet_id > len(pets) - 1:
            abort(404)
        return pets[pet_id]

    @app.input(PetIn(partial=True))
    @app.output(PetOut)
    def patch(self, pet_id, data):
        """Update a pet"""
        if pet_id > len(pets) - 1:
            abort(404)
        for attr, value in data.items():
            pets[pet_id][attr] = value
        return pets[pet_id]

app.add_url_rule('/', view_func=Hello.as_view('hello'))
app.add_url_rule('/pets/<int:pet_id>', view_func=Pet.as_view('pet'))
```

</details>

<details>
<summary>یا میتوانید از  <code>async def</code> با فلسک ۲.۰ استفاده کنید</summary>

```bash
$ pip install -U flask[async]
```

```python
import asyncio

from apiflask import APIFlask

app = APIFlask(__name__)

@app.get('/')
async def say_hello():
    await asyncio.sleep(1)
    return {'message': 'Hello!'}
```

<em><a href="https://flask.palletsprojects.com/async-await">استفاده از async و await</a></em> برای اطلاعات بیشتر از async def در فلسک ۲.+ ببینید

</details>

این را با نام `app.py` ذخیره کنید و به صورت زیر اجرا کنید.

```bash
$ flask run --reload
```

و حالا میتوانید مستندات API (Swagger UI) را در <http://localhost:5000/docs> ببینید:

![](https://apiflask.com/_assets/swagger-ui.png)

یا میتوانید UI مستندات API را توسط پارامتر `docs_ui` در هنگام ساخت کلاس APIFlask تغییر دهید.

```py
app = APIFlask(__name__, docs_ui='redoc')
```

حالا مستندات API توسط Redoc در <http://localhost:5000/docs> رندر خواهد شد.

مقادیر پشتیبانی شده از `docs_ui` عبارت اند از:

* `swagger-ui` (مقدار پیشفرض): [Swagger UI](https://github.com/swagger-api/swagger-ui)
* `redoc`: [Redoc](https://github.com/Redocly/redoc)
* `elements`: [Elements](https://github.com/stoplightio/elements)
* `rapidoc`: [RapiDoc](https://github.com/rapi-doc/RapiDoc)
* `rapipdf`: [RapiPDF](https://github.com/mrin9/RapiPdf)

فایل OpenAPI spec به تولید خودکار در <http://localhost:5000/openapi.json> در دسترس است. همچنین شما میتوانید spec را با [دستور `flask spec` ](https://apiflask.com/openapi/#the-flask-spec-command) دریافت کنید.

```bash
$ flask spec
```

برای سایر مثال های کاملتر [/examples](https://github.com/apiflask/apiflask/tree/main/examples) ببینید.

## رابطه با فلسک

APIFlask یک بسته بندی کوچک بروی فلسک است. شما فقط باید تفاوت های زیر را به خاطر بسپارید(برای اطلاعات بیشتر *[Migrating from Flask](https://apiflask.com/migrating)* را ببینید):

* هنگام ساخت یک برنامه بجای `Flask` از `APIFlask` استفاده کنید
* هنگام ساخت بلوپرینت ها بجای `Blueprint` از `APIBlueprint` استفاده کنید.
* تابع `abort()` در APIFlask(`apiflask.abort`)  ارور ها را به صورت JSON بر میگرداند.

برای یک برنامه مینیمال فلسک:

```python
from flask import Flask, request, escape

app = Flask(__name__)

@app.route('/')
def hello():
    name = request.args.get('name', 'Human')
    return f'Hello, {escape(name)}'
```

و حالا در APIFlask:

```python
from apiflask import APIFlask  # قدم اول
from flask import request, escape

app = APIFlask(__name__)  # قدم دوم

@app.route('/')
def hello():
    name = request.args.get('name', 'Human')
    return f'Hello, {escape(name)}'
```

در یک کلمه، برای توسعه راحتر web-api، APIFlask، `APIFlask` و `APIBlueprint` را فراهم میکند تا آبجکت های `Flask` و `Blueprint` را گسترش دهد و همچنین با برخی از ابزار های مفید همراه است. به جز این شما در از فلسک استفاده میکنید.

## رابطه با مارشمالو

Apiflask طرحواره مارشمالو را به عنوان طرحواره داده می پذیرد ، از WebARGS برای تأیید داده های درخواست در برابر طرح طرحواره می کند و از APISPEC برای تولید بازنمایی OpenAPI از طرح استفاده می کند.
شما می توانید مانند گذشته طرح های مارشمالو را بسازید ، اما Apiflask همچنین برخی از API های مارشمالو را برای راحتی در معرض دید قرار می دهد (این اختیاری است ، شما هنوز هم می توانید همه چیز را مستقیماً از مارشمالو وارد کنید):

* `apiflask.Schema`: کلاس پایه طرحوارهی مارشمالو
* `apiflask.fields`: فیلد های ماشرمالو، شامل هر دوی مارشمالو و Flask-Marshmallow می‌شود. مراقب باشید که نام های مستعار (`Url`,     `Str`,     `Int`,     `Bool`, غیره) حذف شده اند.
* `apiflask.validators`: اعتبارسنج های مارشمالو

</p>

```python
from apiflask import Schema
from apiflask.fields import Integer, String
from apiflask.validators import Length, OneOf
from marshmallow import pre_load, post_dump, ValidationError
```

## اساس

APIFlask یک فورک از [APIFairy](https://github.com/miguelgrinberg/APIFairy) است و از [flask-smorest](https://github.com/marshmallow-code/flask-smorest) و [FastAPI](https://github.com/tiangolo/fastapi) ( *[Comparison and Motivations](https://apiflask.com/comparison)*  برای مقایسه بین پروژه ها ببینید) الهام گرفته است.
