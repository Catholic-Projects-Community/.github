# Theming MPNext Tools for Your Diocese

MPNext Tools can wear your organization's colors and fonts through a single stylesheet, chosen
per MP database with the Configuration Setting **`<your Application Code>` / `Project_CSS`**.
This guide explains how to make one, whether you are editing it yourself or asking an AI
assistant to do it.

> **About the code.** These instructions are for **MPNext-Tools-SF**, a version of
> [MPNext-Tools](https://github.com/Catholic-Projects-Community/MPNext-Tools) made for Catholic
> dioceses. Its code repository is private for now; file paths below (like
> `src/lib/...`) refer to that code. Ask the maintainers listed on the
> [Catholic Projects Community](https://github.com/Catholic-Projects-Community) page about access.

The fork owner's databases (SFCatholic) keep the built-in look and ignore `Project_CSS`.

---

## 1. Start from the example

`public/themes/sfcatholic-example.css` in the MPNext-Tools-SF code is a complete
theme that reproduces the app's built-in sfcatholic.org look. Every variable the app uses is in
it, grouped and commented:

1. **Brand colors**: your palette (navy, blue, gold, grays)
2. **Page and text**: backgrounds, text, headings, cards, menus
3. **Buttons, links and states**: main and secondary buttons, quiet text, borders, focus ring,
   corner rounding
4. **Status colors**: success, warning, information
5. **Charts and sidebar**: rarely used today
6. **Dark mode**: the colors used when someone chooses Dark (or Match device) under
   the **Appearance** button (sun/moon icon) in each tool's title bar or on the dashboard

Loading the example unchanged changes nothing. It is the template.

## 2. Make your theme

1. Copy the example to `public/themes/<name>.css`, using letters, digits and hyphens only, e.g.
   `public/themes/archomaha.css`.
2. Change the **values** to your brand. Keep every **variable name** exactly as it is, and keep
   the `html:root` selectors (they make your values win over the app's defaults).
3. Check contrast (section 4).
4. Commit the file to your fork so it deploys with the app.

## 3. Turn it on

In MP, under **System Setup → Configuration Settings**, add (or edit):

| Application Code | Key Name | Value |
|---|---|---|
| *your code*<br>Default: `CP_NEXT` | `Project_CSS` | `archomaha` |

Two kinds of value are accepted:

- **A theme kept in the app**: the file name without `.css`, e.g. `archomaha` loads
  `/themes/archomaha.css`. Recommended: the theme is versioned and reviewed with the code, and
  nothing needs to be hosted elsewhere.
- **A stylesheet on your own website**: a full `https://` address, e.g.
  `https://www.example.org/mpnext/theme.css`. Use this to manage the theme outside the code.
  The address must use `https://` and a real web host name (not an IP address or `localhost`).
  A stylesheet can't run code, but it can change what every page looks like, so host it where
  only your staff can edit it.

The app checks the setting at most every **5 minutes**, so a change appears within that time.
Each MP database has its own setting, so sandbox and production can use different themes.

## 4. Rules that keep the app readable

These follow the WCAG AA accessibility standard:

- **Text needs contrast:** at least **4.5:1** against its background for normal text, **3:1**
  for large text (24px and up, or 19px and up in bold).
- **Check every pair.** Each `…-foreground` variable is the text color used *on* its partner:
  `--primary-foreground` is the text on a `--primary` button, `--muted-foreground` is quiet text
  on `--muted`, and so on. Change one, re-check the pair.
- **Bright accents are for decoration.** Colors like gold usually fail as text on white. Use them
  for backgrounds, underlines and highlights.
- **Keep the focus ring visible** (`--ring`). Keyboard users depend on it.
- **Dark brand colors carry white text.** `--color-brand-navy` is used for active tabs with white
  text on it, so it must be dark.

A free checker: <https://webaim.org/resources/contrastchecker/>.

## 5. Fonts

Add an `@import` for your web fonts as the **very first line** of the theme, then set the two
font variables inside the `html:root` block:

```css
@import url("https://fonts.googleapis.com/css2?family=Merriweather:wght@400;700&family=Open+Sans:wght@400;500;600;700&display=swap");

html:root {
  --font-eb-garamond: "Merriweather", Georgia, serif;   /* headings */
  --font-poppins: "Open Sans", system-ui, sans-serif;   /* body text and buttons */
  /* ...colors... */
}
```

The variable names are the app's built-in font names. They set the heading and body fonts
whatever font you put in them.

## 6. Asking an AI assistant to make a theme

Give the assistant your brand guide (colors with hex values, fonts, any rules) plus the example
file, and ask something like:

> Using `public/themes/sfcatholic-example.css` as the template, create
> `public/themes/<name>.css` for our brand: *[colors, fonts and rules]*. Keep every variable name
> and the `html:root` / dark-mode selectors exactly as in the template, and change only values.
> Make every text/background pair meet WCAG AA (4.5:1, or 3:1 for large text), state the
> contrast ratio in each comment you change, and use decorative-only colors (like gold) only for
> backgrounds. Put any font `@import` on the first line.

Then review the result (sections 4–5) and test it (section 7).

## 7. Test before production

1. Commit the theme file and deploy it to your **sandbox** site.
2. In the **sandbox** MP database, set `Project_CSS` to the theme's name.
3. Open a few tools and check headings, tabs, buttons, warnings and the printed Diocese
   Directory. Within 5 minutes the theme applies.
4. When it looks right, set the same `Project_CSS` value in production.

To remove a theme, delete the `Project_CSS` row (or clear its Value).

## Dark mode

Each person chooses **Light**, **Dark** or **Match device** with the **Appearance** button (sun/moon
icon) in each tool's title bar and on the dashboard.
The choice is remembered in that browser. Your theme's dark-mode block sets the colors used
then; leave it out to use the app's own dark colors. Printing and PDFs always use light colors.
The fork owner's databases (SFCatholic) don't offer the choice and stay light.

## What a theme doesn't change (yet)

- **The Diocese Directory PDF** uses its own color list (`src/app/(web)/tools/diocesedirectory/pdf/brand-colors.ts`).
- **The seal or logo:** set with `Directory_Logo_Url` or the diocese record's attached image (see
  [setup-for-your-diocese.md](setup-for-your-diocese.md)).
- **The browser tab icon and mobile browser bar color** are built into the app.
