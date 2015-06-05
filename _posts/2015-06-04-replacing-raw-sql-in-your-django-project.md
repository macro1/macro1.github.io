---
layout: post
---

If you work on an old Django project you probably have them, lurking in dark
corners of your code. Small strings that make what would otherwise be
impossible with Django's ORM possible with a little SQL magic. Raw SQL. By the
time it's been added to the project, the cost has been weighed and accepted.

There is another way though. I discovered recently that individual clauses or
expressions in SQLAlchemy can be compiled piecemeal, allowing them to be
passed to the Django ORM in place of raw SQL snippets. The first example from
the [SQLAlchemy docs][concat example] that really intrigued me about this was
'CONCAT', which MySQL implements as a function:


    >>> print (users.c.name + users.c.fullname).\
    ...      compile(bind=create_engine('mysql://'))
    concat(users.name, users.fullname)


Those of you familiar with other engines already know the more standard strict
concat operation is done with pipes ('||'), but SQLAlchemy makes this change
for us if we bind the correct engine when compiling.

So, SQLAlchemy can express queries, and expressions within queries, and it can
be compiled to any major dialect... We now need a comfortable way to fit all
of that into writing a '.extra()'.

Here is an example clause compiling helper using [django-sabridge]:

alchemy.py:


    from django.db import connection
	import sabridge
	import sqlalchemy
	
	bridge = sabridge.Bridge()
	
	
	def compile(epxression):
		engine = sqlalchemy.create_engine('{}://'.format(connection.vendor))
	    return str(expression.compile(bind=engine))
		

That allows us to use sqlalchemy instead of raw SQL without adding too much extra to our code.
		
[concat example]: http://docs.sqlalchemy.org/en/rel_1_0/core/tutorial.html#operators "SQLAlchemy Tutorial: Operators"
[django-sabridge]: http://django-sabridge.readthedocs.org/
