Great! Here's a simple, clean **example project** that shows how to make your website dynamic by reusing shared HTML parts like a header and footer using just **vanilla HTML + JavaScript (with `fetch()`)** — perfect for static sites.

---

## 📁 Project Structure

```
/my-site
├── index.html
├── blog.html
├── header.html
├── footer.html
└── /assets
    └── style.css
```

---

### ✅ 1. `header.html`

```html
<!-- header.html -->
<header>
  <h1>🚀 My Cool Site</h1>
  <nav>
    <a href="index.html">Home</a>
    <a href="blog.html">Blog</a>
  </nav>
  <hr />
</header>
```

---

### ✅ 2. `footer.html`

```html
<!-- footer.html -->
<hr />
<footer>
  <p>&copy; 2025 My Cool Site</p>
</footer>
```

---

### ✅ 3. `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Home - My Site</title>
  <link rel="stylesheet" href="assets/style.css" />
</head>
<body>

  <div id="header"></div>

  <main>
    <h2>Welcome to the homepage!</h2>
    <p>This is the home page content.</p>
  </main>

  <div id="footer"></div>

  <script src="assets/include.js"></script>
</body>
</html>
```

---

### ✅ 4. `blog.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Blog - My Site</title>
  <link rel="stylesheet" href="assets/style.css" />
</head>
<body>

  <div id="header"></div>

  <main>
    <h2>Blog Posts</h2>
    <p>Here are my articles...</p>
  </main>

  <div id="footer"></div>

  <script src="assets/include.js"></script>
</body>
</html>
```

---

### ✅ 5. `assets/include.js` – Inject shared parts

```js
// include.js
function includeHTML(id, file) {
  fetch(file)
    .then(res => res.text())
    .then(data => {
      document.getElementById(id).innerHTML = data;
    })
    .catch(err => console.error(`Error loading ${file}:`, err));
}

includeHTML("header", "header.html");
includeHTML("footer", "footer.html");
```

---

### ✅ 6. `assets/style.css` (optional)

```css
body {
  font-family: sans-serif;
  padding: 20px;
  background: #111;
  color: #eee;
}

nav a {
  margin-right: 1rem;
  color: #00ffe7;
  text-decoration: none;
}
```

---

## 🚀 How to Use

1. Put all files in a folder (e.g. `my-site`).
2. Open `index.html` in a browser **via a local server** (due to CORS restrictions with `fetch`).

> 🧪 You can serve it locally using:

* **VS Code + Live Server extension**
* Or run this in terminal:

  ```bash
  npx serve
  ```

---

## ✅ Done!

You now have:

* Shared `header.html` and `footer.html`
* No more copy-pasting across pages
* A dynamic-feeling static site!

Would you like me to zip this up for download or turn it into a GitHub-ready repo structure?
