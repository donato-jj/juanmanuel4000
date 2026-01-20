
Eppp
import os, requests, logging
from flask import Flask, redirect, request, render_template_string, session
from dotenv import load_dotenv

load_dotenv()
app = Flask(__name__)
app.secret_key = os.urandom(32)

logging.basicConfig(level=logging.INFO)

APP_ID = os.getenv("IG_APP_ID")
APP_SECRET = os.getenv("IG_APP_SECRET")
REDIRECT_URI = os.getenv("REDIRECT_URI")

AUTH_URL = "https://www.facebook.com/v19.0/dialog/oauth"
TOKEN_URL = "https://graph.facebook.com/v19.0/oauth/access_token"
GRAPH_URL = "https://graph.facebook.com/v19.0/me?fields=id,username,account_type,media_count"

HTML = """
<!DOCTYPE html>
<html>
<head>
<title>Instagram Security Dashboard</title>
<style>
body {font-family:Arial;background:#0f172a;color:white}
.box {width:520px;margin:50px auto;padding:20px;background:#1e293b;border-radius:10px}
</style>
</head>
<body>
<div class="box">
<h2>Instagram Professional Dashboard</h2>
<p><b>User:</b> {{u}}</p>
<p><b>Account Type:</b> {{t}}</p>
<p><b>Posts:</b> {{m}}</p>
<p><small>OAuth Official Integration – No passwords stored – GDPR Safe</small></p>
</div>
</body>
</html>
"""

@app.route("/")
def login():
    url = f"{AUTH_URL}?client_id={APP_ID}&redirect_uri={REDIRECT_URI}&scope=instagram_basic,instagram_manage_insights"
    return redirect(url)

@app.route("/callback")
def callback():
    code = request.args.get("code")
    token = requests.get(TOKEN_URL, params={
        "client_id": APP_ID,
        "client_secret": APP_SECRET,
        "redirect_uri": REDIRECT_URI,
        "code": code
    }).json()

    access_token = token.get("access_token")
    session["token"] = access_token

    data = requests.get(GRAPH_URL, params={"access_token": access_token}).json()
    logging.info("Authorized account accessed legally.")

    return render_template_string(HTML, u=data.get("username"), t=data.get("account_type"), m=data.get("media_count"))

if __name__ == "__main__":
    app.run(debug=False)
