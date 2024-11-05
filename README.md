# CSV JSON and SQL alchemy

### csv and json

csv
Typical and easy way to represent tabular data.
Use it with import csv .
Read from files with a reader (or DictReader ).
Write to files with a writer (or DictWriter ).

    import csv
    with open("file.csv", encoding="utf-8") as fp:
    reader = csv.reader(fp) # or DictReader
    for row in reader:
    print(row)

to write csv files use writer and write lists/tuples
better! use DictWriter

- requires fields to be provided as a list

- writes header row with .writeheader()

- takes dicts and write rows

```
    example = {"name": "Tim", "grade": 25}

    fields = list(example.keys())

    with open("output.csv", "w", encoding="utf-8", newline="") as fp:
        writer = DictWriter(fp, fields=fields)
        writer.writeheader()
        writer.writerow(example)

```

```python

# Import necessary modules for scraping
import csv
from urllib.parse import urljoin

import httpx
from bs4 import BeautifulSoup

from book import Book  # Import the Book class from the book module

def scrape(url="https://books.toscrape.com/index.html"):
    # An array to hold all the book objects
    books = []

    # The CSV file we write to
    out = open("books.csv", "w", newline="")
    # Create a CSV writer object with the field names from the Book class
    writer = csv.DictWriter(out, fieldnames=Book.FIELDS)
    writer.writeheader()  # Write the header to the CSV file

    keep_going = True  # Flag to control the loop
    while keep_going:
        # Main loop for scraping each page of books
        print("--- INDEX PAGE ---", url)
        
        response = httpx.get(url)  # Fetch the HTML content of the page
        
        soup = BeautifulSoup(response.text, "html.parser")  # Parse the HTML
        # Find book links and create book objects
        for book in soup.select(".product_pod h3 a"):            
            book_url = urljoin(url, book.attrs["href"])  # Construct the full book URL
            book_obj = Book(book_url)  # Create a Book object for the book URL
            # Add the book object to the list
            books.append(book_obj)
            # Write the book object data to the CSV file
            writer.writerow(book_obj.to_dict())
            
        # Find the next page link
        next_link = soup.select("li.next a")
        
        if not next_link or len(next_link) == 0:
            # No link found - we stop the loop
            keep_going = False  # Set the flag to stop the loop
        else:
            # Update the URL with the next page link and continue the loop
            url = urljoin(url, next_link[0].attrs["href"])
    
    out.close()  # Close the CSV file after writing all data
    return books  # Return the list of book objects


if __name__ == "__main__":
    books = scrape()  # Call the scrape function when the script is executed


```




### json
JSON: JavaScript Object Notation

One of the many ways to represent data

Similar to the way JavaScript represents data internally

Very heavily used for communications between clients and servers using ReSTful APIs

Pros

- Lightweight and very flexible

- Language independent

- Easy to read and write (even by humans)

- Flat (text format)

Cons

- Very flexible – no validation

- No comments

- Hard to "debug

JSON data types

Key / value pairs

Keys are strings

Values can be:

- strings

- numbers (integer, floating)

- objects (= another JSON block)

- arrays

- special (booleans, null)

```
    { # object dict
        "name": "Alice", #string
        "age": 30, #number int/float
        "city": "New York",
        "skills": [ # array - list
            "Python",
            "Data Analysis",
            "Machine Learning"
        ]
    }
```

### json in python

```py

import json
with open("input.json") as fp:
json.load(fp)
value = """["JSON list", {"keyword": "value"}]"""
data = json.loads(value)
with open("output.json", "w") as fp:
json.dump(data, fp)
print(json.dumps(data))

```


python json mapping

dict - object

list,tuple - array

str - string

int,long,float - number

boolean - boolean

None - null

### Using JSON to represent class instances
- There is no built in way to serialize or deserialize Python classes
- Use the built-in conversion for the Python primitive types
- Serialization: create a to_json method that returns a JSON string
    - converts the state into a Python dict or list, serialize to JSON

Create a to_dict method that return a dictionary
- Deserialization: convert from JSON
    - Create a class method from_json method that takes in a JSON string
    - Deserialize the string to a dict or list
    - Create an instance, and set the instance attributes to the values from the dict / list

### Definitions:
Serialization: creating JSON from data

Deserialization: creating data from JSON

The json library supports the serialization / deserialization of several Python native data types.


### SQL ALCHEMY

- SQL alchemy is a module that make it easy to interact with sql databases

- install using
    - pip install sqlalchemy

- has ORM capabilities (Object Relational Mapping)

- only use SQL alchemy 2.0

#### SETUP

    from sqlalchemy import create_engine
    engine = create_engine("sqlite:///bing.db", echo=True)

set echo to False to stop seeing translated queries in terminal

#### making a session class

    #file:database.py

    from sqlalchemy.orm import sessionmaker
    Session = sessionmaker(bind=engine)

#### always use a session

    from database import Session

    session = Session()
    #do stuff

    OR

    with Session() as session
    do stuff

#### create base class

    from sqlalchemy.orm import DeclarativeBase
    class Base(DeclaritiveBase):
    pass

#### database backed class

    from sqlalchemy import String, DECIMAL, Integer, ForeignKey

    class Book(Base):
        __tablename__ = "books"
        id = mapped_column(Integer, primary_key=True)
        title = mapped_column(String)
        price = mapped_column(DECIMAL(10, 2))
        available = mapped_column(Integer, default=0)

#### create the tables

    Base.metadata.create_all()

### use the session to make queries (execute statements)

1. select
2. execute session.execute
3. get the results .scalars() or .scalar()
-----

    from sqlalchemy import select

    statement = select(Book)

    results = session.execute(statement)

    books = results.scalars()

----

add rows

----

    book = Book(title="ACIT2515", available=10, price=22.95)
    session.add(book)
    book = Book(title="Other book", available=1, price=12.95)
    session.commit()

---

delete rows

---

    # Remove book with ID = 10
    book = session.execute(select(Book).where(Book.id == 10)).scalar()
    session.delete(book)
    session.commit()

---

### statements

SQL like methods with python syntax

    select(Book).where(Book.rating <=1)
    select(Book).where(Book.rating >= 3).where(Book.title.ilike("%last%"))


    select(Book).where(Book.available > 0).order_by(Book.rating)

    select(Book).order_by(Book.rating.desc())

#### functions

backend functions can be useful i.e Random

    from sqlalchemy.sql import func
    # Random sort using SQL RANDOM function
    select(Book).where(Book.available > 0).order_by(func.random())

### using results

after executing a statement you can extract the results:

- .scalars() returns a list of result objects

- .scalar() returns one result

- .first() returns the first object or None

### Relationships

#### one to many

you must define a ForeignKey column

the ForeignKey links to the primary_key of other tables

use relationship fields to automatically fetch the related objects

make sure you know how back_populates works

one to many:python code

```py
class Book(Base):
    # [...] other fields
    category_id = mapped_column(Integer, ForeignKey("categories.id"))
    category = relationship("Category", back_populates="books")

class Category(Base):
    __tablename__ = "categories"

    id = mapped_column(Integer, primary_key=True)
    name = mapped_column(String)
    books = relationship("Book", back_populates="category")

```

one to many: using relationship attributes

```python
book = session.execute(select(Book).order_by("?")).scalar()
# Relationship from Book to Category (one)
print(book.category.name)

poetry = session.execute(select(Category).where(name="Poetry")).scalar()
# Relationship from Category to Book(s) (many)
for book in poetry.books:
    print(book.title)
```

### back_populates –

    Indicates the name of a relationship() on the related class that will be 

    synchronized with this one. It is usually expected that the relationship() on 
    
    the related class also refer to this one. This allows objects on both sides of 
    
    each relationship() to synchronize in-Python state changes and also provides 
    
    directives to the unit of work flush process how changes along these 
    
    relationships should be persisted.

### many to many

use an association object with two foreign keys to each class you want to link

carefully name and use relationship fields to set up associations

##### tim example

    class User(Base):
        __tablename__ = "users"

        id = mapped_column(Integer, primary_key=True)
        name = mapped_column(String)
        rentals = relationship("BookRental", back_populates="user")

    class BookRental(Base):
        __tablename__ = "book_rentals"

        id = mapped_column(Integer, primary_key=True)
        user_id = mapped_column(Integer, ForeignKey("users.id"))
        book_id = mapped_column(Integer, ForeignKey("books.id"))
        rented_on = mapped_column(DateTime(timezone=True), nullable=False)
        returned_on = mapped_column(DateTime(timezone=True), nullable=True)
        user = relationship("User", back_populates="rentals")
        book = relationship("Book", back_populates="rentals"

    # create bookrental instance

    from datetime import datetime, timedelta
    tim = session.execute(select(User).where(name == "Tim")).scalar()
    book = session.execute(select(Book).where(title == "ACIT2515")).scalar()
    another_book = session.execute(select(Book).where(Book.id == 10)).scalar()
    
    rental1 = BookRental(user=tim, book=book, rented_on=datetime.now())
    
    yesterday = datetime.now() - timedelta(days=1)
    rental2 = BookRental(user=time, book=another_book, rented_on=yesterday)
   
    session.add(rental1)
    session.add(rental2)
    session.commit()

    # find unreturned books for user tim

    stmt = select(BookRental).where(BookRental.user == tim).where(BookRental.returned_on != None)
    books_not_returned = [result.book.title for result in session.execute(stmt).scalars()]

    # above but pythonized

    books_not_returned = []
    for rental in tim.rentals:
        if rental.returned is None:
            books_not_returned.append(rental.book)
    books_not_returned = [rental for rental in tim.rentals if rental.returned_on is None]

```py
from typing import Optional

from sqlalchemy import ForeignKey
from sqlalchemy import Integer
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy.orm import relationship


class Base(DeclarativeBase):
    pass


class Association(Base):
    __tablename__ = "association_table"
    left_id: Mapped[int] = mapped_column(ForeignKey("left_table.id"), primary_key=True)
    right_id: Mapped[int] = mapped_column(
        ForeignKey("right_table.id"), primary_key=True
    )
    extra_data: Mapped[Optional[str]]
    child: Mapped["Child"] = relationship()


class Parent(Base):
    __tablename__ = "left_table"
    id: Mapped[int] = mapped_column(primary_key=True)
    children: Mapped[List["Association"]] = relationship()


class Child(Base):
    __tablename__ = "right_table"
    id: Mapped[int] = mapped_column(primary_key=True)

```


```py
# create parent, append a child via association
p = Parent()
a = Association(extra_data="some data")
a.child = Child()
p.children.append(a)

# iterate through child objects via association, including association
# attributes
for assoc in p.children:
    print(assoc.extra_data)
    print(assoc.child)
```


```py
from typing import Optional

from sqlalchemy import ForeignKey
from sqlalchemy import Integer
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy.orm import relationship


class Base(DeclarativeBase):
    pass


class Association(Base):
    __tablename__ = "association_table"

    left_id: Mapped[int] = mapped_column(ForeignKey("left_table.id"), primary_key=True)
    right_id: Mapped[int] = mapped_column(
        ForeignKey("right_table.id"), primary_key=True
    )
    extra_data: Mapped[Optional[str]]

    # association between Assocation -> Child
    child: Mapped["Child"] = relationship(back_populates="parent_associations")

    # association between Assocation -> Parent
    parent: Mapped["Parent"] = relationship(back_populates="child_associations")


class Parent(Base):
    __tablename__ = "left_table"

    id: Mapped[int] = mapped_column(primary_key=True)

    # many-to-many relationship to Child, bypassing the `Association` class
    children: Mapped[List["Child"]] = relationship(
        secondary="association_table", back_populates="parents"
    )

    # association between Parent -> Association -> Child
    child_associations: Mapped[List["Association"]] = relationship(
        back_populates="parent"
    )


class Child(Base):
    __tablename__ = "right_table"

    id: Mapped[int] = mapped_column(primary_key=True)

    # many-to-many relationship to Parent, bypassing the `Association` class
    parents: Mapped[List["Parent"]] = relationship(
        secondary="association_table", back_populates="children"
    )

    # association between Child -> Association -> Parent
    parent_associations: Mapped[List["Association"]] = relationship(
        back_populates="child"
    )
```

