To add an **autocomplete feature** to your search bar, you have a few good options depending on how advanced you want it. Here's a **simple vanilla JavaScript solution** using your existing setup.

---

## ✅ Step-by-Step: Basic Autocomplete

### 1. 🏷️ Prepare Your Data

Let’s say your wallpapers have names or tags you want to suggest.

You can extract them from `.wallpaper` elements dynamically, or use a predefined array:

```javascript
// Example: Collect names from dataset
const items = document.querySelectorAll(".wallpaper");
const suggestions = Array.from(items).map(item => item.dataset.name.toLowerCase());
```

Or use a static list:

```javascript
const suggestions = ["mountain", "ocean", "cityscape", "sunset", "forest"];
```

---

### 2. ➕ Add HTML for the Suggestion Box

Below your search input:

```html
<input type="text" id="search" placeholder="Search wallpapers...">
<ul id="autocomplete-list" class="autocomplete-list"></ul>
```

Add styles:

```css
.autocomplete-list {
    position: absolute;
    background: white;
    border: 1px solid #ccc;
    list-style: none;
    padding: 0;
    margin: 0;
    max-height: 200px;
    overflow-y: auto;
    z-index: 1000;
}

.autocomplete-list li {
    padding: 8px;
    cursor: pointer;
}

.autocomplete-list li:hover {
    background-color: #eee;
}
```

---

### 3. ✨ Add the JavaScript Logic

```javascript
const searchInput = document.getElementById("search");
const autocompleteList = document.getElementById("autocomplete-list");

searchInput.addEventListener("input", () => {
    const query = searchInput.value.toLowerCase();
    autocompleteList.innerHTML = ""; // Clear previous suggestions

    if (!query) return;

    const matches = suggestions.filter(item => item.includes(query)).slice(0, 5); // max 5 suggestions

    matches.forEach(match => {
        const li = document.createElement("li");
        li.textContent = match;
        li.addEventListener("click", () => {
            searchInput.value = match;
            autocompleteList.innerHTML = "";
            applyFilters();
            searchInput.blur(); // optional: blur after selection
        });
        autocompleteList.appendChild(li);
    });
});

// Hide suggestions when clicking outside
document.addEventListener("click", (e) => {
    if (!autocompleteList.contains(e.target) && e.target !== searchInput) {
        autocompleteList.innerHTML = "";
    }
});
```

---

Dynamic server search (like Google)? That’s possible too with a fetch call.

Great! Here's how to build **dynamic autocomplete (like Google)** using a **`fetch` call to a server or API** — useful when suggestions come from a backend (e.g., a large database, user history, etc.).

---

## ✅ Dynamic Autocomplete with `fetch`

### 🧱 Assumptions:

* You have a backend API like:
  `GET /api/suggestions?q=mount`

  * Returns: `["mountain", "mount fuji", "mountains at sunset"]`
* You want real-time suggestions as the user types.

---

### 1. 🔧 Frontend HTML (same as before)

```html
<input type="text" id="search" placeholder="Search wallpapers...">
<ul id="autocomplete-list" class="autocomplete-list"></ul>
```

---

### 2. 💻 JavaScript with `fetch`

```javascript
const searchInput = document.getElementById("search");
const autocompleteList = document.getElementById("autocomplete-list");

let debounceTimeout;

searchInput.addEventListener("input", () => {
    const query = searchInput.value.trim();

    clearTimeout(debounceTimeout);
    
    if (!query) {
        autocompleteList.innerHTML = "";
        return;
    }

    // Debounce to limit server calls
    debounceTimeout = setTimeout(() => {
        fetch(`/api/suggestions?q=${encodeURIComponent(query)}`)
            .then(res => res.json())
            .then(suggestions => {
                autocompleteList.innerHTML = "";

                suggestions.slice(0, 5).forEach(suggestion => {
                    const li = document.createElement("li");
                    li.textContent = suggestion;
                    li.addEventListener("click", () => {
                        searchInput.value = suggestion;
                        autocompleteList.innerHTML = "";
                        applyFilters();
                        searchInput.blur();
                    });
                    autocompleteList.appendChild(li);
                });
            })
            .catch(err => {
                console.error("Autocomplete fetch error:", err);
                autocompleteList.innerHTML = "";
            });
    }, 300); // Adjust delay for performance (300ms is good)
});
```

---

### 3. 📡 Example Backend (Node.js/Express)

```javascript
app.get("/api/suggestions", (req, res) => {
    const q = req.query.q.toLowerCase();
    const allSuggestions = ["mountain", "sunset", "beach", "desert", "ocean", "forest", "mount fuji"];
    const filtered = allSuggestions.filter(s => s.toLowerCase().includes(q));
    res.json(filtered);
});
```

---

### ✅ Optional Enhancements:

* **Arrow key navigation** in the dropdown
* **Loading spinner** while waiting for results
