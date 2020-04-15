# Canvas Elemnt and The Sub Projects Within

This section of Creative Javascript covers the `<canvas>` element, and the tools that you can use to interact with it. By interacting of course I mean drawing animations.

## Table of Contents

| Section | Name | 
| --- | --- |
| [I.](#bouncing-box) | [Bouncing Box](#bouncing-box) |

___

## Bouncing Box

The fist chapter of this book hits the ground running with ceating a box that (supposedly) follows the mouse around the computer screen, and then bounces off the sides of the container.

The setup for this project consists of the following HTML:

```html
<html>
  <body>
    <canvas id='animation'>
    </canvas>
    <script src='./scripts/bouncingBox.js'>
  </body>
</html>
```

### What I Learned

- The reintroduction to vanilla Javascript and embedding it in HTML was a trip. React really abstracts that aspect of things.

- Loading the script at the bottom of the page got rid of the need for a method that listens for the rendering of the dom. In jQuery this was `document.ready()`, but there was a cool work around that I found using vanilla JavaScript:
  - `document.addEventListener('DOMContentLoaded', (e) => { ... //code here });`
  
  Ultimately I did not go down this path and settled on following along with the code in the book verbatim.

- Polyfills are functions that set one variable to many different variables depending on the usecase. In this code we used one to access the `requestAnimationFrame` method in all browsers.

- You can render animations recursively and make them smooth by setting the `setInterval` method to fire at 1000fps.

- `document.getElementByID()` returns the HTML element whereas `document.getElementByTagName()` returns an array.

### Code Flow

Our intial setup starts out by capturing the canvas element, and setting its width and height to the size of the window. This will allow for the entire screen to be used in the animation. Then we create a context interface used for drawing 2 dimensional shapes on the DOM. Afte this is written we set changable global variables used to set the starting point, a duration, sizing, movement direction, and a distance for our box to bounce.

After the variables are declared seven functions are needed to bounce the box around the screen:

1. [`logic`](#logic())
2. [`draw`](#draw())
3. [`lerp()`](#lerp())
4. [`degreesToRadians()`](#degreesToRadians())
5. [`dir_x`/`dir_y`](#directions)

### Canvas Methods Used

| Method Name | Definition | Example |
| --- | --- | --- |
| `getContext('2d')` | Returns a drawing context on the screen. Analogous to a painters canvas. It returns a `CanvasRenderingContext2D` interface.| `document.querySelector("#canvas-id-here").getContext('2d')`. _It is important to note that this method can take multiple paramaters to create different objects._  |
| `clearRect(x, y, width, height)` | Erases pixels within the canvas area by setting them to black. This method uses a rectangluar area with the starting coordinates being at x, y and a size specified by the width and height attributes. | `ctx.clearRect(0, 0, ele.width, ele.height)` |
| `fillStyle` | Not really a method, but a property of the shapes within the canvas. Can be set to a color, gradient, or a pattern. | `ctx.fillStyle = "#eee"` |
| `fillRect` | Draws a rectangle that fills accordingly to the `fillStyle` property. | ctx.fillRect(x, y, 50, 50) _i.e.(starting coordinates x/y, width, height._ |

### Methods Created

| Method Name | Definition | Example |
| --- | --- | --- |
| `logic()`| This function controls our settings and makes sure the box bounces in the window smoothly. | `setInterval(logic 1000/60)`|

- #### `logic()`
  - 