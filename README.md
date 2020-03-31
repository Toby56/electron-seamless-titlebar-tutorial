
# Electron seamless titlebar tutorial (Windows 10 style)<!-- omit in toc -->

### A guide to creating a seamless Windows 10 title bar in your Electron app.<!-- omit in toc -->

![Intro 1]
![Intro 2]

I was inspired by the way [Hyper terminal](https://hyper.is/) achieved a native look, and a tutorial by [Shawn Rakowski](https://github.com/srakowski) (no longer available, it seems).

I'm going to start with the [Electron quick start app](https://github.com/electron/electron-quick-start). After this tutorial, you should end up with a minimal app template that you can customize to your liking. The full example app source code is located in the [`src` directory](/src) of this repo, so you can just clone and run to give it a try, or use it as a template.

> **Note:** Currently, there is a bug with window maximization on Electron versions 6.0.0+ on Windows (see [#6](https://github.com/binaryfunt/electron-seamless-titlebar-tutorial/issues/6)).
>
> There is also a scaling issue with the chromium window

## How it works<!-- omit in toc -->

## Installation of example app<!-- omit in toc -->

Assuming you have [Node](https://nodejs.org/en/) installed, clone this repo, navigate to the `src` directory, and run `npm install` and `npm start` to launch the app.

**Command line:**

```bash
git clone https://github.com/binaryfunt/electron-seamless-titlebar-tutorial.git

cd electron-seamless-titlebar-tutorial/src

npm install

npm start
```

# The tutorial<!-- omit in toc -->

- [1. Let's begin!](#1-lets-begin)
  - [Starting out with the electron quickstart app](#starting-out-with-the-electron-quickstart-app)
  - [Adding some basic styling](#adding-some-basic-styling)
  - [Theming](#theming)
- [2. Making the window frameless](#2-making-the-window-frameless)
  - [Bye bye old ugly window frame!](#bye-bye-old-ugly-window-frame)
  - [Creating a replacement title bar](#creating-a-replacement-title-bar)
- [4. Make the title bar draggable](#4-make-the-title-bar-draggable)
- [5. Add window control buttons](#5-add-window-control-buttons)
- [6. Style the window control buttons](#6-style-the-window-control-buttons)
- [7. Add the window title](#7-add-the-window-title)
- [8. Implement window controls functionality](#8-implement-window-controls-functionality)
- [9. Adding styling for when the window is maximized](#9-adding-styling-for-when-the-window-is-maximized)
- [Customisability and more!](#customisability-and-more)
  - [Theming more in-depth](#theming-more-in-depth)
  - [The 1px border!](#the-1px-border)
  - [WIndows 10 Acrylic effect](#windows-10-acrylic-effect)
  - [Multiplatform compatability (having a seamless titlbar on MacOS)](#multiplatform-compatability-having-a-seamless-titlbar-on-macos)

## 1. Let's begin!

![S1]

### Starting out with the electron quickstart app

First, we're going to clone the [electron-quickstart-app](https://github.com/electron/electron-quick-start) from GitHub (`git clone https://github.com/electron/electron-quick-start`). You should run `npm install` to install all the dependencies, and `npm start` every time you want to run your app. If everything goes well, you will have the starting point of our app:

And you should have these files inside the project:

```
.gitignore
index.html
LICENSE.md
main.js
package-lock.json
package.json
preload.js
README.md
renderer.js
```

For the moment we'll just need to add a `css` file for our styles, call it `style.css`.

Next we'll open `index.html` and make some modifications. We'll wrap all the main content in it's own `<div id="main">` element and put all the text under the heading in a `<p>` element.

Afterwards your code should look something like this:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <meta http-equiv="X-Content-Security-Policy" content="default-src 'self'; script-src 'self'">

    <link rel="stylesheet" href="style.css" />

    <title>Hello World!</title>
  </head>
  <body>
    <div id="main">
      <h1>Hello World!</h1>
      <p>
        We are using Node.js <span id="node-version"></span>, Chromium
        <span id="chrome-version"></span>, and Electron
        <span id="electron-version"></span>.
      </p>
    </div>

    <!-- You can also require other files to run in this process -->
    <script src="./renderer.js"></script>
  </body>
</html>
```

And the app should look something like this:

As you can see, we've also added some lorem ipsum so our window isn't looking so empty!

### Adding some basic styling

Next we'll add some basic styles to make the app look a bit better than some sort of web 1.0 site. Add the following to `style.css`:

```css
* {
  box-sizing: border-box;
}

body {
  height: 100vh;
  margin: 0;
  font-family: "Segoe UI", sans-serif;
}

h1 {
  margin-top: 0;
  font-weight: 400;
  line-height: 1;
}

p {
  opacity: 0.8;
  line-height: 1.5;
}

#main {
  position: fixed;
  top: 32px;
  bottom: 0;
  padding: 32px;
  overflow-y: auto;
}
```
`#main` will now become our scrolling element (instead of `body`) and we've left some space above it for the upcoming titlebar.

### Theming

One small feature of the titlebar in this tutorial is that the text and icons can be be black or white depending on whether the b At the moment this is done by applying a class to the `<body>` element and styling everything accordingly, like so:

```css
body.theme-light {
  background-color: #fff;
  color: #000;
  border-color: #0003;
}

body.theme-dark {
  background-color: #1a2933;
  color: #fff;
  border-color: #fff3;
}
```

As you can see, there are two classes, one for light themes and one for dark themes. This is pretty convenient because you can switch between light and dark at any time just by changing the class on `<body>`. For now just add the CSS and the `theme-dark` class so that everything works:

```html 
<body class="theme-dark">
```

In the CSS, you can also see I also set the body background color depending on what theme is set. The dark theme has a nice dark blue-green, but you can change the background colors for each theme to whatever you want. If you only want a light theme and/or a dark theme, then this is good! Just set and forget the theme, and/or set/switch it with JavaScript using the classes.

If you want your app to have multiple background colors/themes, you can also set the background color and theme class instead with JavaScript. Or, if you really only want one theme, merge it's styles with the rest of the body styles, and do the same thing with any other elements that are styled depending on theme later on.

I've also added a subtle 1px border to the window. A note on theming in relation to this and theming in general at the [end of the tutorial](#theming-more-in-depth).

## 2. Making the window frameless

### Bye bye old ugly window frame!

The next step in our transition to a seamless titlebar is to remove the standard Windows title bar and border. In `main.js`, modify the `new BrowserWindow()` line so it includes `frame: false`:

```javascript
const mainWindow = new BrowserWindow({
  width: 800,
  height: 600,
  backgroundColor: "#1a2933",
  frame: false,
  webPreferences: {
    preload: path.join(__dirname, "preload.js")
  }
});
```

Also make sure you set the `backgroundColor` to whatever you want it to be, which will make you app look smoother when it starts and also enables sub-pixel antialiasing ([see here for details](https://github.com/electron/electron/issues/6344#issuecomment-420371918)).

> **Tip:** uncomment `mainWindow.webContents.openDevTools()` to open developer tools every time the app is run.

If everything worked, you can run the app now and you'll see the title bar is gone!

![S2]

See the [docs](https://electronjs.org/docs/api/frameless-window) for more info on frameless windows.

### Creating a replacement title bar

We're going to create our own title bar using HTML and CSS. Just add some HTML like so:

```html
<body class="theme-dark">
  <!-- Titlebar -->
  <div id="titlebar">
    <div id="titlebar-drag"></div>
    <div id="titlebar-text"></div>
    <div id="titlebar-buttons"></div>
  </div>

  <div id="main">
    <h1>Hello World!</h1>
    <p>
      We are using Node.js <span id="node-version"></span>,
      Chromium <span id="chrome-version"></span>, 
      and Electron <span id="electron-version"></span>.
    </p>
  </div>

  <!-- You can also require other files to run in this process -->
  <script src="./renderer.js"></script>
</body>
```

You can see there isn't very much at the moment.

The default title bar height in Windows is 32px. We want the titlebar fixed at the top of the DOM. I'm giving it a background colour temporarily so we can see where it is. We need to make sure `#main` is position 32px lower, so it doesn't overlap with the titlebar (`#main` is now replacing `body` as the scrolling content).

```css
#titlebar {
  position: fixed;
  height: 32px;
  width: calc(100% - 2px);
  display: grid;
  grid-template-columns: auto 138px;
  z-index: 1000;
  background-color: #254053;
}

#titlebar-text {
  grid-column: 1;
  font-family: "Segoe UI", sans-serif;
  padding-left: 12px;
  font-size: 12px;
  user-select: none;
  overflow: hidden;
  text-overflow: ellipsis;
  line-height: 32px;
  white-space: nowrap;
}

/* Make sure the text is white instead of black when the theme is dark */
.theme-dark #titlebar-text {
  color: white;
}

/* Make sure the drag region has space around it for the window resize handles */
#titlebar-drag {
  position: absolute;
  top: 4px;
  left: 4px;
  right: 4px;
  bottom: 0;
  -webkit-app-region: drag;
}

#titlebar-buttons {
  display: grid;
  grid-template-columns: repeat(3, 46px);
  position: relative;
  -webkit-app-region: none;
}
```

![S3]

> **Tip:** you can do <kbd>ctrl</kbd>+<kbd>R</kbd> to reload the the webpage whithout restarting the whole app.

## 4. Make the title bar draggable

You might notice our new titlebar isn't actually draggable. To fix this, we add a div to `#titlebar`:

```html
<header id="titlebar">
  <div id="drag-region"></div>
</header>
```

We need to give it a style of `-webkit-app-region: drag`. The reason we don't just add this style to `#titlebar` is that we also want the cursor to change to resize when we hover near the edge of the window at the top. If the whole title bar was draggable, this wouldn't happen. So we also add some padding to the non-draggable `#titlebar` element.

```css
#titlebar {
  padding: 4px;
}

#titlebar #drag-region {
  width: 100%;
  height: 100%;
  -webkit-app-region: drag;
}
```

If you reload now, you will be able to drag the window around again, and the window can be resized at the top.

## 5. Add window control buttons

It's time to add the minimise, maximise, restore and close buttons. To do this, we'll need the icons. [A previous version of this tutorial](https://github.com/binaryfunt/electron-seamless-titlebar-tutorial/tree/v1.0.0#5-add-window-control-buttons) used the [Segoe MDL2 Assets](https://docs.microsoft.com/en-us/windows/uwp/design/style/segoe-ui-symbol-font) font for the icons. However, these have some issues, such as [high DPI display scaling](https://github.com/binaryfunt/electron-seamless-titlebar-tutorial/issues/11) and [lack of cross-platform compatibility](https://github.com/binaryfunt/electron-seamless-titlebar-tutorial/issues/1).

To overcome these issues, I recreated the icons as PNGs for crisp viewing at display scaling factors from 100-350% (but only the predefined Windows 10 scaling values; for custom scaling values you will probably see some anti-aliasing blur/pixelation and may be better off using the old method or the [SVG approach](https://github.com/binaryfunt/electron-seamless-titlebar-tutorial/pull/13)).

The icons are located in [`src/icons`](src/icons). Make sure to copy them into your project. We'll make use of the [`srcset` attribute](https://bitsofco.de/the-srcset-and-sizes-attributes/#thesrcsetattribute) of the `<img>`, we can load the correct resolution icons for the user's display scaling factor/device pixel ratio.

Because having all of the srcsets We'll put the buttons inside the `#titlebar-buttons` div.

```javascript
// All of the Node.js APIs are available in the preload process.
// It has the same sandbox as a Chrome extension.
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector)
    if (element) element.innerText = text
  }

  for (const type of ['chrome', 'node', 'electron']) {
    replaceText(`${type}-version`, process.versions[type])
  }
})
```

The buttons are 46px wide & 32px high. We'll use [CSS grid](https://css-tricks.com/snippets/css/complete-guide-grid/) to overlap the maximise/restore buttons, and later use JavaScript to alternate between them.

```css
#titlebar {
  color: #fff;
}

#window-controls {
  display: grid;
  grid-template-columns: repeat(3, 46px);
  position: absolute;
  top: 0;
  right: 0;
  height: 100%;
}

#window-controls .button {
  grid-row: 1 / span 1;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
}
#min-button {
  grid-column: 1;
}
#max-button,
#restore-button {
  grid-column: 2;
}
#close-button {
  grid-column: 3;
}
```

![S5]

For some reason, the PNG icons are not rendered at the correct 10 device independent pixels width/height for the scaling factors 150%, 200% or 300%, so we need to target those using a [media query](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries) and the [`-webkit-device-pixel-ratio` feature](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/-webkit-device-pixel-ratio).

```css
@media (-webkit-device-pixel-ratio: 1.5),
  (device-pixel-ratio: 1.5),
  (-webkit-device-pixel-ratio: 2),
  (device-pixel-ratio: 2),
  (-webkit-device-pixel-ratio: 3),
  (device-pixel-ratio: 3) {
  #window-controls .icon {
    width: 10px;
    height: 10px;
  }
}
```

The feature `device-pixel-ratio` without the webkit prefix does not exist yet, but no harm in future-proofing. The reason we don't just give the width/height rule for all device pixel ratios is that this leads to fuzzy anti-aliasing for some of them.

## 6. Style the window control buttons

First of all, the buttons shouldn't be part of the window drag region, so we'll exclude them. Also, we don't want to be able to select the icon images. We also need to add hover effects. The default Windows close button hover colour is `#E81123`. When active, the button becomes `#F1707A` and the icon becomes black, which can be achieved using [the invert filter](https://developer.mozilla.org/en-US/docs/Web/CSS/filter-function/invert). Lastly, we'll hide the restore button by default (again, we'll implement switching between the maximise/restore buttons later).

```css
#window-controls {
  -webkit-app-region: no-drag;
}

#window-controls .button {
  user-select: none;
}
#window-controls .button:hover {
  background: rgba(255, 255, 255, 0.1);
}
#window-controls .button:active {
  background: rgba(255, 255, 255, 0.2);
}

#close-button:hover {
  background: #e81123 !important;
}
#close-button:active {
  background: #f1707a !important;
}
#close-button:active .icon {
  filter: invert(1);
}

#restore-button {
  display: none !important;
}
```

![S6]

## 7. Add the window title

There are lots of ways you could do this, depending on whether you wanted to add any buttons or file menus to the titlebar, and whether you wanted the title centered or to the left.

![S7]

My way is to put it the left. Add this inside the `#drag-region` div, above the window controls:

```html
<div id="window-title">
  <span>Electron quick start</span>
</div>
```

I've gone with grid, as you can change the template columns to suit whatever you decide to do. Here we have an auto width column and a 138px column (3 \* 46px = 138px). The default Windows window title font size is 12px. The default margin/padding on the left of the title is 10px, but 2px is already taken up by `#titlebar`'s padding, so 8px of margin/padding is needed on the left. I've taken the precaution of hiding any overflowing text if the title happens to be very long.

```css
#titlebar #drag-region {
  display: grid;
  grid-template-columns: auto 138px;
}

#window-title {
  grid-column: 1;
  display: flex;
  align-items: center;
  margin-left: 8px;
  overflow: hidden;
  font-family: "Segoe UI", sans-serif;
  font-size: 12px;
}

#window-title span {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  line-height: 1.5;
}
```

At this point, you can remove the background colour from `#titlebar` and admire your handiwork.

![Final]

## 8. Implement window controls functionality

Now, open `renderer.js`. We're going to code the windows controls, which is part of a single BrowserWindow instance and so should be in `renderer.js` as opposed to `main.js` (according to my interpretation of [the docs](https://electronjs.org/docs/tutorial/application-architecture)). The code isn't too complex so here it is in its entirety:

```javascript
const remote = require("electron").remote;

const win = remote.getCurrentWindow();

// When document has loaded, initialise
document.onreadystatechange = event => {
  if (document.readyState == "complete") {
    handleWindowControls();
  }
};

window.onbeforeunload = event => {
  /* If window is reloaded, remove win event listeners
    (DOM element listeners get auto garbage collected but not
    Electron win listeners as the win is not dereferenced unless closed) */
  win.removeAllListeners();
};

function handleWindowControls() {
  // Make minimise/maximise/restore/close buttons work when they are clicked
  document.getElementById("min-button").addEventListener("click", event => {
    win.minimize();
  });

  document.getElementById("max-button").addEventListener("click", event => {
    win.maximize();
  });

  document.getElementById("restore-button").addEventListener("click", event => {
    win.unmaximize();
  });

  document.getElementById("close-button").addEventListener("click", event => {
    win.close();
  });

  // Toggle maximise/restore buttons when maximisation/unmaximisation occurs
  toggleMaxRestoreButtons();
  win.on("maximize", toggleMaxRestoreButtons);
  win.on("unmaximize", toggleMaxRestoreButtons);

  function toggleMaxRestoreButtons() {
    if (win.isMaximized()) {
      document.body.classList.add("maximized");
    } else {
      document.body.classList.remove("maximized");
    }
  }
}
```

## 9. Adding styling for when the window is maximized

Now all there is to do is to add some CSS for when the window is in the maximized state. When the window is maximized we should be able to drag from the very top down to restore it, so we should remove the drag region spacing, and we also  need to swap between the maximize and restore buttons.

```css
.maximized #titlebar {
  width: 100%;
  padding: 0;
}

.maximized #window-title {
  margin-left: 12px;
}

.maximized #restore-button {
  display: flex !important;
}

.maximized #max-button {
  display: none;
}
```

## Customization and more!

### Theming more in-depth

### The 1px border!

The 1px border is quite problematic, firstly because of a chromium bug that caused 1px from the bottom and right edges of the window to be usually cut off depending on window dimensions on scaling factors that aren't a multiple of 100%, and secondly because when the everything is scaled, the border will get thicker, which we don't want.

So everything is okay for 100% scaling, but for any other scaling factor (which will be enabled by default on windows for resolutions 1080p and above), the border is likely to be messed up in one way or another. That might be one reason why none of the other apps that use electron or NW.js and have a custom Windows titlebar (GitHub Desktop, Spotify, VS Code, Slack, Discord, Microsoft Teams etc) have a 1px border. It looks completely fine without it, so I would recommend not having it.

### WIndows 10 Acrylic effect

One of the screenshots at the top shows off a fancy transparent Windows 10 fluent acrylic window background, a bit like vibrancy for Mac. This, unlike vibrancy, is not a standard electron feature by any means, and it has some pitfalls. BUT, it also looks really epic, so I will tell you that it can be achieved fairly easily by installing a node module called ["ewc"](https://github.com/23phy/ewc) into you project, importing it with node, setting the background color of your app to `#0000` and putting in the following code:

```javascript
ewc.setAcrylic(win, 0xbb000000);
```

The first argument is the electron window that you want to have the acrylic effect, and the second is an ARGB hexadecimal color value that you want the acrylic to be. This means you can make the acrylic background any color or opacity you want. Give it a try!

You can find all the other functions of ewc in it's source code and example app [here](https://github.com/23phy/ewc). The main problem with ewc at the moment is that there is a Windows mouse polling issue which means window dragging is really slow and delayed when acrylic is enabled.

### Multiplatform compatability (having a seamless titlebar on MacOS)

This tutorial will look great on Windows, but what about Mac? If you want your app to be cross platform and have a seamless titlebar on MacOS too, then it is easy! Just use the `titleBarStyle: 'hidden'` (which basically activates a Mac feature that does all of this for you lol). That's great! Then you will need to only enable the Windows titlebar on Windows. Just use:

```javascript
if (process.platform == "win32") {
  // CODE!
}
```

And put all of the titlebar buttons code in there instead and just leave the bar and text somehow

[intro 1]: screenshots/Intro-1.png
[intro 2]: screenshots/Intro-2.png
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTc5OTUyNTk1LC0yMDA3ODk4MDc2XX0=
-->