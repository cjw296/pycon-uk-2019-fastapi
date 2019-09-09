.. title: FastAPI from the ground up

:data-transition-duration: 150
:skip-help: true
:css: presentation.css

.. role:: bold

-------------

:id: title
:class: slide

FastAPI from the ground up
====================================


Chris Withers

.. class:: grey

  Jump Trading

.. note::

  Who am I?

--------------

:class: slide

History
-------

.. container:: sp50 tighter

    - Static HTML
    - CGI scripts
    - Zope
    - DRF + React

.. container:: sp50 center

    .. image:: images/zope.png
     :width: 600px

--------------

:class: slide

What are the technologies?
--------------------------

:class: slide

.. container:: sp100

    - REST
.. container:: sp100

    - Async
.. container:: sp100

    - Dependency Injection

--------------

:class: slide

REST
----

.. container:: center

  Representational State Transfer

.. container:: sp100 big

    ::

      GET /entry/?contains=foo

      GET /entry/123

      POST /entry/

      PUT /entry/123

      DELETE /entry/123

--------------

:class: slide

Async
-----

.. container:: center

  Co-operative multitasking

.. container:: sp100

    .. code-block:: python

        async def get_item(name):
            client = httpx.AsyncClient()
            response = await client.get('https://.../'+name)
            return response.json()


.. container:: sp100

    Why should you hate it?

--------------

:class: slide

Dependency Injection
--------------------

.. container:: center

  Say what you want, not how to get it.


.. container:: sp100

    .. code-block:: python

        def render(session: Session, name: str):
            item = Session.query(Item).filter_by(name=name)
            return make_json(item)


.. container:: sp100

    Why should you love it?

--------------

:class: slide

What are the options?
----------------------

.. container:: sp100

    - Django + DRF
    - Flask
    - Pyramid
    - Hug
    - FastAPI

--------------

:class: slide-wide

Django + DRF
------------

.. code-block:: python

    class UserSerializer(serializers.HyperlinkedModelSerializer):
        class Meta:
            model = User
            fields = ['url', 'username', 'email', 'groups']

    class UserViewSet(viewsets.ModelViewSet):
        queryset = User.objects.all().order_by('-date_joined')
        serializer_class = UserSerializer

.. container:: center

    .. image:: images/drf.png
     :width: 400px

.. note::

  - super flexible
  - web UI for free
  - django ORM
  - django settings

--------------

:class: slide-wide

Flask
-----

.. code-block:: python

    from flask import Flask, session, redirect, \
        url_for, escape, request

    app = Flask(__name__)

    @app.route('/login', methods=['GET', 'POST'])
    def login():
        if request.method == 'POST':
            session['username'] = request.form['username']
            return redirect(url_for('index'))
        return '''
            <form method="post">
                <p><input type=text name=username>
                <p><input type=submit value=Login>
            </form>
        '''

.. note::

  - horrible globals
  - wsgi sync
  - no "DRF"?

--------------

:class: slide-wide

Pyramid
-------

.. code-block:: python

    def hello_world(request):
        return Response('Hello World!')

    if __name__ == '__main__':
        with Configurator() as config:
            config.add_route('hello', '/')
            config.add_view(hello_world, route_name='hello')
            app = config.make_wsgi_app()
        server = make_server('0.0.0.0', 6543, app)
        server.serve_forever()


.. note::

  - super flexible
  - very abstracted, but "no DRF" library problem
  - wsgi / sync only
  - (small community?)

--------------

:class: slide

Hug
---

.. code-block:: python

    @hug.get(examples='name=Timothy&age=26')
    @hug.local()
    def happy_birthday(
            name: hug.types.text,
            age: hug.types.number,
            hug_timer=3
    ):
        """Says happy birthday to a user"""
        return {
            'message': f'Happy {age} Birthday {name}!',
            'took': float(hug_timer)
        }

.. note::

  - dependency injection
  - maintainer went awol
  - weird cli stuff?

--------------------

:class: slide

Why FastAPI?
------------

- clean, new
- dependency injection driven
- optionally synchronous
- automatic OpenAPI front end
- great community
- future proof


.. note::

    future proof: (websocket, graphql, etc)

------------------

:class: slide

URL mapping
-----------

.. code-block:: python

    app = FastAPI()

    @app.get("/")
    def root(...):
        return {"greeting": "Hello World"}

    app.include_router(events.router, prefix="/events"...)

.. code-block:: python

    router = APIRouter()

    @router.post("/", ..., status_code=201)
    def create_object(...):
        ...

------------------

:class: slide

From Requests: Path Variables
-----------------------------

.. container:: pre big

    GET /events/:bold:`1234`?detail=full
    Authorization: Token ABCD1234

.. code-block:: python

    @router.get("/event/{id}")
    def get_object(id: int):
        ...

.. code-block:: python

    from fastapi import Path
    from pydantic import Required

    @router.get("events/{id}")
    def get_object(id: int = Path(Required)):
        ...

------------------

:class: slide-wide

From Requests: Query Parameters
-------------------------------

.. container:: pre big

    GET /search/:bold:`?text=something`
    Authorization: Token ABCD1234

.. code-block:: python

    @router.get("/search")
    def search(text:str = None, offset:int = 0, limit:int = 100):
        ...

.. code-block:: python

    from fastapi import Query

    @router.get("/search")
    def search(
        text: str = Query(None),
        offset: int = Query(0),
        limit: int = Query(100),
    ):

------------------

:class: slide

From Requests: Headers
-------------------------------

.. container:: pre big

    GET /search?text=:bold:`something`
    Authorization: Token ABCD1234



{'

Parameters:
- path
- query

- json (multiple, schema nested)

- cookie
- header

- request body
- form (files)

------------------

:class: slide

FastAPI: Putting Stuff in Responses
------------------------------------


------------------


OpenAPI Front End


------------------

Sync vs Async
-------------


----------------

Dependencies

Databases

Authentication

Configuration (configurator?)

What comes next?
- abstract out model helpers
- abstract out Auth

----------

:class: slide

Questions?
==========

  Getting these slides:

  * https://github.com/cjw296/pycon-uk-2019-fastapi

.. container:: sp50

  Fast API Documentation:

  * https://fastapi.tiangolo.com/

.. container:: sp50

  Getting hold of me:

  * chris@python.org
  * cwithers@jumptrading.com
  * @chriswithers13
