# VexNet — editable source (no build step)

This is the **same site you already have live**, but as plain, readable source:
HTML + CSS + JavaScript + JSON-style data files. Nothing is minified, compiled
or locked. Open a file, change a line, refresh the browser — that's the whole
workflow.

```
vexnet-src/
├── index.html          home page (hero, features, videos, payments, categories, shop, reviews, FAQ)
├── free.html           free cheats page
├── product.html        product detail page (opened as product.html?id=14)
├── checkout.html       cart / checkout page
│
├── css/
│   ├── app.css         base + layout styles
│   ├── site.css        design system (cards, drawer, popups, marquee, FAQ…)
│   ├── ui.css          utility classes used by the markup
│   └── custom.css      ← put your own tweaks here
│
├── js/
│   ├── core.js         language, currency, cart, price formatting
│   ├── templates.js    header + footer markup (shared by every page)
│   ├── layout.js       navigation, cart drawer, popups, FAQ, video player
│   ├── shop.js         product grid, filters, search
│   └── pages.js        free page, product page, checkout page
│
├── data/
│   ├── products.js     ← every product: name, price, plans, image, badge, description
│   ├── free-items.js   ← the 11 free downloads
│   ├── i18n.js         ← every visible string, Arabic + English
│   └── site.js         ← categories, currencies (rates), payment logos
│
└── img/                all images (products, categories, games, avatars, payment logos)
```

## 1. Running it

Double-click `index.html` — it works straight from the folder.

If your browser is strict about local files, run a tiny local server instead:

```bash
cd vexnet-src
python3 -m http.server 8080
# open http://localhost:8080
```

## 2. Editing products and prices

Open `data/products.js`. One product looks like this:

```js
{
  "id": 14,
  "name": "FiveM UnBan",
  "nameAr": "فك باند فايف ام",
  "game": "FiveM",
  "gameAr": "فايف ام",
  "cat": "unban",              // "unban" or "services" (the shop filter chips)
  "price": 15,                 // base price in USD
  "img": "img/products/820c201469dcbe44.webp",
  "badge": { "ar": "الأكثر طلباً", "en": "Most Wanted" },
  "descAr": "…",
  "descEn": "…",
  "plans": [
    { "v": "once", "ar": "مرة واحدة", "en": "One Time", "price": 15 },
    { "v": "week", "ar": "أسبوع", "en": "Week", "price": 15 }
  ]
}
```

* change a number → the price changes everywhere (grid, product page, cart, checkout)
* delete an object → the product disappears from the grid
* copy a whole block and give it a new `id` → a new product appears
* `plans: null` → the product has no duration options, just one price
* `custom: true` → shows “custom order” instead of a price

Free downloads live in `data/free-items.js` (title, description, image,
download `link`, password note in `badge`, author name/avatar, `category`).

## 3. Editing text (Arabic / English)

All wording is in `data/i18n.js`, one key per string:

```js
"hero.t1": { ar: "الخدمة الأكثر موثوقة", en: "The Most Reliable" },
```

The HTML uses the key (`data-i18n="hero.t1"`), so changing the value updates
the whole site in both languages. Language switching stays automatic.

## 4. Currency

`data/site.js` → `currencies`. `rate` is “how many units per 1 USD”:

```js
"SAR": { rate: 3.75, decimals: 2, symbol: "ر.س", label: "SAR" }
```

Add a currency by adding a new line; the dropdown picks it up if you also add a
button with `data-currency="AED"` in `js/templates.js` (`#currency-menu`).
Rates are static — edit them by hand, or fetch them from an API once a day.

## 5. Colours and styling

The look is a dark red/black theme. The red accent is `red-500` / `red-600`
in the markup plus a few `#ef4444` / `#dc2626` values in `css/site.css`.
Search-and-replace `from-red-600 to-red-500` and `bg-red-500` to change the
brand colour, and put any extra rules in `css/custom.css`.

## 6. Cart, product page, checkout, free page

Everything works out of the box, no backend needed:

* **cart** — saved in the browser (`localStorage`), badge in the header, drawer with quantities
* **product page** — `product.html?id=14`, duration plans, quantity, live total
* **checkout** — contact fields, payment method choice, order summary

Because there is no server, an order is **not** sent anywhere yet: it is logged
in the browser console. To send it somewhere real, put one line in the page
before the scripts (Discord webhook, your own API, Google Sheet…):

```html
<script>window.VEX_ORDER_ENDPOINT = "https://your-endpoint.example/order";</script>
```

The same works for the “join the team” form:

```html
<script>window.VEX_APPLY_ENDPOINT = "https://your-endpoint.example/apply";</script>
```

Both send JSON via `POST`.

Sign-in (`js/layout.js` → `buildAuth`) is interface only: Discord/Google
buttons and the email-code flow need your own OAuth apps / mail server.
Real payments likewise need your merchant or crypto gateway credentials.

## 7. Putting it online

It is a static site — upload the whole folder anywhere:

* **Netlify / Vercel / Cloudflare Pages** — drag the folder in, done
* **cPanel / any hosting** — copy the folder into `public_html`
* **your own server** — put it in `/var/www/yoursite` and point nginx at it

The only external dependency is the Google Fonts link (Cairo) in each page's
`<head>`; for a fully offline copy, download the font and host it yourself.

## 8. Adding a page

Copy `free.html`, change the `data-page` attribute, add a container
(`<div id="my-page"></div>`) and render into it from `js/pages.js`. The header,
footer, cart and popups come from `js/layout.js`, so they appear automatically.
