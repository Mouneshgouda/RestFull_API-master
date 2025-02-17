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




