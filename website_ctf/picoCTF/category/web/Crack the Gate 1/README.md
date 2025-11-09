# 🚪 Crack the Gate 1 - Writeup

## Challenge Information

| **Category** | **Difficulty** | **Author** | **Solver** |
|--------------|----------------|------------|------------|
| Web Exploitation | Easy | Yahaya Meddy | Radzi Zamri |

### Challenge Description
Investigate a restricted web portal where a person of interest is hiding sensitive data. The login email (`ctf-player@picoctf.org`) is known, but the password remains unknown. The challenge suggests the developer may have left a secret bypass method.

---

## 🛠️ Tools Used
- **Web Browser** (Developer Tools)
- **Burp Suite** Community Edition
- **CyberChef**

---

## 🔍 Solution Walkthrough

### 1. Reconnaissance - Finding the Leak

The challenge hinted at potential developer notes hidden in the client-side code.

**Steps:**
1. Navigated to the login page
2. Used browser's **Inspect Element** to examine HTML source code
3. Discovered a commented-out, encoded string in the body

![HTML Source Code Inspection](./images/inspect-element.png)

### 2. Analysis - Decoding the Secret

The hints suggested using **ROT13** cipher for decoding.

**Steps:**
1. Extracted the encoded string from HTML comments
2. Used **CyberChef** with ROT13 operation to decode the message
3. Revealed the bypass method: `NOTE: Jack - temporary bypass: use header "X-Dev-Access: yes"`

![CyberChef ROT13 Decoding](./images/cyberchef.png)

### 3. Exploitation - Header Injection

**Steps:**
1. Intercepted a failed login attempt using **Burp Suite Repeater**
2. Added the custom header: `X-Dev-Access: yes`
3. Sent the modified request:

```http
POST /login HTTP/1.1
Host: amiable-citadel.picoctf.net:57543
X-Dev-Access: yes
Content-Type: application/json
Content-Length: 48

{"email":"ctf-player@picoctf.org","password":"test"}
