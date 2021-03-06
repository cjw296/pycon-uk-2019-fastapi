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
        text: str = Query(default=None, max_length=20),
        offset: int = Query(0),
        limit: int = Query(100),
    ):

------------------

:class: slide

From Requests: Headers
----------------------

.. container:: pre big

    GET /search/?text=something
    :bold:`Authorization: Token ABCD1234`

.. code-block:: python

    from fastapi import Header

    @router.get("/search")
    def search(authorization=Header(default=None)):
        ...

------------------

:class: slide

From Requests: Cookies
----------------------

.. container:: pre big

    GET /search/?text=something
    Authorization: Token ABCD1234
    :bold:`🍪_suggest_author: Chris`

.. code-block:: python

    from fastapi import Cookie

    @router.get("/search/")
    def search(_suggest_author=Cookie(default=None)):
        ...


------------------

:class: slide-wide

From Requests: Forms
--------------------

.. code-block:: html

    <form action="/files/"
          enctype="multipart/form-data" method="post">
    <input name="token" type="text">
    <input name="file" type="file">
    <input type="submit">
    </form>

.. code-block:: python

    from fastapi import File, Form, UploadFile
    from pydantic import Required

    @router.post("/files/")
    async def create_file(
        token: str = Form(Required),
        file: UploadFile = File(Required),
    ):
        ...

------------------

:class: slide

From Requests: JSON Body
------------------------

.. container:: pre big

    POST /events/
    Authorization: Token ABCD1234

    .. container:: bold

        ::

            {
                'date': '2019-06-02',
                'type': 'DONE',
                'text': 'some stuff got done'
            }



------------------

:class: slide

Data Validation: Pydantic
-------------------------

- https://pydantic-docs.helpmanual.io/

.. code-block:: python

    from datetime import date as DateType
    from enum import Enum
    from pydantic import BaseModel

    class Types(Enum):
        done = 'DONE'
        cancelled = 'CANCELLED'

    class Event(BaseModel):
        date: DateType
        type: Types
        text: str

------------------

:class: slide-wide

Data Validation: Pydantic
-------------------------

.. code-block:: python

    class Event(BaseModel):
        date: DateType
        type: Types
        text: str

.. container:: code medium mg20

        >>> Event(date='2019-01-01', type='DONE', text='stuff')
        <Event date=datetime.date(2019, 1, 1)
               type=<Types.done: 'DONE'> text='stuff'>

.. container:: code medium

        >>> Event(date='2019-01-', text='some stuff')
        Traceback (most recent call last):
        ...
        pydantic.error_wrappers.ValidationError: 3 validation errors
        date
          invalid date format (type=type_error.date)
        type
          field required (type=value_error.missing)

------------------

:class: slide

From Requests: JSON Body
------------------------

.. container:: pre big

    POST /events/
    Authorization: Token ABCD1234

    .. container:: bold

        ::

            {
                'date': '2019-06-02',
                'type': 'DONE',
                'text': 'some stuff got done'
            }

.. code-block:: python

    @router.post("/events/")
    def create_object(event: Event = Required):
        ...

------------------

:class: slide-wide

To Responses: Body
------------------

.. code-block:: python

    @app.get("/")
    def root(...):
        return {"greeting": "Hello World"}

.. code-block:: python

    @app.post("/events/", response_model=Event, status_code=201)
    def create_object(event: Event = Required):
        ...
        return {
            'date': '2019-06-02',
            'type': 'DONE',
            'text': 'some stuff got done'
        }

------------------

:class: slide-wide

To Responses: Headers
---------------------

.. code-block:: python

    from starlette.responses import Response

    @app.get("/headers/")
    def get_headers(response: Response):
        response.headers["X-Cat-Dog"] = "alone in the world"
        return {"message": "Hello World"}

.. code-block:: python

    from starlette.responses import JSONResponse

    @app.get("/headers/")
    def get_headers():
        content = {"message": "Hello World"}
        headers = {"X-Cat-Dog": "alone in the world"}
        return JSONResponse(content=content, headers=headers)

------------------

:class: slide-wide

To Responses: Cookies
---------------------

.. code-block:: python

    from starlette.responses import Response

    @app.post("/cookie-and-object/")
    def create_cookie(response: Response):
        response.set_cookie(key="fakesession",
                            value="fake-cookie-session-value")
        return {"message": "Come to the dark side"}

.. code-block:: python

    from starlette.responses import JSONResponse

    @app.post("/cookie/")
    def create_cookie():
        response = JSONResponse(content=...)
        response.set_cookie(key="fakesession",
                            value="fake-cookie-session-value")
        return response

------------------

:class: slide-wide

OpenAPI Front End
------------------

.. container:: sp50 center

    .. image:: images/openapi.png
     :width: 800px


------------------

:class: slide

Sync vs Async
-------------

.. container:: sp100

    .. code-block:: python

        app = FastAPI()

        @app.get("/sync")
        def root():
            return {"greeting": "Hello World"}

        @app.get("/async")
        async def root():
            return {"message": "Hello World"}

----------------

:class: slide

Dependencies
------------

.. container:: sp100

    .. code-block:: python

        from fastapi import Depends, Security

        @router.get("/{id}", response_model=EventFull)
        def get_object(
            id: int,
            session: Session = Depends(db_session),
            current_user: User = Security(get_current_user)
        ):
            ...


.. note::

    - can be sync or async
    - cached once per request

------------------

:class: slide

An application: Diary!
----------------------


.. container:: sp100 big code

   DID travel to pycon UK
   DID speak to Evil Util Co:
   --
   Have promised it will be fixed tomorrow
   --
   CANCELLED go to gym

------------------

:class: slide-wide

Databases
---------

.. code-block:: python

    from enum import Enum
    from sqlalchemy import Column, Integer, Text, Date
    from sqlalchemy.dialects.postgresql import ENUM
    from sqlalchemy.ext.declarative import declarative_base

    Base = declarative_base()

    class Types(Enum):
        done = 'DONE'
        cancelled = 'CANCELLED'

    class Event(Base):
        __tablename__ = 'entry'
        id = Column(Integer(), primary_key=True)
        date = Column(Date, nullable=False)
        type = Column(ENUM(Types, name='types_enum'))
        text = Column(Text, nullable=False)

.. note::

  - SQLAlchemy
  - Where does the session come from?
  - Where do the credentials for that session come from?

------------------

:class: slide-wide

Configuration
-------------

.. code-block:: python

    from configurator import Config
    from pydantic import BaseModel, DSN

    # schema
    class DatabaseConfig(BaseModel):
        url: DSN

    class AppConfig(BaseModel):
        testing: bool
        db: DatabaseConfig

    # defaults
    config = Config({
        'testing': False,
    })

------------------

:class: slide-wide

Configuration
-------------

.. code-block:: python

    def load_config(path=None):
        if config.testing:
            return
        # file
        if path is None:
            path = Path(__file__).resolve().parent / 'app.yml'
        config.merge(
            Config.from_path(path)
        )
        # env
        config.merge(os.environ, {
            'DB_URL': 'db.url',
        })
        # validate
        AppConfig(**config.data)
        return config

.. note::

  Pathlib! :-)

------------------

:class: slide-wide

Configuration
-------------

.. container:: sp100

    .. code-block:: python

        from fastapi import FastAPI
        from sqlalchemy import create_engine

        app = FastAPI()

        @app.on_event("startup")
        def configure():
            load_config()
            Session.configure(bind=create_engine(config.db.url))

------------------

:class: slide-wide

Databases Again
---------------

.. container:: sp50

    .. code-block:: python

        @app.get("/")
        def root(session: Session = Depends(db_session)):
            return {"count": session.query(Event).count()}

.. container:: sp50

    .. code-block:: python

        from sqlalchemy.orm import sessionmaker

        Session = sessionmaker()

        def db_session(request: Request):
            return request.state.db


------------------

:class: slide-wide

Databases Again
---------------

.. container:: sp50

    .. code-block:: python

        from starlette.requests import Request
        from starlette.concurrency import run_in_threadpool

        def finish_session(session):
            session.rollback()
            session.close()

        @app.middleware('http')
        async def make_db_session(request: Request, call_next):
            request.state.db = session = Session()
            response = await call_next(request)
            await run_in_threadpool(finish_session, session)
            return response

------------------

:class: slide

CRUD: Schemas
-------------

.. code-block:: python

    class EventNonPrimaryKey(BaseModel):
        date: DateType
        type: Types
        text: str

    class EventFull(EventNonPrimaryKey):
        id: int

    class EventList(BaseModel):
        items: List[EventFull]
        count: int
        prev: str = None
        next: str = None

------------------

:class: slide-wide

CRUD: Create
-------------

.. container:: sp50

    .. code-block:: python

        @router.post("/", response_model=EventFull, status_code=201)
        def create_object(
            event: EventNonPrimaryKey = Required,
            session: Session = Depends(db_session),
        ):
            """
            Create new Event.
            """
            with session.transaction:
                obj = Event(**event.dict())
                session.add(obj)
                session.flush()
                return simplify(obj)


.. note::

  explain simplify and why the code isn't there

------------------

:class: slide-wide

CRUD: Read
----------

.. code-block:: python

    @router.get("/{id}", response_model=EventFull)
    def get_object(
        id: int,
        session: Session = Depends(db_session),
    ):
        """
        Get Event by ID.
        """
        with session.transaction:
            try:
                obj = session.query(Event).filter_by(id=id).one()
            except NoResultFound:
                raise HTTPException(status_code=404)
            else:
                return simplify(obj)

------------------

:class: slide-wide

CRUD: List
----------

.. container:: small

    .. code-block:: python

        @router.get("/", response_model=EventList, name='events_list')
        def list_object(
            text: str = None, offset: int = Query(0), limit: int = 100,
            session: Session = Depends(db_session),
            request: Request = Required,
        ):
            with session.transaction:
                items = session.query(Event).order_by(Event.date.desc(), 'id')
                if text:
                    items = items.filter(Event.text.ilike('%'+text.strip()+'%'))
                count = items.count()
                items = [EventFull(**simplify(i))
                         for i in items.offset(offset).limit(limit)]
                if len(items) != count:
                        ...
                        prev = url_for(request, ...)
                        next = url_for(request, ...)
                else:
                    prev = next = None
                return EventList(count=count, items=items, prev=prev, next=next)

.. notes

    request.url_for(name)+'?'+urlencode({'limit': limit, 'offset': offset})

------------------

:class: slide-wide

CRUD: Update
------------

.. code-block:: python

    @router.put("/{id}", response_model=EventFull)
    def update_object(
        id: int,
        event: EventNonPrimaryKey = Required,
        session: Session = Depends(db_session),
    ):
        with session.transaction:
            try:
                obj = session.query(Event).filter_by(id=id).one()
            except NoResultFound:
                raise HTTPException(status_code=404)
            else:
                for key, value in event.dict().items():
                    setattr(obj, key, value)
                return simplify(obj)

------------------

:class: slide-wide

CRUD: Delete
------------

.. code-block:: python

    @router.delete("/{id}", response_model=EventFull)
    def delete_object(
        id: int,
        session: Session = Depends(db_session),
    ):
        """
        Delete an Event.
        """
        with session.transaction:
            try:
                obj = session.query(Event).filter_by(id=id).one()
            except NoResultFound:
                raise HTTPException(status_code=404)
            else:
                session.delete(obj)
                return simplify(obj)

------------------

:class: slide-wide

Authentication
--------------

.. code-block:: python

    from fastapi import Depends, Security
    from fastapi.security import HTTPBasic, HTTPBasicCredentials

    security = HTTPBasic()

    def get_current_user(
            credentials: HTTPBasicCredentials = Depends(security)
    ):
        return credentials.username

    @router.get("/{id}", response_model=EventFull)
    def get_object(
        id: int,
        session: Session = Depends(db_session),
        current_user: User = Security(get_current_user)
    ):
        ...

.. note::

  Supports OAuth2, all the other goodness
  Obviously need to actually check passwords!

------------------

:class: slide

Testing: The fixtures
---------------------

.. code-block:: python

    @pytest.fixture(scope='session')
    def client():
        with config.push({
            'testing': True,
            'db': {'url': os.environ['TEST_DB_URL']}
        }):
            with TestClient(app) as client:
                yield client

------------------

:class: slide

Testing: The fixtures
---------------------

.. code-block:: python

    @pytest.fixture(scope='session')
    def db(client):
        engine = Session.kw['bind']
        conn = engine.connect()
        transaction = conn.begin()
        try:
            Base.metadata.create_all(bind=conn,
                                     checkfirst=False)
            yield conn
        finally:
            transaction.rollback()
            Session.configure(bind=engine)

------------------

:class: slide

Testing: The fixtures
---------------------

.. code-block:: python

    @pytest.fixture()
    def session(db):
        transaction = db.begin_nested()
        try:
            Session.configure(bind=db)
            yield Session()
        finally:
            transaction.rollback()

------------------

:class: slide

Testing: The test
-----------------

.. code-block:: python

    def test_create_full_data(session, client):
        response = client.post('/events/', json={
            'date': '2019-06-02',
            'type': 'DONE',
            'text': 'some stuff got done'
        })
        actual = session.query(Event).one()
        compare(actual.date, expected=date(2019, 6, 2))
        compare(response.json(), expected={
            'id': actual.id,
            'date': '2019-06-02',
            'type': 'DONE',
            'text': 'some stuff got done'
        })
        compare(response.status_code, expected=201)

.. note::

  remember to explain testfixtures!

----------------

:class: slide

What's still left to do?
------------------------

- Abstract out model helpers
- Abstract out auth library
- Support context manager dependencies

----------

:class: slide

Questions?
==========

  Getting the talk materials:

  * https://cjw296.github.io/pycon-uk-2019-fastapi/
  * https://github.com/cjw296/diary/tree/master/backend

.. container:: sp50

  Fast API Documentation:

  * https://fastapi.tiangolo.com/

.. container:: sp50

  Getting hold of me:

  * chris@python.org / @chriswithers13
