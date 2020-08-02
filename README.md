This Tailwind CSS plugin registers variants for theming *without needing custom properties*. It has support for 
* Controlling themes with media queries
* Controlling themes with selectors (such as classes and data attributes)
* Responsive variants
* Extra stacked variants
* Falling back to a particular theme when none matches.

You are recommended to check out [the comparison table of all Tailwind CSS theming plugins below](#alternatives) before committing to any one. 

If you want your site to 

# Installation

```sh
npm install --save-dev tailwindcss-theme-variants
```

# Basic usage

## Using selectors to choose the active theme

With this Tailwind configuration,

```js
const { tailwindcssThemeVariants } = require("tailwindcss-theme-variants");

module.exports = {
    theme: {
        backgroundColor: {
            "gray-900": "#1A202C",
        },
    },

    variants: {
        backgroundColor: ["light", "dark"],
    },

    plugins: [
        tailwindcssThemeVariants({
            themes: {
                light: {
                    selector: ".light-theme",
                },
                dark: {
                    selector: ".dark-theme",
                },
            },
        }),
    ],
};
```

this CSS is generated:

```css
.bg-gray-900 {
    background-color: #1A202C;
}

/* If you're having trouble understanding,
   imagine it said html instead of :root,
   like in the example below */

:root.light-theme .light\:bg-gray-900 {
    background-color: #1A202C;
}

:root.dark-theme .dark\:bg-gray-900 {
    background-color: #1A202C;
}
```

After also enabling `"light"` and `"dark"` variants for `textColor` and bringing in more colors from the [default palette](https://tailwindcss.com/docs/customizing-colors/#default-color-palette), we can implement a simple themed button in HTML like this:

```html
<html class="light-theme"> <!-- Change to dark-theme -->
    <button class="light:bg-teal-200 dark:bg-teal-800 light:text-teal-700 dark:text-teal-100">
        Sign up
    </button>
</html>
```

This will result in dark text on a light background in the light theme, and light text on a dark background in the dark theme.

💡 You can choose more than just classes for your selectors. Other, good options include data attributes, like `[data-padding=compact]`. You *can* go as crazy as `.class[data-theme=light]:dir(rtl)`, for example, but at that point you need to be careful with specificity!


## Using media queries to choose the active theme

You may rather choose to tie your theme selection to matched media queries, like [`prefers-color-scheme`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme):

```js
const { tailwindcssThemeVariants, prefersLight, prefersDark } = require("tailwindcss-theme-variants");

module.exports = {
    theme: {
        backgroundColor: {
            "teal-500": "#38B2AC",
        },
    },

    variants: {
        backgroundColor: ["light", "dark"],
    },

    plugins: [
        tailwindcssThemeVariants({
            themes: {
                light: {
                    mediaQuery: prefersLight /* "@media (prefers-color-scheme: light)" */,
                },
                dark: {
                    mediaQuery: prefersDark /* "@media (prefers-color-scheme: dark)" */,
                },
            },
        }),
    ],
};
```

Which generates this CSS:

```css
.bg-teal-500 {
    background-color: #38B2AC
}

@media (prefers-color-scheme: light) {
    .light\:bg-teal-500 {
        background-color: #38B2AC;
    }
}

@media (prefers-color-scheme: dark) {
    .dark\:bg-teal-500 {
        background-color: #38B2AC;
    }
}
```

💡 Keep the `variants` listed in the same order as in `themes` in this plugin's configuration for consistency and the most expected behavior. In `backgroundColor`'s `variants`, `light` came first, then `dark`, so we also list `light` before `dark` in `tailwindcssThemeVariants`'s `themes` option. There is a planned feature that should make it unnecessary to remember this information, but until then, please follow this advice.

# Full configuration

This plugin expects configuration of the form

```ts
{
    themes: {
        [name: string]: {
            // At least one is required
            selector?: string;
            mediaQuery?: string;
        }
    };

    baseSelector?: string;
    fallback?: string | boolean;

    variants?: {
        [name: string]: (selector: string) => string;
    };
}
```

Where each parameter means:

- `themes`: an object mapping a theme name to the conditions that determine whether or not the theme will be active.

   - `selector`: a selector that has to be active on `baseSelector` for this theme to be active. For instance, if `baseSelector` is `html`, and `themes.light`'s `selector` is `.light-theme`, then the `light` theme's variant(s) will be in effect whenever `html` has the `light-theme` class on it.

   - `mediaQuery`: a media query that has to be active for this theme to be active. For instance, if the `reduced-motion` theme has `mediaQuery` `"@media (prefers-reduced-motion: reduce)"` (importable as `prefersReducedMotion`), then the `reduced-motion` variant(s) will be active whenever that media query matches: if the visitor's browser reports preferring reduced motion.


- `baseSelector` (default `""` if you **only** use media queries to activate your themes, otherwise `":root"`): the selector that each theme's `selector` will be applied to to determine the active theme.

- `fallback` (default `false`): chooses a theme to fall back to when none of the media queries or selectors are active. You can either manually select a theme by giving a string like `"solarized-dark"` or implicitly select the first one listed in `themes` by giving `true`. 

  ⚠️ Passing a string referring to a theme name is deprecated in favor of the `true`/`false` approach, and is planned to be removed.

- `variants` (default is nothing): an object mapping the name of a variant to a function that gives a selector for when that variant is active. 

  For example, the importable `even` variant takes a `selector` and returns `` `${selector}:nth-child(even)` ``. The importable `groupHover` (which you are recommended to name `"group-hover"` for consistency) variant returns `` `.group:hover ${selector}` ``


# Examples
💡 At the time of writing, this documentation is a work in progress. For all examples, where I've done my best to stretch the plugin to its limits (especially towards the end of the file), see the test suite in [`tests/index.ts`](https://github.com/SirNavith/tailwindcss-theme-variants/blob/master/tests/index.ts#L67).

## Fallback
### Media queries
With the same media-query-activated themes as [above](#using-media-queries-to-choose-the-active-theme),
```js
themes: {
    light: {
        mediaQuery: prefersLight /* "@media (prefers-color-scheme: light)" */,
    },
    dark: {
        mediaQuery: prefersDark /* "@media (prefers-color-scheme: dark)" */,
    },
},
```
we can create a table to show what the active theme will be under all possible conditions:

<table>
<tr align="center">
<th align="left">Matching media query</th>
<th>Neither</th>
<th><code>prefers-color-scheme: light</code></th>
<th><code>prefers-color-scheme: dark</code></th>
</tr>

<tr align="center">
<th align="left">Active theme</th>
<td>None</td>
<td><code>light</code></td>
<td><code>dark</code></td>
</tr>
</table>

**The whole point of the fallback feature is to address that *None* case.** It could mean that the visitor is using a browser that doesn't [support `prefers-color-scheme`](https://caniuse.com/#search=prefers-color-scheme), such as IE11. Instead of leaving them on an unthemed site, we can "push" them into a particular theme by specifying `fallback`.

```js
themes: {
    light: {
        mediaQuery: prefersLight /* "@media (prefers-color-scheme: light)" */,
    },
    dark: {
        mediaQuery: prefersDark /* "@media (prefers-color-scheme: dark)" */,
    },
},
// New addition
fallback: "light",
// Because light is the first theme in the list, `true` would've worked too
```

Which will change the generated CSS to activate `light` earlier than any media queries, so they can still override that declaration by being later in the file. **You could think of `light` as the *default theme*** in this case.
```css
.bg-teal-500 {
    background-color: #38B2AC;
}

/* New addition */
.light\:bg-teal-500 {
    background-color: #38B2AC;
}
/* End new addition */

@media (prefers-color-scheme: light) {
    .light\:bg-teal-500 {
        background-color: #38B2AC;
    }
}

@media (prefers-color-scheme: dark) {
    .dark\:bg-teal-500 {
        background-color: #38B2AC;
    }
}
```

Which, in turn, changes the active theme table to:
<table>
<tr align="center">
<th align="left">Matching media query</th>
<th>Neither</th>
<th><code>prefers-color-scheme: light</code></th>
<th><code>prefers-color-scheme: dark</code></th>
</tr>

<tr align="center">
<th align="left">Active theme</th>
<td><code>light</code></td>
<td><code>light</code></td>
<td><code>dark</code></td>
</tr>
</table>

💡 Even though `background-color` has been used in every example so far, theme variants are available for *any* utility. 

### Selectors
`fallback` also works for selector-activated themes.

💡 If you control themes on your site by adding / removing classes or attributes on the `html` or `body` element with JavaScript, then visitors without JavaScript enabled would see the `fallback` theme!

```js
themes: {
    light: {
        selector: ".light-theme",
    },
    dark: {
        selector: ".dark-theme",
    },
},
fallback: "dark",
```

This generates:
```css
.bg-gray-900 {
    background-color: #1A202C;
}

:root.light-theme .light\:bg-gray-900 {
    background-color: #1A202C;
}

:root:not(.light-theme) .dark\:bg-gray-900 {
    background-color: #1A202C;
}

:root.dark-theme .dark\:bg-gray-900 {
    background-color: #1A202C;
}
```

Which has the active theme table:
<table>
<tr>
<th align="left">Matching selector</th>
<th align="left">Active theme</th>
</tr>

<tr>
<th align="left">Neither</th>
<td align="center"><code>dark</code></td>
</tr>

<tr>
<th align="left"><code>:root.light-theme</code></th>
<td align="center"><code>light</code></td>
</tr>

<tr>
<th align="left"><code>:root.dark-theme</code></th>
<td align="center"><code>dark</code></td>
</tr>
</table>


## Stacked variants
💡 All of Tailwind CSS's core variants and more are bundled for use with this plugin. You can see the full list in [`src/variants.ts`](https://github.com/SirNavith/tailwindcss-theme-variants/blob/master/src/variants.ts).

By specifying `variants` in this plugin's options, you can "stack" extra variants on top of the existing theme variants. (We call it *stacking* because there are multiple variants required, like in `night:focus:border-white`, the border will only be white if the `night` theme is active **and** the element is `:focus`ed on).

Here's an example of combining [`prefers-contrast: high`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-contrast) (which you can import as `prefersHighContrast`) with the `:hover` variant (which you can import as `hover`):
```js
const { tailwindcssThemeVariants, hover, prefersHighContrast } = require("tailwindcss-theme-variants");

module.exports = {
    theme: {
        // Your Tailwind CSS theme configuration
    },

    variants: {
        backgroundColor: ["high-contrast"],
        textColor: ["high-contrast", "high-contrast:hover"],
    },

    plugins: [
        tailwindcssThemeVariants({
            themes: {
                "high-contrast": {
                    mediaQuery: prefersHighContrast /* "@media (prefers-contrast: high)" */,
                },
            },
            variants: {
                "hover": hover /* (selector) => `${selector}:hover` */,
            },
        }),
    ],
};
```

You could create a simple card that uses contrast pleasant for fully sighted visitors, but intelligently switches to functional high contrast for those who specify it:
```html
<div class="bg-gray-100 high-contrast:bg-white text-gray-800 high-contrast:text-black">
    <h1>Let me tell you all about...</h1>
    <h2>... this great idea I have!</h2>

    <a href="text-blue-500 high-contrast:text-blue-700 hover:text-blue-600 high-contrast:hover:text-blue-900">
        See more
    </a>
</div>
```

#### Writing a custom variant function
You might need to write a variant function yourself if it's not `export`ed with this plugin. 

TODO hocus explanation

```js
const { tailwindcssThemeVariants, hover, odd } = require("tailwindcss-theme-variants");

module.exports = {
    theme: {
        // Your Tailwind CSS theme configuration
    },

    variants: {
        opacity: [
            "transparency-safe",        "transparency-reduce",
            "transparency-safe:hocus",  "transparency-reduce:hocus",
        ],
    },

    plugins: [
        tailwindcssThemeVariants({
            themes: {
                "transparency-safe": { 
                    mediaQuery: prefersAnyTransparency /* "@media (prefers-reduced-transparency: no-preference)" */,
                },
                "transparency-reduce": { 
                    mediaQuery: prefersReducedTransparency /* "@media (prefers-reduced-transparency: reduce)" */,
                },
            },
            fallback: true, // prefers-reduced-transparency is not supported in any browsers yet
            variants: {
                // The custom variant function, written by you
                "hocus": (selector) => `${selector}:hover, ${selector}:focus`,
            },
        }),
    ],
};
```

TODO hocus HTML



Another—complex—example: suppose you want to zebra stripe your tables, matching the current theme, and change it on hover:

```js
const { tailwindcssThemeVariants, hover, odd } = require("tailwindcss-theme-variants");

module.exports = {
    theme: {
        // Your Tailwind CSS theme configuration
    },

    variants: {
        backgroundColor: [
            "no-accent",            "green-accent",            "orange-accent",
            "no-accent:hover",      "green-accent:hover",      "orange-accent:hover", 
            "no-accent:odd",        "green-accent:odd",        "orange-accent:odd", 
            "no-accent:odd:hover",  "green-accent:odd:hover",  "orange-accent:odd:hover",
        ],
    },

    plugins: [
        tailwindcssThemeVariants({
            baseSelector: "table.themed",
            themes: {
                "no-accent": { selector: "" },
                "green-accent": { selector: ".themed-green" },
                "orange-accent": { selector: ".themed-orange" },
            },
            variants: {
                hover /* (selector) => `${selector}:hover` */,
                odd /* (selector) => `${selector}:nth-child(odd)` */,

                // The custom variant function, written by you
                "odd:hover": (selector) => `${selector}:nth-child(odd):hover`,

                // By the way, the ordering here doesn't matter
                // (as opposed to the ordering of variants above)
            },
        }),
    ],
};
```

We can then implement the themeable table in HTML (Svelte) like so:

```html
<table class="themed themed-green"> <!-- Try changing themed-green to themed-orange or removing it -->
    {#each people as person}
        <tr class="no-accent:bg-white               green-accent:bg-green-50             orange-accent:bg-orange-50
                   no-accent:hover:bg-gray-100      green-accent:hover:bg-green-100      orange-accent:hover:bg-orange-100
                   no-accent:odd:bg-gray-100        green-accent:odd:bg-green-100        no-accent:orange-accent:odd:bg-orange-100
                   no-accent:odd:hover:bg-gray-200  green-accent:odd:hover:bg-green-200  orange-accent:odd:hover:bg-orange-100
                  ">

            <td>{person.firstName} {person.lastName}</td>
            <td>{person.responsibility}</td>
            <!-- ... -->
        </tr>
    {/each}
</table>
```



### Responsive variants
Responsive variants let you distinguish the current breakpoint per theme, letting you say `lg:green-theme:border-green-200` to have a `green-200` border only when the breakpoint is `lg` (or larger) and the `green-theme` is active, for instance.

⚠️ Responsive variants are automatically generated whenever `responsive` is listed in the utility's `variants` in the Tailwind CSS configuration, **not** this plugin's configuration. Also, because this feature is provided by Tailwind CSS rather than this plugin, you have to type `breakpoint:` **before** the `theme-name:` instead of after.

```js
const { tailwindcssThemeVariants } = require("tailwindcss-theme-variants");

module.exports = {
    theme: {
        // Your Tailwind CSS theme configuration
    },
    variants: {
        textColor: ["responsive", "day", "night"]
    },
    plugins: [
        tailwindcssThemeVariants({
            themes: {
                day: { selector: "[data-time=day]" },
                night: { selector: "[data-time=night]" },
            },
        }),
    ],
};
```

With this, we could make the landing page's title line change color at different screen sizes "within" each theme:
```html
<h1 class="day:text-black          night:text-white
           sm:day:text-orange-800  sm:night:text-yellow-100
           lg:day:text-orange-600  lg:night:text-yellow-300">
    
    The best thing that has ever happened. Ever.
</h1>
```


We could also make a group of themes for data density, like you can [configure in GMail](https://www.solveyourtech.com/switch-compact-view-gmail/):

```js
const { tailwindcssThemeVariants } = require("tailwindcss-theme-variants");

module.exports = {
    theme: {
        // Your Tailwind CSS theme configuration
    },
    variants: {
        padding: ["responsive", "comfortable", "compact"]
    },
    plugins: [
        tailwindcssThemeVariants({
            themes: {
                comfortable: { selector: "[data-density=comfortable]" },
                compact: { selector: "[data-density=compact]" },
            },
            // Fall back to the first theme listed (comfortable) when density is not configured
            fallback: true,
        }),
    ],
};
```

This will allow us to configure the padding for each theme for each breakpoint, of a list of emails in the inbox (so original!):
```html
<li class="comfortable:p-2     compact:p-0
           md:comfortable:p-4  md:compact:p-1
           xl:comfortable:p-6  xl:compact:p-2">
    
    FWD: FWD: The real truth behind...
</li>
```

#### Extra stacked variants
You can still stack extra variants even while using responsive variants.

TODO

## Using both selectors and media queries
⚠️ If you use both selectors and media queries to activate themes, then **make sure that each specified class is specified as an *all or nothing* approach**. For instance, if you have `winter` and `summer` themes and want to add the `winter:bg-teal-100` class, then you also need to add the `summer:bg-orange-300` class. If you don't do this, then it will look like the values from an theme that's *supposed to be* inactive are "leaking" into the active theme.

Every feature previously discussed still works as you'd expect when you decide to also add selectors or media queries (whichever you weren't using before) to theme control. TODO 

TODO: Show active theme tables for every example
Such as:
<table>
<tr>
<th align="left">Match</th>
<th align="left">Neither</th>
<th align="left"><code>prefers-color-scheme: light</code></th>
<th align="left"><code>prefers-color-scheme: dark</code></th>
</tr>

<tr>
<th align="left">Neither</th>
<td align="center">None</td>
<td align="center"><code>cyan</code></td>
<td align="center"><code>navy</code></td>
</tr>

<tr>
<th align="left"><code>:root.day</code></th>
<td align="center"><code>cyan</code></td>
<td align="center"><code>cyan</code></td>
<td align="center"><code>cyan</code></td>
</tr>

<tr>
<th align="left"><code>:root.night</code></th>
<td align="center"><code>navy</code></td>
<td align="center"><code>navy</code></td>
<td align="center"><code>navy</code></td>
</tr>
</table>

⚠️ If you are stacking variants on while using both selectors and media queries to activate themes, then **make sure that each stacked variant is specified as an *all or nothing* approach** on each element. For instance, if you have `normal-motion` and `reduced-motion` themes and want to add the `reduced-motion:hover:transition-none` class, then you also need to add the `normal-motion:hover:transition` class (or any [value of `transitionProperty`](https://tailwindcss.com/docs/transition-property/)). If you don't do this, then it will look like the values from a theme that's *supposed* to be inactive are "leaking" into the active theme.

### Fallback
TODO

## Call the plugin more than once for multiple groups
TODO

## The ultimate example: how I use every feature together in production
TODO

# Alternatives
Both because there are many theme plugins for Tailwind CSS, and because "what's the right way to do theming" is a frequently asked question, we've compiled this table listing every theme plugin to compare their featuresets:

<table>
    <thead>
        <tr>
            <th></th>
            <th><a href="https://tailwindcss.com/docs/breakpoints/#dark-mode">Native screens</a></th>
            <th><a href="https://github.com/ChanceArthur/tailwindcss-dark-mode">tailwindcss-dark-mode</a></th>
            <th><a href="https://github.com/danestves/tailwindcss-darkmode">tailwindcss-darkmode</a></th>
            <th><a href="https://github.com/estevanmaito/tailwindcss-multi-theme">tailwindcss-multi-theme</a></th>
            <th><a href="https://github.com/javifm86/tailwindcss-prefers-dark-mode">tailwindcss-prefers-dark-mode</a></th>
            <th><a href="https://github.com/crswll/tailwindcss-theme-swapper">tailwindcss-theme-swapper</a></th>
            <th><a href="https://github.com/SirNavith/tailwindcss-theme-variants">tailwindcss-theme-variants</a></th>
            <th><a href="https://github.com/innocenzi/tailwindcss-theming">tailwindcss-theming</a></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th>Classes can be <code>@apply</code>ed</th>
            <td>❌*</td>
            <td>❌**</td>
            <td>❌**</td>
            <td>❌**</td>
            <td>❌**</td>
            <td>✅</td>
            <td>❌</td>
            <td>✅</td>
        </tr>
        <tr>
            <th>Controllable with selectors (classes or data attributes)</th>
            <td>❌</td>
            <td>✅</td>
            <td>✅</td>
            <td>✅</td>
            <td>✅º</td>
            <td>✅</td>
            <td>✅</td>
            <td>✅</td>
        </tr>
        <tr>
            <th>Responsive†</th>
            <td>❌</td>
            <td>✅</td>
            <td>✅</td>
            <td>✅</td>
            <td>✅</td>
            <td>✅</td>
            <td>✅</td>
            <td>✅</td>
        </tr>
        <tr>
            <th>Requires <a href="https://developer.mozilla.org/en-US/docs/Web/CSS/--*">custom properties</a>‡</th>
            <td>❌</td>
            <td>❌</td>
            <td>❌</td>
            <td>❌</td>
            <td>❌</td>
            <td>✅</td>
            <td>❌</td>
            <td>✅</td>
        </tr>
        <tr>
            <th>Supports <a href="https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme"><code>prefers-color-scheme: dark</code></a>★</th>
            <td>✅</td>
            <td>❌</td>
            <td>❌</td>
            <td>❌</td>
            <td>✅º</td>
            <td>✅</td>
            <td>✅</td>
            <td>✅</td>
        </tr>
        <tr>
            <th>Supports <a href="https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme"><code>prefers-color-scheme: light</code></a>★</th>
            <td>✅</td>
            <td>❌</td>
            <td>❌</td>
            <td>❌</td>
            <td>❌</td>
            <td>✅</td>
            <td>✅</td>
            <td>✅</td>
        </tr>
    </tbody>
</table>

†**Responsive**: While "inside" of a theme, it must be possible to "activate" classes / variants depending on the current breakpoint. For instance, it has to be possible to change `background-color` when **both** the screen is `sm` **and** the current theme is `dark`.

‡**Requires <a href="https://developer.mozilla.org/en-US/docs/Web/CSS/--*">custom properties</a>**: Plugins who meet this description (have a ✅) usually have you write semantically named classes like `bg-primary`, `text-secondary`, etc, and swap out what `primary` and `secondary` mean with custom properties depending on the theme. This means that in IE11, themes cannot be controlled, and in some cases the default theme won't work at all without [preprocessing](https://github.com/postcss/postcss-custom-properties).

★`prefers-color-scheme`, like any media query, [can be detected in JavaScript](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia). Any plugin marked as not supporting `prefers-color-scheme` could "support" it by adding or removing classes or data attributes, such as with the [`prefers-dark.js` script](https://github.com/estevanmaito/tailwindcss-multi-theme/blob/master/prefers-dark.js).

*[Native screens](https://tailwindcss.com/docs/breakpoints/#dark-mode) cannot have their generated classes `@apply`ed, but you can still nest an `@screen` directive within the element, like this: 
```css
.btn-blue {
    @apply bg-blue-100 text-blue-800;
    /* Wouldn't have worked: @apply dark:bg-blue-700 dark:text-white */
    @screen dark {
        @apply bg-blue-700 text-white;
    }
}
```
This may require nesting support, provided by [`postcss-nested`](https://github.com/postcss/postcss-nested) or [`postcss-nesting`](https://github.com/jonathantneal/postcss-nesting) (part of [`postcss-preset-env`](https://github.com/csstools/postcss-preset-env)).

**You can nest under whatever selector(s) the theme plugin uses to control themes

º[`tailwindcss-prefers-dark-mode`](https://github.com/javifm86/tailwindcss-prefers-dark-mode) cannot use selectors and media queries at the same time; it's one or the other.

# License and Contributing

MIT licensed. There are no contributing guidelines. Just do whatever you want to point out an issue or feature request and I'll work with it.


---

*Repository preview image generated with [GitHub Social Preview](https://social-preview.pqt.dev/)*
