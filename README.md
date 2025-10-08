# ds1_pbl
DSA phase 1 
SecureCloudStorage/
├─ backend/
│   ├─ app.py
│   ├─ auth.py
│   ├─ storage.py
│   ├─ requirements.txt
│   └─ .env         # Add sensitive info here (don’t upload)
├─ frontend/
│   ├─ index.html
│   ├─ login.html
│   ├─ dashboard.html
│   └─ style.css
├─ database/
│   └─ database.db  # SQLite DB (optional to include; can be created at runtime)
├─ README.md
└─ .gitignore


---

1. backend/app.py (Flask main file)

from flask import Flask, render_template, request, redirect, url_for, session
from auth import login_user, register_user
from storage import upload_file, download_file

app = Flask(__name__)
app.secret_key = "your_secret_key"  # Should be in .env

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        if login_user(username, password):
            session['user'] = username
            return redirect(url_for('dashboard'))
        else:
            return "Invalid credentials!"
    return render_template('login.html')

@app.route('/dashboard')
def dashboard():
    if 'user' not in session:
        return redirect(url_for('login'))
    return render_template('dashboard.html')

@app.route('/upload', methods=['POST'])
def upload():
    if 'user' not in session:
        return redirect(url_for('login'))
    file = request.files['file']
    upload_file(file, session['user'])
    return "File uploaded successfully!"

@app.route('/download/<filename>')
def download(filename):
    if 'user' not in session:
        return redirect(url_for('login'))
    return download_file(filename, session['user'])

if __name__ == '__main__':
    app.run(debug=True)


---

2. backend/auth.py (User authentication)

import sqlite3
from werkzeug.security import generate_password_hash, check_password_hash

def get_db():
    return sqlite3.connect('../database/database.db')

def register_user(username, password):
    conn = get_db()
    cursor = conn.cursor()
    hashed_password = generate_password_hash(password)
    cursor.execute("INSERT INTO users (username, password) VALUES (?, ?)", (username, hashed_password))
    conn.commit()
    conn.close()

def login_user(username, password):
    conn = get_db()
    cursor = conn.cursor()
    cursor.execute("SELECT password FROM users WHERE username = ?", (username,))
    row = cursor.fetchone()
    conn.close()
    if row and check_password_hash(row[0], password):
        return True
    return False


---

3. backend/storage.py (File upload/download)

import os

UPLOAD_FOLDER = '../uploads/'

def upload_file(file, username):
    user_folder = os.path.join(UPLOAD_FOLDER, username)
    os.makedirs(user_folder, exist_ok=True)
    file.save(os.path.join(user_folder, file.filename))

def download_file(filename, username):
    from flask import send_from_directory
    user_folder = os.path.join(UPLOAD_FOLDER, username)
    return send_from_directory(user_folder, filename)


---

4. frontend/index.html (Homepage)

<!DOCTYPE html>
<html>
<head>
    <title>Secure Cloud Storage</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>Welcome to Secure Cloud Storage</h1>
    <a href="login.html">Login</a>
</body>
</html>


---

5. .gitignore

__pycache__/
.env
database/database.db
uploads/


---

6. README.md (Short project description)

# Secure Cloud Storage System

A simple cloud storage project with user authentication and file upload/download functionality. Built with Python (Flask), SQLite, and basic HTML/CSS frontend.


---
