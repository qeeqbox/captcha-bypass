<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/captcha-bypass/main/content/captcha-bypass.svg"></p>

## CAPTCHA Bypass
A CAPTCHA bypass is an attack in which a perpetrator circumvents or defeats a CAPTCHA (Completely Automated Public Turing test to tell Computers and Humans Apart) mechanism that is designed to distinguish human users from automated bots. Successfully bypassing a CAPTCHA enables attackers to automate actions that the CAPTCHA was intended to prevent, such as creating fake accounts, submitting spam, scraping websites, or executing credential stuffing attacks.

## CAPTCHA Technologies
- Image-based CAPTCHAs: Users identify characters, objects, or images.
- Audio-based CAPTCHAs: Users listen to spoken words or numbers and transcribe them.
- reCAPTCHA: A Google service that utilizes risk analysis, behavioral signals, and interactive challenges to verify users.
- hCaptcha: A CAPTCHA service that combines image recognition challenges with behavioral analysis to identify users.
- Cloudflare Turnstile: A modern CAPTCHA alternative that verifies users with minimal interaction by analyzing browser and behavioral signals.

## How CAPTCHA Bypass Works
1. Identify the CAPTCHA Mechanism: The attacker determines the type of CAPTCHA employed by the application and analyzes how it is validated.
2. Circumvent the CAPTCHA: Attackers can bypass CAPTCHA protections using several techniques:
   - Human CAPTCHA-Solving Services: Attackers may employ specialized services that use human workers to solve challenges in real time.
   - Automated CAPTCHA Solvers: Machine learning and computer vision techniques can automatically solve certain image- or text-based CAPTCHAs.
   - Exploiting Implementation Weaknesses: Poor CAPTCHA implementations may allow attackers to bypass verification.
   - CAPTCHA Token Replay: If CAPTCHA tokens remain valid after their initial use or are not properly linked to a specific session, attackers may capture and reuse previously valid tokens to bypass verification.
   - Social Engineering: Attackers may trick legitimate users into solving CAPTCHAs on their behalf, enabling automated systems to operate without interruption.

## Impact of CAPTCHA Bypass
Successful CAPTCHA bypass attacks can lead to the following consequences:
- Spam and Automated Abuse: Increased spam and misuse of services.
- Credential Stuffing and Brute-Force Attacks: Attackers can automate login attempts using stolen usernames and passwords or perform numerous password guesses.
- Web Scraping: Bots can rapidly collect protected content, pricing information, or personal data.
- Resource Exhaustion: A high volume of automated requests can deplete server resources and degrade application performance.
- Financial Loss: Organizations may incur costs related to fraud, infrastructure scaling, incident response, and abuse prevention.
- Reputation Damage: Successful automated abuse can erode customer trust and damage an organization's reputation.

## CAPTCHA Bypass Mitigation Strategies
To prevent CAPTCHA Bypass attacks:
- Utilize Modern CAPTCHA Solutions: Consider using tools like reCAPTCHA, hCaptcha, or Cloudflare. These systems employ behavioral analysis, browser signals, and machine learning to effectively differentiate between humans and automated bots.
- Validate CAPTCHA Responses on the Server: Always perform validation on the server side; never rely solely on client-side validation.
- Implement Short-Lived, Single-Use Tokens: Use tokens that expire quickly and can be used only once to enhance security.
- Enforce Rate Limiting: Limit the number of requests that can be made from a single IP address, device, or user account to decrease the chances of automated abuse.
- Monitor User Behavior: Analyze user behavior to distinguish between legitimate users and automated bots.
- Adopt Multi-Layered Security: Combine CAPTCHA with additional security measures, such as rate limiting, account lockout policies, multi-factor authentication (MFA), IP reputation services, and bot detection systems. Relying on a single security control is not sufficient to prevent automated attacks.
- Keep CAPTCHA Implementations Updated: Regularly update your CAPTCHA libraries and integrations to ensure you benefit from the latest security enhancements and protections against new attack techniques.

## CAPTCHA Bypass Example
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
