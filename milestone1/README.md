## Milestone 1 – User Authentication System

---

## 📌 Description

In this milestone, we implemented a secure user authentication system for **PolicyNav** using **Streamlit**, **JWT (JSON Web Tokens)**, and **SQLite**.

The system provides functionality including **Signup**, **Login**, **Forgot Password**, **Reset Password** and a **JWT-protected Dashboard**, ensuring proper security practices and authentication workflows.

Passwords and security answers are securely hashed using **bcrypt**, and protected routes are validated via **JWT token verification**.

---

## ✅ Features Implemented

**User Signup/Login with Validation**

- Mandatory input checks
- Email format validation
- Strong password (Alphanumeric)
- Duplicate username/email prevention

**Secure Login System**

- Credential verification
- JWT token generation on success

**Forgot Password Flow**

- Email verification
- Security question validation
- Strong password reset rules

**JWT-Based Authentication**

- Token generation
- Protected dashboard access
- Session validation

**Security Best Practices**

- bcrypt password hashing
- No plaintext password storage
- Input validation & sanitization

**SQLite Database Integration**

- User data persistence

**Ngrok Integration**

- Public access for Streamlit app
- Useful for Colab & demos

Ngrok website: ngrok.com

- Navigate to Authtoken and copy the token_id
- Paste the token_id ( ngrok.set_auth_token("PASTE_TOKEN") )

## ▶️ How to Run the Application

### 1. Install Dependencies

```bash
!pip install streamlit bcrypt PyJWT pyngrok
```

### 2. Streamlit

```bash
!streamlit run app.py &>/content/log.txt &
```

### 3. Use Ngrok to expose the app

```bash
from pyngrok import ngrok
ngrok.set_auth_token("TOKEN")
print(ngrok.connect(8501))
```

Open the public link after running Ngrok, example link below:
Public URL: https://731d-34-11-85-163.ngrok-free.app/

### 🔑 Login Page

![Login Page](screenshots/login.png)

### 🔐 Signup Page

![Signup Page](screenshots/signup.png)

### 🧭 Dashboard

![Dashboard](screenshots/dashboard.png)

### 🔒 Reset

![Dashboard](screenshots/reset.png)

### 🔁 Forgot Password Page

# ![Forgot Password](screenshots/forgot.png)

# Infosys_Springboard_PolicyNav_Public-\_Policy_Navigation_Using_AI

Gaurav Mehta | Flagship Module: PolicyNav – Public Policy Navigation Using AI
