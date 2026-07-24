# JavaScript DOM — Complete Reference Guide

The DOM (Document Object Model) is the tree-like representation of your HTML page that JS can read and manipulate. Below are the important DOM methods/properties grouped by category, with use cases.

---

## 1. Selecting Elements

| Method | Returns | Use case |
|---|---|---|
| `document.getElementById('id')` | Single element or `null` | Fastest way to grab one element by its unique ID. Use when you control the HTML and can add IDs. |
| `document.getElementsByClassName('cls')` | Live `HTMLCollection` | Grab all elements with a class. "Live" means it auto-updates if the DOM changes. |
| `document.getElementsByTagName('div')` | Live `HTMLCollection` | Grab all elements of a tag type, e.g. all `<li>`. |
| `document.querySelector('css-selector')` | First matching element or `null` | Use any CSS selector (`.class`, `#id`, `div > p`, `[data-x]`). Most flexible, use for one-off selections. |
| `document.querySelectorAll('css-selector')` | Static `NodeList` | Same as above but grabs all matches. Static means it does NOT auto-update. |

**When to use which:** `querySelector`/`querySelectorAll` for anything involving complex CSS selectors or when you think in CSS terms. `getElementById` when you just need one specific element and want max performance. `getElementsByClassName`/`TagName` are less common now but useful when you need a *live* collection that reflects DOM changes automatically.

```js
const btn = document.querySelector('.submit-btn');
const allItems = document.querySelectorAll('li.active');
```

---

## 2. Traversing the DOM (moving between related elements)

| Property | Use case |
|---|---|
| `element.parentElement` | Get the direct parent element. Use to walk upward. |
| `element.children` | Get only element children (skips text/comment nodes) — most commonly used. |
| `element.childNodes` | Get ALL child nodes including text/whitespace/comments — rarely needed. |
| `element.firstElementChild` / `lastElementChild` | Grab first/last child element, e.g. first `<li>` in a list. |
| `element.nextElementSibling` / `previousElementSibling` | Move to the next/previous sibling element — useful in loops or click handlers on lists. |
| `element.closest('selector')` | Walk UP the tree and find nearest ancestor matching a selector. Extremely useful in event delegation. |

```js
const li = document.querySelector('.active');
li.nextElementSibling.classList.add('highlight');

// closest() example — event delegation
button.addEventListener('click', e => {
  const row = e.target.closest('.table-row');
});
```

---

## 3. Reading/Changing Content

| Property | Use case |
|---|---|
| `element.textContent` | Get/set plain text (fast, safe from XSS, ignores HTML tags). **Preferred** for inserting user-generated text. |
| `element.innerText` | Similar but respects CSS (won't return hidden text), slower (triggers reflow). Use rarely. |
| `element.innerHTML` | Get/set HTML markup inside an element. Powerful but **dangerous** with untrusted input (XSS risk) — sanitize first. |
| `element.outerHTML` | Like innerHTML but includes the element itself, not just its contents. |

```js
title.textContent = 'Hello World';       // safe text
card.innerHTML = `<strong>${name}</strong>`; // only if name is trusted/sanitized
```

---

## 4. Creating, Inserting, Removing Elements

| Method | Use case |
|---|---|
| `document.createElement('div')` | Create a new element in memory (not yet in the page). |
| `parent.appendChild(node)` | Add a node as the last child. Classic way to insert elements. |
| `parent.append(node1, node2, 'text')` | Modern version of appendChild — accepts multiple nodes AND raw strings. |
| `parent.prepend(node)` | Insert as the FIRST child. |
| `parent.insertBefore(newNode, referenceNode)` | Insert before a specific existing child — for precise ordering. |
| `element.before(node)` / `element.after(node)` | Insert a sibling right before/after this element. |
| `element.remove()` | Remove the element from the DOM directly (modern, simplest). |
| `parent.removeChild(child)` | Older way to remove — needs reference to the parent. |
| `parent.replaceChild(newNode, oldNode)` | Swap one element for another. |
| `node.cloneNode(true)` | Duplicate a node. `true` = deep clone (includes children), `false` = shallow. Useful for templating repeated UI (e.g. list items, cards). |

```js
const li = document.createElement('li');
li.textContent = 'New item';
list.appendChild(li);

// Removing
document.querySelector('.toast').remove();

// Cloning a template row
const newRow = document.querySelector('.template-row').cloneNode(true);
table.appendChild(newRow);
```

---

## 5. Attributes & Data

| Method/Property | Use case |
|---|---|
| `element.getAttribute('name')` | Read any HTML attribute value (e.g. `src`, `href`, custom attrs). |
| `element.setAttribute('name', 'value')` | Set/update an attribute. |
| `element.removeAttribute('name')` | Remove an attribute entirely. |
| `element.hasAttribute('name')` | Check existence — returns boolean. |
| `element.dataset` | Read/write `data-*` attributes as a JS object. Best way to store custom metadata on elements (e.g. `data-user-id="42"` → `element.dataset.userId`). |
| `element.id`, `element.className`, `element.src`, `element.href`, etc. | Direct property access for common attributes — usually faster/simpler than getAttribute for standard ones. |

```js
button.setAttribute('disabled', 'true');
console.log(card.dataset.userId); // reads data-user-id="..."
card.dataset.status = 'active';   // writes data-status="active"
```

---

## 6. Classes & Styling

| Method | Use case |
|---|---|
| `element.classList.add('cls')` | Add a class — preferred way to trigger CSS-based styling changes. |
| `element.classList.remove('cls')` | Remove a class. |
| `element.classList.toggle('cls')` | Add if absent, remove if present — perfect for toggle buttons, dark mode switches, dropdowns. |
| `element.classList.contains('cls')` | Check if a class exists — useful for conditional logic. |
| `element.classList.replace('old', 'new')` | Swap one class for another. |
| `element.style.propertyName = 'value'` | Set inline CSS directly (e.g. `el.style.display = 'none'`). Use sparingly — prefer toggling classes for maintainability. |
| `getComputedStyle(element)` | Read the actual rendered CSS values (including from stylesheets), not just inline styles. |

```js
modal.classList.toggle('open');
if (input.classList.contains('error')) { /* ... */ }
box.style.backgroundColor = 'red'; // direct inline style (use for dynamic/computed values only)
```

---

## 7. Events

| Method | Use case |
|---|---|
| `element.addEventListener('click', handler)` | The standard way to listen for events (click, input, submit, keydown, etc.). Supports multiple listeners on the same element. |
| `element.removeEventListener('click', handler)` | Remove a previously attached listener (handler must be a named/reference function, not inline). |
| `event.preventDefault()` | Stop default browser behavior (e.g. stop a form from submitting/reloading the page). |
| `event.stopPropagation()` | Stop the event from bubbling up to parent elements. |
| `event.target` | The actual element that triggered the event — key for **event delegation**. |
| `event.currentTarget` | The element the listener is attached to (may differ from `target` when delegating). |

**Event delegation** (important pattern): instead of attaching listeners to every list item, attach ONE listener to the parent and check `event.target`. Much more efficient for dynamic lists.

```js
list.addEventListener('click', e => {
  const item = e.target.closest('li');
  if (item) console.log('Clicked:', item.textContent);
});

form.addEventListener('submit', e => {
  e.preventDefault();
  // handle form data manually
});
```

Common events to know: `click`, `input`, `change`, `submit`, `keydown`/`keyup`, `focus`/`blur`, `mouseenter`/`mouseleave`, `DOMContentLoaded` (fires when HTML is fully parsed, before images/styles finish — best place to run setup JS), `load` (fires when the ENTIRE page including assets is loaded).

```js
document.addEventListener('DOMContentLoaded', () => {
  // safe to query/manipulate DOM here
});
```

---

## 8. Forms & Inputs

| Property | Use case |
|---|---|
| `input.value` | Get/set text field content. |
| `checkbox.checked` | Get/set boolean state of checkboxes/radios. |
| `select.value` / `select.selectedIndex` | Get selected dropdown option. |
| `element.focus()` / `element.blur()` | Programmatically focus/unfocus an input — useful for UX (e.g. auto-focus first field, focus on validation error). |
| `form.reset()` | Clear all fields in a form. |
| `element.disabled = true/false` | Enable/disable a form control. |

---

## 9. Node Relationships Quick Cheat-Sheet

- `document` → the whole page
- `document.documentElement` → the `<html>` tag
- `document.body` → the `<body>` tag
- `window` → the browser tab itself (not part of DOM technically, but closely related — handles things like `window.innerWidth`, `window.location`, `window.scrollTo()`)

---

## Suggested Learning Order

1. **Selecting** → `querySelector`/`querySelectorAll` (cover 90% of cases)
2. **Reading/writing content** → `textContent`, `innerHTML`
3. **Events** → `addEventListener`, `event.target`, `preventDefault`
4. **Classes/styles** → `classList`
5. **Creating/removing elements** → `createElement`, `append`, `remove`
6. **Traversal & delegation** → `closest`, `parentElement`, event delegation
7. **Attributes & dataset** → for custom data-driven UI

### Practice project idea
Build a **To-Do list**: it forces you to use selecting, creating elements, appending, removing, event listeners, classList (for marking complete), and dataset (for storing item IDs) — basically everything above in one small app.