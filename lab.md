```python

from flask import Flask, jsonify, request
from flask_jwt_extended import JWTManager, jwt_required, create_access_token, get_jwt_identity

app = Flask(__name__)

# Secret key for encoding JWTs
app.config['JWT_SECRET_KEY'] = 'your_secret_key'  # Change this!

# Initialize the JWT Manager
jwt = JWTManager(app)

# Sample users for demonstration (In real-world, use a database)
users = {"admin": "password123"}

@app.route('/login', methods=['POST'])
def login():
    # Get username and password from the request
    username = request.json.get('username', None)
    password = request.json.get('password', None)

    # Check if user exists and password is correct
    if username in users and users[username] == password:
        # Create a JWT token valid for 30 minutes
        access_token = create_access_token(identity=username)
        return jsonify(access_token=access_token), 200
    else:
        return jsonify({"msg": "Bad username or password"}), 401

@app.route('/protected', methods=['GET'])
@jwt_required()
def protected():
    # Access the identity of the current user with `get_jwt_identity`
    current_user = get_jwt_identity()
    return jsonify(logged_in_as=current_user), 200

if __name__ == '__main__':
    app.run(debug=True)



```


- Steps to Test:
 Login Request: Send a POST request to /login with the correct username and password.
```json
{
  "username": "admin",
  "password": "password123"
}

```
- Access Protected Route: Once you receive the JWT token, include it in the Authorization header as a Bearer token and send a GET request to /protected:
```
Authorization: Bearer <your_jwt_token>

{
  "logged_in_as": "admin"
}

```
### 4

```python

from flask import Flask, jsonify, request
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

# Setup SQLite database
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///example.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# Initialize SQLAlchemy
db = SQLAlchemy(app)

# Example model
class Item(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    description = db.Column(db.String(255), nullable=False)

# Initialize the database (Uncomment to create database schema if needed)
with app.app_context():
    db.create_all()
    
    # Insert test data if needed
    if Item.query.count() == 0:  # Check if the database is empty
        db.session.add(Item(name="Item 1", description="This is item 1"))
        db.session.add(Item(name="Item 2", description="This is item 2"))
        db.session.add(Item(name="Item 3", description="This is item 3"))
        db.session.add(Item(name="Item 4", description="This is item 4"))
        db.session.add(Item(name="Item 5", description="This is item 5"))
        db.session.commit()

@app.route('/items', methods=['GET'])
def get_items():
    # Get pagination parameters from query string
    page = request.args.get('page', 1, type=int)  # Default to page 1
    per_page = request.args.get('per_page', 10, type=int)  # Default to 10 items per page
    
    # Query items with pagination
    items = Item.query.paginate(page=page, per_page=per_page, error_out=False)
    
    # Prepare data to return
    result = {
        'total': items.total,
        'pages': items.pages,
        'current_page': items.page,
        'items': [
            {
                'id': item.id,
                'name': item.name,
                'description': item.description
            }
            for item in items.items
        ]
    }
    
    return jsonify(result)

if __name__ == '__main__':
    app.run(debug=True)
```

### 5
```python

from flask import Flask, request, jsonify
from marshmallow import Schema, fields, validate, ValidationError
from werkzeug.exceptions import HTTPException

# Initialize Flask app
app = Flask(__name__)

# Define Marshmallow schema for data validation
class UserSchema(Schema):
    name = fields.String(required=True, validate=validate.Length(min=1, error="Name cannot be empty"))
    email = fields.Email(required=True, error="Invalid email address")
    age = fields.Integer(required=True, validate=validate.Range(min=18, error="Age must be at least 18"))
    
# Custom error handler for Marshmallow validation errors
@app.errorhandler(ValidationError)
def handle_validation_error(error):
    response = jsonify({"message": "Validation failed", "errors": error.messages})
    response.status_code = 400
    return response

# General error handler
@app.errorhandler(HTTPException)
def handle_http_exception(error):
    response = jsonify({"message": error.description})
    response.status_code = error.code
    return response

# POST endpoint to create a user
@app.route('/users', methods=['POST'])
def create_user():
    # Validate the incoming JSON using Marshmallow schema
    try:
        user_data = request.get_json()
        user_schema = UserSchema()
        result = user_schema.load(user_data)  # This validates and deserializes the data
    except ValidationError as err:
        return handle_validation_error(err)

    # Simulate saving the user (in real applications, save to database)
    return jsonify({"message": "User created successfully", "data": result}), 201

# Run the Flask app
if __name__ == "__main__":
    app.run(debug=True)




```
- Test the API using Postman or cURL:
 Valid Request:

```json
POST /users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "age": 25
}
```
```json
Response:

json
Copy
{
  "message": "User created successfully",
  "data": {
    "name": "John Doe",
    "email": "john.doe@example.com",
    "age": 25
  }
}
```
Invalid Request (e.g., age less than 18):

``` json
POST /users
Content-Type: application/json

{
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "age": 15
}
```
```json
Response:

json
Copy
{
  "message": "Validation failed",
  "errors": {
    "age": ["Age must be at least 18"]
  }
}
```

