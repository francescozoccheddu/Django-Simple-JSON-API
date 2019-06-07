# Django Simple JSON API
## Install
`pip install django-simple-json-api`
## Instructions
Add `dsjapi.decorators.api` or `dsjapi.decorators.api_rq` to your view function.  
Return a JSON serializable object or a `django.http.HttpResponse` object.  
Raising a `dsjapi.exceptions.ServerException` exception will automatically return the appropriate JSON response.
### Decorator parameters
- **`field`**  
    `dsjapi.fields.Field`  
    Expected field in request body. Default is `None`.
- **`allowedMethods`**  
    `list` of `str`  
    Allowed HTTP methods (uppercase). Default is `["GET", "POST"]`.
- **`safeEncoding`**  
    `bool`  
    `True` to allow non-`dict` response objects. Default is `False`.
 ### `@api` vs `@api_rq`
- `@api` only provides the validated request data.  
- `@api_rq` provides a `dsjapi.decorators.Request` object that contains both `request` (the original `django.http.HttpRequest` object) and `data` (the validated request data) attributes.

## Examples
### Views
```python
from dsjapi.decorators import api
from dsjapi.fields import *

@api (DictField ({
    "name": StringField (minLength=2, maxLength=32, regex=r'^[a-zA-Z ]+$'),
    "bio": StringField (maxLength=64, missing=Field.Do.DEFAULT, default=""), # Optional (default to "")
    "age": IntField (min=18, max=130)
}))
def index (data):
    result = putUser (data["name"], data["bio"], data["age"])
    return { "result": result }
```
```python
from dsjapi.decorators import api
from dsjapi.fields import *

@api (DictField ({
    "start": ListField.byLength (2, FloatField ()),
    "end": ListField.byLength (2, FloatField (), missing=Field.Do.SKIP), # Optional
    "time": FloatField (min=120, max=60*60)
}))
def index (data):
    if "end" in data:
        route = getARoute (data["start"], data["time"])
    else:
        route = getABRoute (data["start"], data["end"], data["time"])
    return { "route": route }
```
```python
from dsjapi.decorators import api
from dsjapi.fields import *

@api (DictField ({
    "numbers": ListField.byLength (2, FloatField (min=0))
}))
def index (data):
    sum = 0
    for number in data["numbers"]:
        sum += number
    avg = sum / len (data["numbers"])
    return { "average": avg }
```
```python
from dsjapi.decorators import api
from dsjapi.fields import *
from dsjapi.exceptions import *

@api (DictField ({
    "id": IntField (min=0, max=4096),
    "password": StringField (min=8, max=32)
}))
def index (data):
    user = getUserById (data["id"])
    if user is None:
        raise NotFoundException ("User not found")
    if user.getPassword () != data["password"]:
        raise AuthenticationException ("Wrong password")
    return { "id": user.getId (), "score": user.getScore () }
```
### Custom fields
```python
from dsjapi.fields import IntField, FieldException

class EvenNumberField (IntField):

    def __init__ (self, min=None, max=None, missing=Field.Do.RAISE, error=Field.Do.RAISE, default=None):
        super ().__init__ (int, min=min, max=max, missing=missing, error=error, default=default)

    def clean (self, value):
        value = super ().clean (value)
        if value % 2 != 0:
            raise FieldException ("Odd number")
        return value
```
