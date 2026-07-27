# HTML — A Backend Developer's Practical Reference

You already think in terms of structure, data, and logic — HTML is just the structural layer of a webpage. This guide skips "what is a tag" basics and focuses on what you actually need to build/understand real pages, plus **how to think about structuring a page** (the part that trips up backend devs most).

---

## 1. Document Skeleton

Every page starts the same way. Know what each line does — you'll see it everywhere.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Page</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <!-- visible content goes here -->
  <script src="app.js"></script>
</body>
</html>
```

- `<!DOCTYPE html>` — tells the browser to use modern HTML5 rendering rules (no quirks mode).
- `<meta charset="UTF-8">` — character encoding, prevents broken symbols/emoji.
- `<meta name="viewport">` — makes the page responsive on mobile. **Always include this.**
- `<script>` at the bottom of `<body>` (or with `defer`) — ensures HTML loads before JS runs and tries to manipulate it.

---

## 2. Semantic Structural Tags (this is the important part)

This is the #1 thing backend devs get wrong: wrapping *everything* in `<div>`. Modern HTML has semantic tags that describe the *purpose* of a section, not just a generic box. Browsers, screen readers, and SEO crawlers understand these — a `<div>` tells them nothing.

| Tag | Use case | Example |
|---|---|---|
| `<header>` | Top section of a page or a section — logo, nav, title | Site header with logo + nav menu |
| `<nav>` | Navigation links | Main menu, breadcrumb, sidebar links |
| `<main>` | The primary unique content of the page (one per page) | The actual blog post, dashboard content |
| `<section>` | A thematic grouping of content, usually with its own heading | "Features" section, "Pricing" section on a landing page |
| `<article>` | Self-contained, independently distributable content | A blog post, a news article, a single product card |
| `<aside>` | Tangential content, related but separate | Sidebar, "related articles", ads |
| `<footer>` | Bottom section — copyright, links, contact info | Site footer |
| `<div>` | Generic container with **no semantic meaning** | Use ONLY when no semantic tag fits — mainly for CSS/JS layout grouping |
| `<span>` | Generic **inline** container, no semantic meaning | Wrapping a word to style it differently within a sentence |

```html
<body>
  <header>
    <h1>My Blog</h1>
    <nav>
      <a href="/">Home</a>
      <a href="/about">About</a>
    </nav>
  </header>

  <main>
    <article>
      <h2>Post Title</h2>
      <p>Post content...</p>
    </article>

    <aside>
      <h3>Related Posts</h3>
    </aside>
  </main>

  <footer>
    <p>&copy; 2026 My Blog</p>
  </footer>
</body>
```

### How to decide "where to put a div" — the actual approach

When structuring any page, ask these questions **top-down**:

1. **What are the major regions of the page?** → header, main, footer. Sketch this first, literally on paper if it helps.
2. **Inside `main`, what are the distinct content groups?** → these become `<section>`s or `<article>`s (e.g., hero section, features section, testimonials section).
3. **Inside each section, is there a repeating unit?** (a card, a list item, a product) → that repeating unit is usually a `<div class="card">` or `<article>` if it's independently meaningful (like a blog post preview or product card).
4. **Only use `<div>` when:**
   - You need a wrapper purely for CSS layout (e.g., a flex/grid container) and no semantic tag fits.
   - You're grouping elements for JS purposes (e.g., a `<div id="modal">` you show/hide).
5. **Only use `<span>` when** you need to style/target a small piece of *inline* text (like highlighting one word in a paragraph) — never for block-level layout.

**Rule of thumb:** semantic tag first, `<div>` as the fallback — not the default. If you're about to type `<div>`, pause and ask "is this actually a section, article, nav, or aside?"

```html
<!-- Bad: everything is a div, no meaning -->
<div class="header">
  <div class="nav">...</div>
</div>
<div class="content">
  <div class="post">...</div>
</div>

<!-- Good: structure communicates meaning -->
<header>
  <nav>...</nav>
</header>
<main>
  <article>...</article>
</main>
```

A useful mental model: **`<div>` is your `Object` / generic dict — use it when nothing more specific exists. Semantic tags are your typed classes.**

---

## 3. Text & Content Tags

| Tag | Use case |
|---|---|
| `<h1>` – `<h6>` | Headings, in strict hierarchical order. Only ONE `<h1>` per page (usually the main title). Don't skip levels for styling — use CSS for that instead. |
| `<p>` | Paragraph of text. |
| `<a href="...">` | Link. `target="_blank"` opens in new tab (pair with `rel="noopener noreferrer"` for security). |
| `<ul>` / `<ol>` / `<li>` | Unordered/ordered lists — for navigation menus, feature lists, steps. |
| `<strong>` / `<em>` | Bold with semantic "importance" / italic with semantic "emphasis" (screen readers announce these differently than plain `<b>`/`<i>`). |
| `<img src="..." alt="...">` | Image. `alt` is NOT optional — it's used for accessibility and shows if the image fails to load. |
| `<br>` | Line break (rarely needed — usually CSS spacing is better). |
| `<hr>` | Thematic break / horizontal line. |

```html
<h1>Dashboard</h1>
<p>Welcome back, <strong>Subham</strong>.</p>
<ul>
  <li><a href="/orders">Orders</a></li>
  <li><a href="/settings">Settings</a></li>
</ul>
<img src="chart.png" alt="Monthly revenue chart">
```

---

## 4. Forms — critical for a backend dev (this is where you connect to your APIs)

```html
<form action="/api/login" method="POST">
  <label for="email">Email</label>
  <input type="email" id="email" name="email" required>

  <label for="password">Password</label>
  <input type="password" id="password" name="password" required minlength="8">

  <button type="submit">Log in</button>
</form>
```

| Element/Attribute | Use case |
|---|---|
| `<form action="/url" method="POST">` | `action` = endpoint the form submits to, `method` = HTTP verb (GET for search/filter, POST for creating/mutating data). |
| `<input type="...">` | `text`, `email`, `password`, `number`, `checkbox`, `radio`, `date`, `file`, `hidden` — browser gives built-in validation/UI per type. |
| `<label for="id">` | Links a label to an input by matching `for` to the input's `id` — clicking the label focuses the input. **Important for accessibility, don't skip it.** |
| `<select>` / `<option>` | Dropdown menu. |
| `<textarea>` | Multi-line text input. |
| `name="..."` attribute | **This is the key your backend receives** — e.g. `req.body.email` on the server maps to `name="email"` on the form field. |
| `required`, `minlength`, `pattern`, `min`/`max` | Client-side validation attributes — always mirror these with server-side validation too, never trust the client. |
| `<button type="submit">` vs `type="button"` | `submit` triggers form submission; `button` does nothing by default (use for JS-only actions inside a form, to avoid accidental submits). |

In modern apps you'll often intercept the submit with JS (`fetch`) instead of a full page reload — but understanding the plain `action`/`method`/`name` behavior is essential since that's what's happening under the hood, and it's still the default fallback.

---

## 5. Tables (for structured/tabular data — very backend-relatable)

```html
<table>
  <thead>
    <tr>
      <th>Order ID</th>
      <th>Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1001</td>
      <td>Shipped</td>
    </tr>
  </tbody>
</table>
```

Use tables **only for actual tabular data** (like a DB result set) — never for page layout (that's what CSS Grid/Flexbox are for).

---

## 6. Attributes You'll Use Constantly

| Attribute | Use case |
|---|---|
| `id="..."` | Unique identifier — used for CSS targeting, JS `getElementById`, and `<label for>` linking. One per page per value. |
| `class="..."` | Reusable style/JS hook — can repeat across many elements. |
| `data-*="..."` | Custom attributes to store extra data for JS to read (e.g. `data-user-id="42"`), doesn't affect rendering. |
| `href` / `src` | Target URL for links / resource path for images, scripts, stylesheets. |
| `alt` | Accessibility + fallback text for images. |
| `disabled` | Disables form controls/buttons. |
| `required` | Native form validation. |
| `target="_blank"` | Open link in a new tab. |
| `rel="noopener noreferrer"` | Security best practice when using `target="_blank"` — prevents the new tab from accessing `window.opener`. |

---

## 7. `<div>` Layout Patterns You'll Actually Use

Since you'll still use `<div>` heavily for layout (paired with CSS Flexbox/Grid), here's the practical pattern:

```html
<section class="pricing">
  <div class="pricing-grid">        <!-- layout wrapper: CSS grid container -->
    <div class="pricing-card">      <!-- repeating unit -->
      <h3>Basic</h3>
      <p>$9/mo</p>
    </div>
    <div class="pricing-card">
      <h3>Pro</h3>
      <p>$29/mo</p>
    </div>
  </div>
</section>
```

- Outer semantic tag (`<section>`) = "what this content IS"
- Inner `<div>` = "how it's laid out" (grid/flex container)
- Repeated `<div class="card">` = "one instance of a repeating thing"

This 3-layer pattern (semantic wrapper → layout div → repeating item div) covers most real-world page sections: pricing tables, product grids, dashboards, card lists.

---

## Suggested Learning Order (as a backend dev)

1. **Document skeleton** — you'll copy-paste this forever, just know what it means.
2. **Semantic tags** (`header`, `nav`, `main`, `section`, `article`, `footer`) — this alone makes your HTML dramatically better than "div soup."
3. **Forms** — since you're backend-focused, this is your actual interface to your APIs (`name`, `method`, `action`).
4. **Divs + layout thinking** — the 3-layer pattern above.
5. Tables, lists, text tags — pick up as needed while building.

### Practice project idea
Build a **simple dashboard page**: header with nav, a `main` with a `section` showing a table of data (e.g. orders) and a `section` with a form (e.g. "add new order"). This forces you to combine semantic structure, forms, and tables — the three things you'll touch most as a backend dev writing HTML.