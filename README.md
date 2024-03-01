from flask import Flask, request, jsonify
import jwt
import datetime

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your_secret_key'

users = {
    'user1': 'password1',
    'user2': 'password2'
}

def generate_token(username):
    expiration_date = datetime.datetime.utcnow() + datetime.timedelta(hours=1)
    token = jwt.encode({'username': username, 'exp': expiration_date}, app.config['SECRET_KEY'], algorithm='HS256')
    return token.decode('utf-8')

def verify_token(token):
    try:
        payload = jwt.decode(token, app.config['SECRET_KEY'], algorithms=['HS256'])
        return payload['username']
    except jwt.ExpiredSignatureError:
        return 'Signature expired. Please log in again.'
    except jwt.InvalidTokenError:
        return 'Invalid token. Please log in again.'

@app.route('/login', methods=['POST'])
def login():
    auth = request.authorization
    if auth and auth.username in users and auth.password == users[auth.username]:
        token = generate_token(auth.username)
        return jsonify({'token': token}), 200
    return jsonify({'message': 'Invalid username or password'}), 401

@app.route('/protected', methods=['GET'])
def protected():
    token = request.headers.get('Authorization')
    if not token:
        return jsonify({'message': 'Token is missing'}), 401

    token = token.split(' ')[1]
    username = verify_token(token)
    if isinstance(username, str):
        return jsonify({'message': username}), 401

    return jsonify({'message': 'Hello, {}! You accessed a protected resource.'.format(username)}), 200

if __name__ == '__main__':
    app.run(debug=True)
