---
layout: post
title: Django REST API with React App step-by-step
---

*I use PyCharm for python projects and pipenv for package management.*

Start new PyCharm project named *django-rest-react-blog*

*Optionally*

```
git init
```

clone gist [.gitignore](https://gist.github.com/Vostbur/baabad0ed80ead1b5155fbbce0742591) for Django and React projects

*Main course*

```
pipenv install django
pipenv install djangorestframework
pipenv install ipython --dev
django-admin.py startproject bookcase
cd bookcase
python manage.py startapp mainapp
```

Edit bookcase/settings.py:
--------------------------

```
...
INSTALLED_APPS = [
...
    'rest_framework'
]
...
TEMPLATES = [
    {
	...
        'DIRS': [BASE_DIR / 'mainapp-ui/build'],
	...
    },
]
...
STATIC_ROOT = BASE_DIR / 'static'

STATICFILES_DIRS = (
    (BASE_DIR / 'mainapp-ui/build/static'),
)
```

Edit mainapp/views.py:
----------------------

```
from django.shortcuts import render


def index(request):
    return render(request, 'index.html', {})
```

Edit bookcase/urls.py:
----------------------

```
...
from mainapp.views import index


urlpatterns = [
    ...
    path('', index)
]
```

make dir **mainapp/api** with files there:

```
ls api
__init__.py  selrializers.py  urls.py  views.py
```

Edit mainapp/api/views.py
--------------------

```
from rest_framework.views import APIView
from rest_framework.response import Response


class TestAPIView(APIView):

    def get(self, request, *args, **kwargs):
        data = [{"id": 1, "name": "Alexey"},
                {"id": 2, "name": "Ivan"}]
        return Response(data)
```

Edit mainapp/api/urls.py
-------------------

```
from django.urls import path

from .views import TestAPIView


urlpatterns = [
    path('test-api/', TestAPIView.as_view(), name='test')
]
```

Change bookcase/urls.py
-----------------------
```
...
from django.urls import path, include

from mainapp.views import index


urlpatterns = [
    ...
    path('api/', include('mainapp.api.urls'))
]
```

*Start React JS App*

```
npx create-react-app mainapp-ui
cd mainapp-ui
yarn add axios
```

Replace mainapp-ui/src/App.js with
---------------------
```
import { useState, useEffect } from 'react';
import axios from 'axios';
import './App.css';

function App() {

  const [people, setPeople] = useState([])

  useEffect(() => {
    axios({
        method: "GET",
        url: "http://127.0.0.1:8000/api/test-api/"
    }).then(response => {
      setPeople(response.data)
    })
  }, [])

  return (
    <div className="App">
      <ul>
        {people.map(p => (
            <li key={p.id}>{p.name}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

*Build and run App*

```
yarn run build
cd ..
python manage.py collectstatic
python manage.py migrate
python manage.py runserver
```

Open http://127.0.0.1:8000
