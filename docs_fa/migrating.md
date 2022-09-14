# مهاجرت از فلسک

از آنجایی که APIFlask بر اساس فلسک است، شما فقط نیاز به تغییر بسیار کمی دارید(معمولا کمتر از ده خط کد)

## کلاس `Flask` -> `کلاس APIFlask`

نحوه ایجاد یک برنامه در فلسک اینگونه است:

```python
from flask import Flask

app = Flask(__name__)
```

نحوه تغییر آن به APIFlask:

```python
from apiflask import APIFlask

app = APIFlask(__name__)
```

## کلاس `Blueprint` -> کلاس `APIBlueprint`

نحوه ساخت یک برنامه در فلسک اینگونه است:

```python
from flask import Blueprint

bp = Blueprint('foo', __name__)
```

نحوه تغییر آن به APIFlask:

```python
from apiflask import APIBlueprint

bp = APIBlueprint('foo', __name__)
```

!!! tip

    شما میتوانید از یک آبجکت `Blueprint` در APIFlask استفاده کنید،
    اما نمیتوانید از `APIBlueprint` در فلسک استفاده کنید.

## استفاده از میانبرهای مسیر (اختیاری)

APIFlask برخی از میانبرهای مسیر را فراهم می کند، می توانید یک تابع ویو را به روز کنید:

```python hl_lines="1"
@app.route('/pets', methods=['POST'])
def create_pet():

    return {'message': 'created'}

```

به:

```python hl_lines="1"
@app.post('/pets')
def create_pet():
    return {'message': 'created'}
```

!!! tip

    شما میتوانید میانبر های مسیر را با `app.route()` ترکیب کنید. فلسک ۲.۰ شامل این میانبر ها است.

## ویو های بر اساس کلاس (متودویو)

!!! warning "Version >= 0.5.0"

    این ویژگی در [نسخه 0.5.0](/changelog/#version-050). آمده است.

APIFlask از ویو کلاس های `MethodView` بیس پشتیبانی میکند. برای مثال:

```python
from apiflask import APIFlask, Schema, input, output
from flask.views import MethodView

# ...

class Pet(MethodView):

    decorators = [doc(responses=[404])]

    @app.output(PetOut)
    def get(self, pet_id):
        pass

    @app.output({}, status_code=204)
    def delete(self, pet_id):
        pass

    @app.input(PetIn)
    @app.output(PetOut)
    def put(self, pet_id, data):
        pass

    @app.input(PetIn(partial=True))
    @app.output(PetOut)
    def patch(self, pet_id, data):
        pass

app.add_url_rule('/pets/<int:pet_id>', view_func=Pet.as_view('pet'))
```

کلاس های ویو مبتدی بر `View` پشتیبانی نمیشود ولی همچنان میتوانید از آن استفاده کنید اما در حال حاضر APIFlask نمیتواند مشخصات OpenAPI (و مستندات API) را برای آن ایجاد کند.

## تغییرات رفتاری دیگر و یادداشت ها

### توضیح ایمپورت کردن

شما فقط `APIBlueprint` و `APIFlask` و سایر ابزار های کمکی را از پکیج که توسط `apiflask` ارائه میشود را از ان ایمپورت کنید.
برای سایر آنها، باید از پکیج `flask` استفاده کنید.

```python
from apiflask import APIFlask, APIBlueprint
from flask import request, escape, render_template, g, session, url_for
```

### مقایسه `abort()` در APIFlask و Flask

APIFlask's `abort()` function will return a JSON error response while Flask's `abort()` returns an HTML error page:
تابع `abort()` در APIFlask یک پاسخ خطای JSONی بر بر میگرداند در صورتی که `abort()` در فلسک یک صفحه HTMLی برای ارور بر میگرداند.

```python
from apiflask import APIFlask, abort

app = APIFlask(__name__)

@app.get('/foo')
def foo():
    abort(404)
```

در مثال فوق، زمانی که کاربر `/foo` را بازدید میکند، بدنه پاسخ به این صورت خواهد بود:

```json
{
    "detail": {},
    "message": "Not Found",
    "status_code": 404
}
```

شما میتوانید از پارامتر های `message` و `detail` برای پاس کردن پیغام و دلیل ارور به تابع `abort()` استفاده کنید.

!!! warning
‍    تابع `abort_json()` از [نسخه 0.4.0](/changelog/#version-040) به بعد با نام `abort()` شناخته شده است.

### خطا های JSON و ترکیب استفاده `flask.abort()` و `apiflask.abort()`

زمانی که بیس کلاس را به `APIFlask` تغییر میدهید، تمام پاسخ
های ارور به صورت پیشفرض تبدیل به فرمت JSON میشوند حتی اگر از تابع `abort()` فلسک استفاده کنید:

```python
from apiflask import APIFlask
from flask import abort

app = APIFlask(__name__)

@app.get('/foo')
def foo():
    abort(404)
```

اگر میخواهید این رفتار را غیر فعال کنید، فقط کافی است که پارامتر `json_errors` را `Flase` ست کنید.

```python hl_lines="3"
from apiflask import APIFlask

app = APIFlask(__name__, json_errors=False)

```

اکنون همچنان میتوانید از `abort` از `apiflask` برای بر گرداندن پاسخ های ارور به صورت JSON
 استفاده کنید. برای ترکیب استفاده از `flask.abort` و `apiflask.abort`، باید یکی را به نام دیگری ایمپورت کنید:

```python
from apiflask import abort as abort_json
```

مثال کامل در اینجا آمده است:

```python hl_lines="1 14"
from apiflask import APIFlask, abort as abort_json
from flask import abort

app = APIFlask(__name__, json_errors=False)

@app.get('/html-error')
def foo():

    abort(404)

@app.get('/json-error')
def bar():

    abort_json(404)

```

### مقادیر بازگشتی تابع view

برای یک تابع ویو ساده بدون دکوراتور ``@app.output``، می‌توانید یک دیکشنری یا لیست را به عنوان پاسخ JSON برگردانید. دیکشنری و لیست برگشتی به صورت خودکار به پاسخ JSON تبدیل می شود(با صدا زدن
[`jsonify()`](https://flask.palletsprojects.com/api/#flask.json.jsonify) به صورت داخلی).

!!! tip
    اگر چه لیست به نظر می رسد مثل تاپل فقط مقادیر بازگشتی لیست به پاسخ JSON سریال می شوند.
     تاپل [معنای خاص](https://flask.palletsprojects.com/quickstart/#about-responses) دارد. با شروع از
     فلسک 2.2، مقادیر بازگشتی لیست به صورت بومی پشتیبانی می شوند.

```python
@app.get('/foo')
def foo():
    return {'message': 'foo'}

@app.get('/bar')
def bar():
    return ['foo', 'bar', 'baz']
```

هنگامی که یک دیکوریتور ``@app.output`` را برای تابع ویو خود اضافه کردید، APIFlask از شما انتظار دارد که یک آبجکت مدل ORM/ODM یا یک dict/list که با طرحی که در دیکوریتور ``@app.output`` ارسال کرده اید مطابقت دارد، برگردانید. اگر یک آبجکت ``Response`` را برگردانید، APIFlask آن را مستقیماً بدون هیچ فرآیندی برمی گرداند.

## اقدامات بعدی

اکنون برنامه شما به APIFlask منتقل شده است. برای اطلاعات بیشتر فصل [استفاده پایه](/usage)
را بررسی کنید. لذت ببرید!
