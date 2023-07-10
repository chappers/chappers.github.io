---
layout: post
category: web micro log
tags:
---

Here are some random notes for myself based on my experience in GraphQL/Graphene.

Graphene has two main parts:

- Queries: which are just for querying data
- Mutation: when you have a query which modifies data in some way

# Mutation

To write a mutation there are a few components:

- Structure of the output
- Accepting input parameters
- Modifying data

Sample mutation ([taken from docs](http://docs.graphene-python.org/en/latest/types/mutations/)):

```py
import graphene

class Person(graphene.ObjectType):
    # this is the object which is called in the line
    # ``
    name = graphene.String()
    age = graphene.Int()

class CreatePerson(graphene.Mutation):
    class Input:
        # this mutation takes one parameter "name"
        name = graphene.String()

    # structure of the output is
    # object called "ok"
    # object called person which is a field (sim. to a dictionary)
    ok = graphene.Boolean()
    person = graphene.Field(lambda: Person)

    @staticmethod
    def mutate(root, args, context, info):
        # this extracts the argument when the mutation is called
        person = Person(name=args.get('name'))

        # modify stuff here
        ok = True

        # this is the output of the query after mutation is complete
        return CreatePerson(person=person, ok=ok)
```

Once this is constructed, we only need to specify precisely the mutation and query for this schema defined above

```py
class MyMutations(graphene.ObjectType):
    # this defines the specific mutation function
    # to be called in the function
    # i.e. "create_person" will turn into: "createPerson"
    # in the query
    create_person = CreatePerson.Field()

# We must define a query for our schema
class Query(graphene.ObjectType):
    person = graphene.Field(Person)
```
