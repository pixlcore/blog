<!-- Title: Beyond Dark Mode: Let Users Tune Your UI with CSS Filters -->
<!-- Summary: Allow your users to customize a set of full-screen filters for your web app. -->
<!-- Author: jhuckaby -->
<!-- Date: 2025/09/09 -->
<!-- Tags: Filters, CSS -->

Most web apps today offer users a binary theme choice: **light mode** or **dark mode**. But vision accessibility needs aren't binary. Some people need *slightly* dimmer screens at night, others need higher contrast, or find a warmer hue easier to read. Rather than shipping dozens of themes, which would be a lot of work for the developer / designer, we can let users fine‑tune the interface themselves using [CSS filters](https://developer.mozilla.org/en-US/docs/Web/CSS/filter).

But wait, you may be asking, if a user has vision accessibility needs, won't they already have software to adjust their screen, like at the OS level?  Well, I mean, maybe, but do we know that?  What if they are using a public or guest computer, or on their smartphone?  We never know their situation, so giving the user more options is almost always better!

## Introduction

This article is a step‑by‑step tutorial for building a simple, user‑tunable system with [range sliders](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/range) and [backdrop-filter](https://developer.mozilla.org/en-US/docs/Web/CSS/backdrop-filter). The settings are saved to [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage), so they persist across sessions.

We'll build it in two parts:

1. Applying a `backdrop-filter` overlay to your entire UI (with passthrough pointer events).
2. Adding sliders to let users adjust brightness, contrast, saturation, and hue.

## The Backdrop Overlay

The core trick is to add a full‑screen overlay with a `backdrop-filter` on it. This overlay applies filters to everything *behind* it, while passing through all pointer events (clicks, hovers, etc.).

Why not just apply the filters to the root HTML or BODY elements?  Well, you *can*, but doing it with an overlay element allows you to float controls above it.  Also, in my experience adding filters to the root elements introduces a bunch of weird side effects (fixed scrolling gets wonky, root background color is not included, etc.).  There are also performance concerns with long pages, because the filter is being applied across the *entire* page, even offscreen content.  After a bunch of research and testing, I've found that a screen-sized fixed overlay DIV is the best way to implement this.

Worried about performance?  You needn't.  Filters are rendered entirely on the GPU nowadays, and in my testing on a 2019 Intel MacBook Pro, I still get 60FPS smooth scrolling even with multiple filters applied in the overlay.  Now, if your web app itself already has a large amount of complex animations and/or filters, this *may* introduce some lag.  But for most web apps the performance hit should be negligible.

## Step 1: HTML Markup

Let's start with the easy part.  We need to add a single "forever" element in the HTML to float above everything.  This should go just inside your BODY tag.  Don't worry, we'll keep it entirely hidden until it is needed.  We also add `aria-hidden` so screen readers ignore it.

```html
<div id="filter-overlay" aria-hidden="true"></div>
```

## Step 2: Initial CSS Styles

Now let's add the initial styles to the overlay.  It is sized to cover the entire viewport, will float above everything and scroll with it, and will pass all pointer events straight through.

```css
#filter-overlay {
	position: fixed;
	inset: 0;
	z-index: 1000;
	pointer-events: none;
	backdrop-filter: none;
	will-change: backdrop-filter;
	display: none;
}
```

Let me explain these CSS properties:

| CSS Property | Description |
|--------------|-------------|
| `position: fixed` | This makes the DIV fixed in place, so it won't move or scroll, regardless of the underlying page size. |
| `inset: 0` | This is a shorthand property which sets the `left`, `top`, `right` and `bottom` to the same value.  Setting them all to zero makes the DIV take up the entire screen. |
| `z-index: 1000` | Float the DIV above everything else in your app.  Tune this to taste for your app. |
| `pointer-events: none` | This makes all pointer (mouse) events pass straight through the overlay, including hovers and clicks. |
| `backdrop-filter: none` | Start with no filters.  We will dynamically add them later as needed. |
| `will-change: backdrop-filter` | Give the browser a hint that we will be changing the `backdrop-filter` property over time.  [Read More](https://developer.mozilla.org/en-US/docs/Web/CSS/will-change). |
| `display: none` | Start out completely hidden. |

At this point nothing is visible, but the overlay is ready to use. Next, we'll teach it to apply filters.

## Step 3: Applying The Filters

Let's start by writing the function that will apply the current filters to the overlay element.  We assume a pre-defined object called `filters` which is passed into the function.

```js
const overlay = document.getElementById('filter-overlay');

function applyFilters(filters) {
	const css = [
		(filters.brightness !== 100) ? `brightness(${filters.brightness}%)` : '',
		(filters.contrast !== 100)   ? `contrast(${filters.contrast}%)` :     '',
		(filters.hue !== 0)          ? `hue-rotate(${filters.hue}deg)` :      '',
		(filters.saturate !== 100)   ? `saturate(${filters.saturate}%)` :     '',
		(filters.sepia !== 0)        ? `sepia(${filters.sepia}%)` :           '',
		(filters.grayscale !== 0)    ? `grayscale(${filters.grayscale}%)` :   '',
		(filters.invert !== 0)       ? `invert(${filters.invert}%)` :         ''
	].join(' ').trim();
	
	overlay.style.display = css ? 'block' : 'none';
	overlay.style.backdropFilter = css || 'none';
}
```

The idea here is that we build up a CSS string for the `backdrop-filter` property value, based on any user filters that are adjusted outside of their defaults.  If no filters are adjusted, and everything is at the default settings, we disable the filter and hide the entire element (i.e. for zero overhead if the feature is not being used).

## Step 4: Add User Controls

Now let's add a small control panel with sliders.  First, the HTML markup:

```html
<section id="visual-controls" role="region" aria-label="Visual preferences">
	<h2>Visual Preferences</h2>

	<label for="vf-brightness">Brightness</label>
	<input type="range" id="vf-brightness" min="25" max="200" value="100">

	<label for="vf-contrast">Contrast</label>
	<input type="range" id="vf-contrast" min="25" max="200" value="100">
	
	<label for="vf-hue">Hue</label>
	<input type="range" id="vf-hue" min="-180" max="180" value="0">

	<label for="vf-saturate">Saturation</label>
	<input type="range" id="vf-saturate" min="0" max="200" value="100">

	<label for="vf-sepia">Sepia</label>
	<input type="range" id="vf-sepia" min="0" max="100" value="0">
	
	<label for="vf-grayscale">Grayscale</label>
	<input type="range" id="vf-grayscale" min="0" max="100" value="0">
	
	<label for="vf-invert">Invert</label>
	<input type="range" id="vf-invert" min="0" max="100" value="0">

	<button id="vf-reset" type="button">Reset</button>
</section>
```

And now, some simple CSS styling for the controls:

```css
#visual-controls {
	position: fixed;
	right: 1rem;
	bottom: 1rem;
	padding: 1rem;
	border-radius: 12px;
	background: rgba(0,0,0,0.7);
	color: white;
	z-index: 1001; /* Above our overlay */
}

#visual-controls label { display: block; margin-top: .5rem; }
#visual-controls input[type=range] { width: 100%; }
```

One thing to point out here is that we're floating the controls *above* our overlay filter using `z-index`.  This is important, because the user may accidentally adjust the sliders in such a way where the UI cannot be seen anymore.  We always want to give the user the ability to restore the original settings.

## Step 5: Wire It All Together

Now let's wire everything up with some simple JavaScript.  First, here is the initialization code to run at page load time.  This will fetch the initial values if previously saved, and apply the current set of filters:

```js
let state = {
	brightness: 100,
	contrast: 100,
	hue: 0,
	saturate: 100,
	sepia: 0,
	grayscale: 0,
	invert: 0
};

// Load saved values, merge in with default state
try {
	const saved = JSON.parse( localStorage.getItem('filters') );
	if (saved) state = { ...state, ...saved };
} catch (e) {}

// immediately apply filters
applyFilters(state);
```

Now, let's write the code to handle user interaction with the range sliders:

```js
// Connect sliders
Object.keys(state).forEach(key => {
	const el = document.getElementById('vf-' + key);
	el.value = state[key];
	
	el.addEventListener('input', () => {
		// dragging in progress
		state[key] = el.valueAsNumber;
		applyFilters(state);
	});
	
	el.addEventListener('change', () => {
		// drag complete, save changes
		localStorage.setItem('filters', JSON.stringify(state));
	});
});
```

This sets the initial value for each range slider, and attaches an `input` listener to update the filters as the user slides.  We use `input` instead of `change` so our handler gets called *continuously during a drag*, for real-time user feedback.  We also add a separate `change` event listener (which fires only on mouse release), to save the changes to localStorage.

And now for the reset button, which reverts everything to defaults, removes the localStorage key, removes the filters from the overlay, and resets the range sliders:

```js
// Reset button
const resetBtn = document.getElementById('vf-reset');
resetBtn.addEventListener('click', () => {
	state = { brightness: 100, contrast: 100, saturate: 100, hue: 0, sepia: 0, grayscale: 0, invert: 0 };
	localStorage.removeItem('filters');
	
	Object.keys(state).forEach(key => {
		document.getElementById('vf-' + key).value = filters[key];
	});
	
	applyFilters(state);
});
```

And that's about it!  Want to see it in action?  Click the button just below:

<div style="margin:30px;"><div class="button" onClick="app.openFilterControls(this)"><i class="mdi mdi-palette">&nbsp;</i>Open Filter Controls...</div></div>

In case you're wondering why we are offering a grayscale slider, when you can simply slide the saturation all the way to the left, the answer is: they actually do slightly different things!  Desaturating removes color in a mathematical sense, where the red, green and blue values are averaged, but using the dedicated "grayscale" filter actually does it in a more human-perceptual way, where the different colors are "weighted" based on our eye cones.  Honestly, most users probably won't notice or care, so let's just say I added it for "completeness", haha. 

And why would we include an "invert" slider, you ask?  Because it's FUN, dammit!  Seriously though, who knows what user accessibility needs are.  Someone may benefit from inversion.  Oh, and inverting to 100% plus hue shift to +180 does give you a "poor man's dark theme".

## Future Ideas

Here are some ideas to enhance the experience further:

- **Keyboard Shortcut:** Give the user a key to hit to bring up the visual controls, in case they accidentally hide the entire UI, then dismiss the control dialog.
- **Persistence:** If your app has user accounts, persist all the filter selections to the server, so if the user logs back in on a different browser or machine, their settings will be restored.
- **Printing:** Consider disabling filters when `@media print` is active.
- **De-Hue Images:** If your app features images or videos, consider "reversing" the hue selection for those elements, to preserve their original colors.
- **Presets:** Offer a set of slider presets such as "warm", "cool", etc.

## Summary

That's it — just an overlay, a handful of sliders, and `localStorage`. You've given your users more than a blunt light/dark toggle: they can tune their experience to match their personal comfort or accessibility needs.
