
# Flask-SQLAlchemy Lab 2: Customer-Item Review System

## Learning Goals
In this lab, you will learn to:
- Use **Flask-SQLAlchemy** to define a data model with relationships.
- Implement an **association proxy** for a model.
- Use **SQLAlchemy-Serializer** to serialize an object with relationships.

## Setup Instructions

### 1. **Fork and Clone the Repository**
- Fork the lab repository on GitHub.
- Clone the forked repository to your local machine:
  ```bash
  git clone https://github.com/learn-co-curriculum/python-p4-v2-flask-sqlalchemy-lab-2
  cd python-p4-v2-flask-sqlalchemy-lab-2
  ```

### 2. **Install Dependencies with Pipenv**
- If you don't have `pipenv` installed, install it globally:
  ```bash
  pip install pipenv
  ```

- Inside your project folder, install the required dependencies:
  ```bash
  pipenv install
  ```

- Activate the virtual environment:
  ```bash
  pipenv shell
  ```

### 3. **Setup the Flask Application**

- Navigate to the `server` directory:
  ```bash
  cd server
  ```

- Initialize Flask-Migrate to handle database migrations:
  ```bash
  flask db init
  ```

- Create the initial migration:
  ```bash
  flask db migrate -m "initial migration"
  ```

- Apply the migration to the database:
  ```bash
  flask db upgrade head
  ```

## Task #1: Add Review and Relationships with Customer and Item

### 1. **Modify `models.py` to Add the `Review` Model**

In `server/models.py`, add the `Review` model with the following attributes:
- A `comment` column.
- `customer_id` and `item_id` foreign key columns linking to `customers` and `items`, respectively.

```python
class Review(db.Model):
    __tablename__ = 'reviews'
    id = db.Column(db.Integer, primary_key=True)
    comment = db.Column(db.String)
    customer_id = db.Column(db.Integer, db.ForeignKey('customers.id'))
    item_id = db.Column(db.Integer, db.ForeignKey('items.id'))

    customer = db.relationship('Customer', back_populates='reviews')
    item = db.relationship('Item', back_populates='reviews')
```

### 2. **Update the `Customer` and `Item` Models**

Add relationships to `Customer` and `Item` models:
```python
class Customer(db.Model):
    __tablename__ = 'customers'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)
    reviews = db.relationship('Review', back_populates='customer')

class Item(db.Model):
    __tablename__ = 'items'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)
    price = db.Column(db.Float)
    reviews = db.relationship('Review', back_populates='item')
```

### 3. **Run Migrations**

After making the changes, perform a migration to add the `Review` model:
```bash
flask db migrate -m 'add review'
flask db upgrade head
```

### 4. **Test the Review Model**

Run the tests for the `Review` model in `testing/review_test.py`:
```bash
pytest testing/review_test.py
```

If the tests pass, proceed with seeding data.

### 5. **Seed the Database**

To populate the database with sample data, run:
```bash
python seed.py
```

Verify the data is populated by using Flask shell or an SQLite viewer.

## Task #2: Add Association Proxy

### 1. **Update the `Customer` Model**

Add an **association proxy** to the `Customer` model for accessing items through reviews:
```python
from sqlalchemy.ext.associationproxy import association_proxy

class Customer(db.Model):
    __tablename__ = 'customers'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)
    reviews = db.relationship('Review', back_populates='customer')
    items = association_proxy('reviews', 'item')
```

### 2. **Test the Association Proxy**

Run the tests for the association proxy in `testing/association_proxy_test.py`:
```bash
pytest testing/association_proxy_test.py
```

## Task #3: Add Serialization

### 1. **Implement Serialization for Customer, Item, and Review Models**

Use `SerializerMixin` to make the models serializable and avoid recursion issues:
```python
from sqlalchemy_serializer import SerializerMixin

class Customer(db.Model, SerializerMixin):
    __tablename__ = 'customers'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)
    reviews = db.relationship('Review', back_populates='customer')
    items = association_proxy('reviews', 'item')
    serialize_rules = ('-reviews.customer',)

class Item(db.Model, SerializerMixin):
    __tablename__ = 'items'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)
    price = db.Column(db.Float)
    reviews = db.relationship('Review', back_populates='item')
    serialize_rules = ('-reviews.item',)

class Review(db.Model, SerializerMixin):
    __tablename__ = 'reviews'
    id = db.Column(db.Integer, primary_key=True)
    comment = db.Column(db.String)
    customer_id = db.Column(db.Integer, db.ForeignKey('customers.id'))
    item_id = db.Column(db.Integer, db.ForeignKey('items.id'))
    customer = db.relationship('Customer', back_populates='reviews')
    item = db.relationship('Item', back_populates='reviews')
    serialize_rules = ('-customer.reviews', '-item.reviews')
```

### 2. **Test Serialization**

Run the serialization tests in `testing/serialization_test.py`:
```bash
pytest testing/serialization_test.py
```

## Final Testing

### 1. **Run All Tests**

To ensure everything works as expected, run all tests:
```bash
pytest
```

This will run all the tests in the project, ensuring that the relationships, association proxies, and serialization are correctly implemented.

---

## Folder Structure

```
repository-name/
├── server/
│   ├── models.py
│   ├── app.py
│   ├── seed.py
│   └── __init__.py
├── testing/
│   ├── association_proxy_test.py
│   ├── review_test.py
│   ├── serialization_test.py
│   ├── conftest.py
│   └── __init__.py
├── Pipfile
├── Pipfile.lock
├── pytest.ini
└── README.md
```
