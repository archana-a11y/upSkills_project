# upSkills_project

PASSWORD MANAGER
app.py

from flask import Flask, render_template, request, redirect, session
from pymongo import MongoClient
import bcrypt
import random
import string

app = Flask(__name__)
app.secret_key = "supersecretkey"

# MongoDB Connection
client = MongoClient("YOUR_MONGODB_CONNECTION_STRING")
db = client["password_manager"]
users = db["users"]
passwords = db["passwords"]


# Generate Strong Password
def generate_password(length=12):
    characters = string.ascii_letters + string.digits + "!@#$%^&*"
    return ''.join(random.choice(characters) for i in range(length))


# Home
@app.route("/")
def home():
    return redirect("/login")


# Register
@app.route("/register", methods=["GET", "POST"])
def register():
    if request.method == "POST":
        username = request.form["username"]
        password = request.form["password"]

        hashed = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt())

        users.insert_one({
            "username": username,
            "password": hashed
        })

        return redirect("/login")

    return render_template("register.html")


# Login
@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form["username"]
        password = request.form["password"]

        user = users.find_one({"username": username})

        if user and bcrypt.checkpw(password.encode('utf-8'), user["password"]):
            session["username"] = username
            return redirect("/dashboard")

    return render_template("login.html")


# Dashboard
@app.route("/dashboard", methods=["GET", "POST"])
def dashboard():
    if "username" not in session:
        return redirect("/login")

    if request.method == "POST":
        website = request.form["website"]
        account = request.form["account"]
        password = request.form["password"]

        passwords.insert_one({
            "username": session["username"],
            "website": website,
            "account": account,
            "password": password
        })

    user_passwords = passwords.find({"username": session["username"]})
    return render_template("dashboard.html", data=user_passwords)


# Generate Password
@app.route("/generate")
def generate():
    if "username" not in session:
        return redirect("/login")

    pwd = generate_password()
    return pwd


# Logout
@app.route("/logout")
def logout():
    session.pop("username", None)
    return redirect("/login")


if __name__ == "_main_":
    app.run(debug=True)
