# 🚪 Crack the Gate 1

---

## Challenge Metadata

| Detail | Value |
| :--- | :--- |
| **Title** | Crack the Gate 1 |
| **Category** | Web |
| **Difficulty** | Easy |
| **Author** | Yahaya Meddy |
| **Solver** | Radzi Zamri |
| **Goal** | Find the flag by bypassing the login mechanism. |

### 🛠️ Tools Used

* **Web Browser:** Chrome/Firefox (Developer Tools)
* **Burp Suite** Community Edition: For intercepting and modifying HTTP requests.
* **CyberChef:** For decoding common ciphers like ROT13.

## 💡 Solution Methodology

This challenge tests the ability to **discover and decode developer notes** hidden in the client-side code, which leads to an **authentication bypass** using a custom HTTP header.

### 1. Reconnaissance and Code Inspection

The first hint suggested that developers sometimes leave hidden notes in the code.

* **Action:** I navigated to the login page and used the browser's **Inspect Element** to examine the HTML source code.
* **Finding:** A suspicious, non-plain-text comment was found in the `<body>` of the HTML:
    ```html
    ```
    ![HTML source showing the ROT13 comment](https://raw.githubusercontent.com/zis3c/CTF-Writeups/main/website_ctf/picoCTF/category/web/Crack%20the%20Gate%201/assets/inspect-element.png)

### 2. Analysis and Cipher Decoding (ROT13)

The second hint pointed towards **rotating letters by 13 positions**, identifying the comment as a **ROT13** cipher.

* **Action:** I input the encoded message into **CyberChef** and applied the ROT13 operation.
* **Result:** The decoded message provided the key for the bypass:
    `NOTE: Jack - temporary bypass: use header "X-Dev-Access: yes"`
    
    ![CyberChef decoding the message via ROT13](https://raw.githubusercontent.com/zis3c/CTF-Writeups/main/website_ctf/picoCTF/category/web/Crack%20the%20Gate%201/assets/cyberchef.png)

### 3. Exploitation via HTTP Header Manipulation

The solution required injecting the custom HTTP header found in the note into the login request.

1.  I initiated a failed login attempt with dummy credentials (`test`) and intercepted the `POST /login` request using **Burp Suite Repeater** (test-request.jpg).
2.  I then modified the request by inserting the required header on a new line: `X-Dev-Access: yes`.

    ```http
    POST /login HTTP/1.1
    Host: amiable-citadel.picoctf.net:57202
    X-Dev-Access: yes  <-- ADDED BYPASS HEADER
    Content-Type: application/json
    ...
    {"email":"ctf-player@picoctf.org","password":"test"}
    ```

3.  Sending this modified request resulted in a **HTTP 200 OK** response that successfully contained the flag.
    
    ![Burp Suite showing the request with the added header and the flag in the response](https://raw.githubusercontent.com/zis3c/CTF-Writeups/main/website_ctf/picoCTF/category/web/Crack%20the%20Gate%201/assets/flag.jpg)

## ✅ Flag

`picoCTF{brut4_f0rc4_b3a957eb}`

---

### 🧠 Key Concepts & Lessons Learned

* **Client-Side Leakage:** Always check the HTML source for developer comments; sometimes, critical bypass information is left behind.
* **Cipher Recognition:** Hints about "rotating letters" are a strong giveaway for the simple **ROT13** substitution cipher.
* **HTTP Header Abuse:** Custom HTTP headers, especially non-standard ones (like `X-Dev-Access`), can be used for authentication or authorization bypasses if they are not properly removed before production deployment.
