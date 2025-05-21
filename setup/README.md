<h1>
  <span class="headline">Selenium: Debugging and Stabilizing Tests</span>
  <span class="subhead">Setup</span>
</h1>

## Setup

### 1: Create Your Project Folder

In this lesson, you’ll start fresh with a new project:

Open a terminal and run:

```bash
mkdir selenium-debugging-and-stabilizing-tests
cd selenium-debugging-and-stabilizing-tests
```

### 2: Set Up a Virtual Environment

Create and activate a virtual environment inside this new folder:

```bash
python -m venv venv
```

Activate it:

**macOS/Linux:**

```bash
source venv/bin/activate
```

**Windows:**

```bash
venv\Scripts\activate
```

You should now see (venv) at the start of your terminal prompt.

### 3: Install Selenium

With the virtual environment active, install Selenium using pip:

```bash
pip install selenium
```

Check that it installed correctly:

```bash
pip show selenium
```

> ✅ If Selenium installed correctly, you’ll see output with its version and location.

### 4: Create Your Project Files

For this module, you’ll use three small test pages—each designed to help you practice a different Selenium skill: handling modals, waits, and logging/debugging.

In your terminal, create the files:

```bash
touch example_1_modal.html example_2_add_to_cart.html example_3_login.html
```

#### Copy and Paste the following into **example_1_modal.html**.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Debug Practice Page</title>
    <style>
      body {
        font-family: Arial, sans-serif;
        background: #f9f9f9;
        padding: 2rem;
      }
      .container {
        background: #ffffff;
        border: 1px solid #ccc;
        padding: 2rem;
        max-width: 600px;
        position: relative;
      }
      .action-btn {
        background: #28a745;
        color: white;
        border: none;
        padding: 0.5rem 1rem;
        border-radius: 4px;
        cursor: pointer;
      }
      .notice {
        color: #666;
        font-size: 0.9rem;
        margin-top: 1rem;
      }

      /* Modal styles */
      .modal {
        display: none;
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.6);
        justify-content: center;
        align-items: center;
        z-index: 1000;
      }
      .modal-content {
        background: #fff;
        padding: 1.5rem;
        border-radius: 8px;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.25);
        text-align: center;
      }
      .modal-content button {
        margin-top: 1rem;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <h1>Welcome</h1>
      <p>This is a test page designed for Selenium debugging practice.</p>

      <!-- Working button (but with a different ID than the test expects) -->
      <button id="launch-action" class="action-btn">Launch Modal</button>

      <p class="notice">
        Note: Some elements may be loaded dynamically in a real app.
      </p>
    </div>

    <!-- Modal -->
    <div class="modal" id="debug-modal">
      <div class="modal-content">
        <h2>Modal</h2>
        <p>This modal was triggered by JavaScript.</p>
        <button id="close-modal">Close</button>
      </div>
    </div>

    <script>
      const launchBtn = document.getElementById('launch-action');
      const modal = document.getElementById('debug-modal');
      const closeBtn = document.getElementById('close-modal');

      launchBtn.addEventListener('click', () => {
        modal.style.display = 'flex';
      });

      closeBtn.addEventListener('click', () => {
        modal.style.display = 'none';
      });
    </script>
  </body>
</html>
```

#### Copy and Paste the following into **example_2_add_to_cart.html**.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Delayed Add to Cart</title>
    <style>
      body {
        font-family: sans-serif;
        padding: 2rem;
        background: #f5f5f5;
      }

      #spinner {
        width: 40px;
        height: 40px;
        border: 5px solid #ccc;
        border-top: 5px solid #333;
        border-radius: 50%;
        animation: spin 1s linear infinite;
        margin-bottom: 1rem;
      }

      @keyframes spin {
        to {
          transform: rotate(360deg);
        }
      }

      #confirmation-msg {
        margin-top: 1rem;
        color: green;
        font-weight: bold;
        display: none;
      }

      #add-button:disabled {
        background-color: #aaa;
        cursor: not-allowed;
      }

      #add-button {
        background-color: #007bff;
        color: white;
        border: none;
        padding: 0.6rem 1.2rem;
        font-size: 1rem;
        border-radius: 4px;
        cursor: pointer;
      }
    </style>
  </head>
  <body>
    <h1>Shopping Cart</h1>
    <p>This is a test page designed for Selenium debugging practice.</p>

    <div id="spinner"></div>

    <button id="add-button" disabled>Add to Cart</button>

    <div id="confirmation-msg">Product added.</div>

    <script>
      // Simulate loading state
      setTimeout(() => {
        document.getElementById('spinner').style.display = 'none';
        document.getElementById('add-button').disabled = false;
      }, 2000);

      // Show confirmation when button is clicked
      document.getElementById('add-button').addEventListener('click', () => {
        document.getElementById('confirmation-msg').style.display = 'block';
      });
    </script>
  </body>
</html>
```

#### Copy and Paste the following into **example_3_login.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Login Practice</title>
    <style>
      body {
        font-family: sans-serif;
        padding: 2rem;
      }
      .box {
        width: 300px;
        margin: auto;
      }
      input,
      button {
        display: block;
        margin: 0.5rem 0;
        padding: 0.5rem;
      }
      .message {
        color: green;
        margin-top: 1rem;
      }
    </style>
  </head>
  <body>
    <div class="box">
      <h2>Login</h2>
      <form id="login-form">
        <input id="username" placeholder="Username" required />
        <input id="password" type="password" placeholder="Password" required />
        <button id="submit" type="submit">Log In</button>
      </form>
      <div id="message" class="message" style="display: none">
        Welcome back!
      </div>
    </div>

    <script>
      document
        .getElementById('login-form')
        .addEventListener('submit', function (e) {
          e.preventDefault();
          const u = document.getElementById('username').value.trim();
          const p = document.getElementById('password').value.trim();
          if (u === 'learner2024' && p === 'securetest!') {
            document.getElementById('message').style.display = 'block';
          } else {
            alert('Invalid credentials.');
          }
        });
    </script>
  </body>
</html>
```

Once these files are created, open them in your browser to preview the UI behavior—they’ll be used in upcoming activities throughout this module!
