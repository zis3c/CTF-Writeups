# Crack the Gate 1 - Writeup

---

**Title:** Crack the Gate 1
**Category:** Web
**Difficulty:** Easy
**Author:** Yahaya Meddy
**Writeup by:** Radzi Zamri

**Goal:** Find the flag by bypassing the login gate.

**Tools:**
-   **Web Browser:** For looking at the website.
-   **Burp Suite Community Edition:** For changing the web request.
-   **CyberChef:** For decoding secret messages.

### Steps

#### 1. Recon (First Look)

* **The Problem:** The challenge description and hints told us that a developer might have left a secret note in the code, but it might not be easy to read.
* **Action:** I opened the website and used the browser's **Inspect Element** tool to check the **HTML source code** (Screenshot 518).
* **What I Found:** Inside the HTML, I found a commented message: `ABGR: Wnpx - grzcbenel olcnff: hfr urnqre "K-Qri-Npprff: lrf"`.



---

#### 2. Analysis (Decoding the Secret)

* **The Second Hint:** The second hint said that rotating letters by **13 positions** is a common trick. This means the message is encoded with **ROT13**.
* **The Decoder:** I used **CyberChef** (Screenshot 512) to decode the hidden message.
* **The Result:** The decoded message was: `Jack - temporary bypass: use header "X-Dev-Access: yes"`.

This message gives us a special key (an HTTP header) to bypass the login.

---

#### 3. Exploit (Using the Key)

1.  I tried to log in with a random password (`test`) and caught the request using **Burp Suite Repeater** (Screenshot 513).
2.  I then added the secret key found in the source code to the request: `X-Dev-Access: yes`. I put this on a new line right after the `Host` line.

    ```http
    POST /login HTTP/1.1
    Host: amiable-citadel.picoctf.net:57543
    X-Dev-Access: yes  <-- ADDED SECRET KEY
    Content-Type: application/json
    ...
    ```

3.  I sent the request. This successful bypass gave me the flag in the response (Screenshot 515).

---

#### 4. Flag:

`picoCTF{brut4_f0rc4_b3a957eb}`

---

### Notes / Takeaway

* **Look Everywhere:** Always check comments and hidden parts of the source code.
* **ROT13 is Common:** If you see a hint about rotating letters, think of ROT13.
* **Headers are Doors:** You can often find a way to cheat a system by adding or changing special **HTTP headers** in your web requests.
