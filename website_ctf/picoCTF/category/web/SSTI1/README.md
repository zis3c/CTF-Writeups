# 🔓 SSTI 1 - picoCTF Web Exploitation Writeup

## Challenge Metadata

| Detail | Value |
| :--- | :--- |
| **CTF** | picoCTF |
| **Challenge** | SSTI 1 |
| **Category** | Web Exploitation |
| **Difficulty** | Easy |
| **Author** | Venax |
| **Solver** | Radzi Zamri |
| **Goal** | Exploit Server-Side Template Injection to retrieve the flag |

### 🛠️ Tools Used

* **Web Browser** (Developer Tools)
* **Burp Suite** Community Edition

---

## 🔍 Challenge Overview

**SSTI 1** is a web exploitation challenge that demonstrates **Server-Side Template Injection** vulnerability in a Python Flask application. The application processes user input through a template engine without proper sanitization, allowing arbitrary code execution.

---

### 1. Reconnaissance (Identifying the Vulnerability)

The initial analysis revealed critical information about the application's technology stack through server response headers.

* **Action:** I sent a POST request to the application and examined the server response headers.
* **Finding:** The server header **`Werkzeug/3.0.3 Python/3.8.10`** identified this as a Flask application using Jinja2 templating, indicating potential SSTI vulnerability:

![Server response header](images/server-ver.png)

---

### 2. Analysis (Confirming SSTI)

To confirm the template injection vulnerability, I tested basic template expressions in the application.

* **Action:** I used **Burp Suite Repeater** to send a POST request with a simple template expression.
* **Confirmation:** The expression **`{{7*7}}`** was executed server-side, returning the calculated value **`49`**, confirming Jinja2 SSTI vulnerability:

![Output of the executed template expression {{7*7}}](images/test-ssti.png)

---

### 3. Exploit (Template Injection to RCE)

The solution required exploiting the SSTI vulnerability to access Python's file system and read the flag.

* **Enumeration:** I enumerated available Python classes to identify those with file access capabilities:

    ```python
    {{ ''.__class__.__mro__[1].__subclasses__() }}
    ```

* **Class Discovery:** After analyzing the class list, I identified that index **`283`** provided access to Python's built-in functions including **`open()`**:

![Python __builtins__ module contents showing open() function](images/func-open.png)

* **Flag Extraction:** Using the discovered file access capability, I constructed the final payload:

    ```python
    {{ ''.__class__.__mro__[1].__subclasses__()[283].__init__.__globals__.__builtins__.open('flag').read() }}
    ```

* **Execution:** The server executed the template injection and returned the flag in the response.

![Final output showing the retrieved flag](images/flag.png)

---

### 4. Flag:

`picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_4675f3fa}`

---

### 🧠 Key Concepts & Lessons Learned

* **Server-Side Template Injection (SSTI):** User input processed as template code without sanitization creates critical code execution vulnerabilities.
* **Technology Fingerprinting:** Server headers like **`Werkzeug/Python`** immediately identify Flask/Jinja2 applications and potential attack vectors.
* **Python Object Chain Exploitation:** Using **`__class__`**, **`__mro__`**, and **`__subclasses__()`** to traverse Python's inheritance hierarchy and access sensitive functions.
* **File System Access Through SSTI:** Leveraging built-in functions like **`open()`** through template injection to read server files demonstrates the severity of unpatched SSTI vulnerabilities.
* **Input Validation Importance:** Web applications must rigorously sanitize and validate all user input, especially when processed by template engines.
