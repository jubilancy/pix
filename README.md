# Asset Gallery

A static, paginated gallery for browsing and bulk-downloading hosted image assets (SVG, PNG, GIF, icons, etc.) by category tab. No build step, deployed via GitHub Pages.

## Structure

```
repo/
├── index.html              # gallery UI, tabs, search, pagination, zip download
├── style.css                # minimalist theme
├── .github/workflows/deploy.yml
├── gif/gif.txt
├── svg/svg.txt
├── png/png.txt
└── icon/icon.txt
```

Each `.txt` file holds one asset per line:

```
:cbi/asus-new:|https://files.svgcdn.io/cbi/asus-new.svg
:cbi/aston-martin:|https://files.svgcdn.io/cbi/aston-martin.svg
```

Format is `name|url`. The name is shown as-is and used as the copy/filename text, so it can contain colons or slashes if that's your convention — the split happens on the first `|` only.

## Adding a new category (e.g. `webp` or `cute`)

Three steps — folder, data file, and two lines of code.

**1. Create the folder and text file**

```
mkdir webp
touch webp/webp.txt
```

Fill `webp/webp.txt` with `name|url` lines, same format as the others.

**2. Register the type in `index.html`**

Find this line near the top of the `<script>` block:

```js
const TYPES = ['gif', 'svg', 'png', 'icon'];
```

Add your new type:

```js
const TYPES = ['gif', 'svg', 'png', 'icon', 'webp'];
```

**3. Add a tab button**

In the `<div class="tabs" id="tabs">` block, add a button matching the pattern of the others:

```html
<button class="tab-btn" data-type="webp" onclick="switchType('webp')">Webp</button>
```

The `data-type` and the argument to `switchType()` must exactly match the folder name and the string you added to `TYPES` — that's what the fetch path (`{type}/{type}.txt`) is built from.

**4. Update `deploy.yml`**

Add the new type to the `paths` trigger list and to both `for type in ...` loops:

```yaml
paths:
  - 'gif/gif.txt'
  - 'svg/svg.txt'
  - 'png/png.txt'
  - 'icon/icon.txt'
  - 'webp/webp.txt'      # add this
  - 'index.html'
  - 'style.css'
  - '.github/workflows/deploy.yml'
```

```yaml
- name: Validate txt files format
  run: |
    for type in gif svg png icon webp; do   # add webp here
      ...

- name: Build site folder
  run: |
    ...
    for type in gif svg png icon webp; do   # and here
      ...
```

That's it — push, and the new tab appears with pagination, search, and zip download working the same as the built-in categories.

## Notes

- Only the active tab's `.txt` is fetched, so category size doesn't affect load time for other tabs.
- Switching tabs resets search, page, and selection. Paging within a tab does not.
- The URL stays in sync (`?type=webp&per=48&page=2&q=cat`), so links are shareable and reload-safe.
- Zip downloads use the current tab's file extension as a fallback if a URL has none.
