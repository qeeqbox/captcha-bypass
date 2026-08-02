<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/captcha-bypass/main/content/captcha-bypass.svg"></p>

A threat actor is able to bypass the access controllers and gain access to the target

Clone this current repo recursively
```sh
git clone --recurse-submodules https://github.com/qeeqbox/captcha-bypass
```
Run the webapp using Python
```sh
python3 captcha-bypass/vulnerable-web-app/webapp.py
```
Open the webapp in your browser 127.0.0.1:5142 and click register
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/captcha-bypass/main/content/1.png"></p>
Fill in the username, password, email, and enter a wrong CAPTCHA answer
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/captcha-bypass/main/content/2.png"></p>
Right-click on the page and open Developer  Tools, find the hidden variable named debug in the post form. Then change the variable debug from 0 to 1 and hit complete
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/captcha-bypass/main/content/3.png"></p>
The test user is created without verifying the CAPTCHA
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/captcha-bypass/main/content/4.png"></p>
Log in as test
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/captcha-bypass/main/content/5.png"></p>
Successful login
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/captcha-bypass/main/content/6.png"></p>

## Code
When a user registers using the registration form, they will need to enter a CAPTCHA answer, the POST request has a hidden variable called debug that is checked, the values will be sent to the add_user() function
```py
elif parsed_url.path == "/register" and all(key in post_request_data for key in ["username","password","email","captcha","uuid"]):
    ret = self.add_user(post_request_data["username"][0],post_request_data["password"][0],post_request_data["email"][0],post_request_data["captcha"][0],post_request_data["uuid"][0],post_request_data["debug"][0])
    if ret == "valid":
        self.send_content(200, [('Content-type', 'text/html')], self.msg_page(f"User {post_request_data["username"][0]} created".encode("utf-8"), b"login"))
    elif ret == "captcha":
        self.send_content(200, [('Content-type', 'text/html')], self.msg_page(f"Wrong captcha".encode("utf-8"), b"login"))
    elif ret == "username":
        self.send_content(200, [('Content-type', 'text/html')], self.msg_page(f"User {post_request_data['username'][0]} already exists".encode("utf-8"), b"login"))
    else:
        self.send_content(200, [('Content-type', 'text/html')], self.msg_page(f"User {post_request_data["username"][0]} was not created".encode("utf-8"), b"login"))
    return
```
The add_user() allows a user to register with a wrong CAPTCHA with debug value 1
```py
def add_user(self, username,password,email,captcha,uuid,debug):
    try:
        with connect(DATABASE, isolation_level=None, check_same_thread=False) as connection:
            cursor = connection.cursor()
            results_user = cursor.execute("SELECT * FROM users WHERE username='%s'" % (username)).fetchone()
            if results_user:
                return "username"
            results_captcha = cursor.execute("SELECT * FROM captcha WHERE uuid='%s'" % (uuid)).fetchone()
            if results_captcha[3] == captcha or debug == "1":
                cursor.execute("INSERT into users(username, hash, email, department, access, is_admin) values(?,?,?,?,?,?)", (username, sha512(password.encode("utf-8")+SALT).hexdigest(),email,"none","profile,tickets",0))
                return "valid"
            else:
                return "captcha"
    except Exception as e:
        return str(e).encode("utf-8")
```