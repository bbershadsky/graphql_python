# SQLAlchemy + Python using Graphene

A [SQLAlchemy](http://www.sqlalchemy.org/) integration for [Graphene](http://graphene-python.org/).

## Installation

For instaling graphene, just run this command in your shell

```bash
pip install "graphene-sqlalchemy>=2.0"
```

## Examples

Here is a simple SQLAlchemy model:

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import relationship

from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class UserModel(Base):
    __tablename__ = 'department'
    id = Column(Integer, primary_key=True)
    name = Column(String)
    last_name = Column(String)
```

To create a GraphQL schema for it you simply have to write the following:

```python
from graphene_sqlalchemy import SQLAlchemyObjectType

class User(SQLAlchemyObjectType):
    class Meta:
        model = UserModel
        # only return specified fields
        only_fields = ("name",)
        # exclude specified fields
        exclude_fields = ("last_name",)

class Query(graphene.ObjectType):
    users = graphene.List(User)

    def resolve_users(self, info):
        query = User.get_query(info)  # SQLAlchemy query
        return query.all()

schema = graphene.Schema(query=Query)
```

Then you can simply query the schema:

```python
query = '''
    query {
      users {
        name,
        lastName
      }
    }
'''
result = schema.execute(query, context_value={'session': db_session})
```

You may also subclass SQLAlchemyObjectType by providing `abstract = True` in
your subclasses Meta:
```python
from graphene_sqlalchemy import SQLAlchemyObjectType

class ActiveSQLAlchemyObjectType(SQLAlchemyObjectType):
    class Meta:
        abstract = True

    @classmethod
    def get_node(cls, info, id):
        return cls.get_query(info).filter(
            and_(cls._meta.model.deleted_at==None,
                 cls._meta.model.id==id)
            ).first()

class User(ActiveSQLAlchemyObjectType):
    class Meta:
        model = UserModel

class Query(graphene.ObjectType):
    users = graphene.List(User)

    def resolve_users(self, info):
        query = User.get_query(info)  # SQLAlchemy query
        return query.all()

schema = graphene.Schema(query=Query)
```

### Full Examples

To learn more check out the following [examples](examples/):

- [Flask SQLAlchemy example](examples/flask_sqlalchemy)
- [Nameko SQLAlchemy example](examples/nameko_sqlalchemy)

## Contributing

After cloning this repo, ensure dependencies are installed by running:

```sh
python setup.py install
```

After developing, the full test suite can be evaluated by running:

```sh
python setup.py test # Use --pytest-args="-v -s" for verbose mode
```
