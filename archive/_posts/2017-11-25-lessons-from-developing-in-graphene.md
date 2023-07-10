---
layout: post
category:
tags:
tagline:
---

_For the sake of my own sanity, everything I developed as in Graphene, so whenever I mention GraphQL please substitute with "GraphQL/Graphene" where it makes sense_

GraphQL is an interesting approach to graph queries (after all it stands for Graph Query Language), as it does not explicitly sit on a graph database. Rather it seems to make use of various [data loaders](http://docs.graphene-python.org/en/latest/execution/dataloader/) and constructors in order to create a graph-like experience. In this fashion GraphQL can feel like a virtual database; but for graph outputs.

That being said, it isn't always obvious how to design a simple API for it, but it becomes easier if we stick to a few key principals GraphQL appears to adhere to:

> Key-values in everything!

Do you need to write a `DataLoader`? Please think in terms of key-value pair. Do you need to execute a `query`? Key-value pair.

The relevant aspects to this is that the `key` can be an abstract type, such as a `String` or a `Float`, a `[String]` or even another object. Similar for the `value`. This makes it really powerful and at the same time confusing if that construct wasn't in your mind.

For example, consider [Getting Started](http://docs.graphene-python.org/en/latest/quickstart/) page on graphene.

```py
import graphene

class Query(graphene.ObjectType):
    hello = graphene.String(name=graphene.String(default_value="stranger"))

    def resolve_hello(self, info, name):
        return 'Hello ' + name

schema = graphene.Schema(query=Query)

result = schema.execute('{ hello }')
print(result.data['hello']) # "Hello stranger"
```

If I wanted to change it to, `return firstname + lastname` would I do:

```py
import graphene

class Query(graphene.ObjectType):
    hello = graphene.String(firstname=graphene.String(default_value="stranger"), lastname=graphene.String(default_value="doe"))

    def resolve_hello(self, info, firstname, lastname):
        return 'Hello ' + firstname + " " + lastname

schema = graphene.Schema(query=Query)
result = schema.execute('{ hello }')
print(result.data['hello'])
```

It would appear to work! But how would I substitute variables into it? One approach might look like this:

```py
import graphene

class HelloInfo(graphene.ObjectType):
    hello = graphene.String()

class Query(graphene.ObjectType):
    hello = graphene.Field(HelloInfo, firstname=graphene.String(default_value="stranger"))

    def resolve_hello(self, info, firstname, lastname="doe"):
        return HelloInfo('Hello ' + firstname + " " + lastname)


schema = graphene.Schema(Query)
result = schema.execute(
    '''query hello($firstname: String) {
        hello(firstname: $firstname) {
            hello
        }
    }''',
    variable_values={'firstname': "John"},
)
if result.errors:
    print(result.errors)
print(result.data['hello'])
```

How about adding back the `lastname` variable?

```py
import graphene

class HelloInfo(graphene.ObjectType):
    hello = graphene.String()

class Query(graphene.ObjectType):
    hello = graphene.Field(HelloInfo, firstname=graphene.String(default_value="stranger"), lastname=graphene.String(default_value="doe"))

    def resolve_hello(self, info, firstname, lastname):
        return HelloInfo('Hello ' + firstname + " " + lastname)


schema = graphene.Schema(Query)
result = schema.execute(
    '''query hello($firstname: String) {
        hello(firstname: $firstname) {
            hello
        }
    }''',
    variable_values={'firstname': "John", 'lastname': "strange"},
)
if result.errors:
    print(result.errors)
print(result.data['hello']) # 'Hello John doe'???
```

It doesn't return the provided lastname variable item, nor does it return an error! What happend here?

It turns out we have the _wrong_ understanding of what the arguments do, instead we should write it up as follows:

```py
import graphene

class HelloInfo(graphene.ObjectType):
    hello = graphene.String()

class Query(graphene.ObjectType):
    hello = graphene.Field(HelloInfo, name=graphene.List(graphene.String))

    def resolve_hello(self, info, name):
        full_name = name[0] + " " + name[1]
        return HelloInfo('Hello ' + full_name)


schema = graphene.Schema(Query)
result = schema.execute(
    '''query hello($name: [String]) {
        hello(name: $name) {
            hello
        }
    }''',
    variable_values={'name': ["John", "Strange"]},
)
if result.errors:
    print(result.errors)
print(result.data['hello'])
```

[Which is covered here](http://graphql.org/learn/schema/). Similarly for [DataLoader](http://docs.graphene-python.org/en/latest/execution/dataloader/), the idea applies. When you write the `batch_load_fn` the `keys` can be an item like `[String]`, and the resolution can be an arbitary object. In this way it enforces particular typing so that you can built complex queries.
