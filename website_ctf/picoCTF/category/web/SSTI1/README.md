# 🔓 SSTI 1 - picoCTF Web Exploitation Writeup

This is a comprehensive writeup for the **SSTI 1** challenge from picoCTF, detailing the process of exploiting a **Server-Side Template Injection (SSTI)** vulnerability in a Flask/Jinja2 application to achieve Remote Code Execution (RCE) and retrieve the flag.

---

## ⚙️ Challenge Metadata

| Detail | Value |
| :--- | :--- |
| **CTF** | picoCTF |
| **Challenge** | SSTI 1 |
| **Category** | Web Exploitation |
| **Difficulty** | Easy |
| **Author** | Venax |
| **Solver** | Radzi Zamri |
| **Goal** | Find the flag by exploiting an SSTI vulnerability. |

### 🛠️ Tools Used

* **Web Browser**
* **Burp Suite** Community Edition

---

## 🔍 Challenge Overview

The challenge involved a simple web application that received user input and processed it using a server-side template engine, which was identified as a common Server-Side Template Injection (SSTI) scenario. The objective was to exploit this flaw to read a hidden file named `flag`.

---

### 1. Reconnaissance & Technology Stack Identification

The first step was to identify the underlying technology to determine the correct template syntax and exploitation vector.

* **Action:** Intercepted the initial HTTP response.
* **Finding:** The server header indicated a Python/Flask application: `Server: Werkzeug/3.0.3 Python/3.8.10`.
* **Conclusion:** The application likely uses the **Jinja2** template engine, a known target for SSTI attacks.

### 2. Analysis (Confirming SSTI)

To confirm the vulnerability, a basic mathematical expression test was performed.

* **Action:** Sent a test payload in the `content` parameter: `{{7*7}}`.
* **Result:** The server returned the calculated output, **49**.
* **Confirmation:** This proved that the user input was being executed as template code, confirming the **SSTI vulnerability**.

### 3. Exploit (Achieving RCE to Read the Flag)

The exploitation process involved using Jinja2's object introspection to access system-level functions.

#### A. Introspection and Class Discovery

We need a path to a function that can read files, like `open()`. This requires navigating the Python object hierarchy.

* **Action:** Sent the payload to list all accessible classes:
    ```python
    {{ ''.__class__.__mro__[1].__subclasses__() }}
    ```
* **Finding:** After analyzing the output (as shown in `test-templPython.jpg`), the class at index **283** was identified as providing access to the `__builtins__` module, which contains the `open()` function.

#### B. Reading the Flag File

The final payload was constructed to access the `open()` function and read the file named `flag`.

* **Final RCE Payload:**
    ```python
    {{ ''.__class__.__mro__[1].__subclasses__()[283].__init__.__globals__.__builtins__.open('flag').read() }}
    ```
* **Action:** Sent the payload via Burp Suite Repeater.
* **Result:** The server executed the command and returned the flag in the HTTP response body.

---

### 4. Flag:

`picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_4675f3fa}`

---

### 🧠 Key Concepts & Lessons Learned

* **SSTI Chains:** The core exploit relies on navigating the Python class hierarchy to break out of the template sandbox: `str` -> `object` -> `subclasses` -> `globals` -> `builtins/os`.
* **Builtin Functions:** The Python `__builtins__` module is a powerful resource in an SSTI attack as it grants access to fundamental functions like `open()`.
* **Defense in Depth:** This vulnerability highlights that simply using a template engine is not secure; all user input must be meticulously sanitized or passed only as simple, non-executable data to the template.
