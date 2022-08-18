# Babel - Client-side for ECMAScript modules and expressions

To play around with some tools I like, I need to learn React, welp!

[George Offley](https://twitter.com/georgeoffley) recommended this free [beginner course for react](https://scrimba.com/learn/learnreact)
because it combines theory and practice well: lots of projects to try out whose design is in Figma.

One highlight for was how the setup is simple,
which does away with much tooling.
I.e., there are 3 files (`index.html`, `index.js`, `style.css`) to build upon.
And most times, the other additions are `App.js`, and two directories: `images` and ``components`.

To setup `index.html`

- so as to use functions from React and to understand [JSX](https://reactjs.org/docs/introducing-jsx.html) (a JavaScript extension that React uses),
we acess [React and ReactDOM via CDN](https://reactjs.org/docs/cdn-links.html).

- But the browser, which understands HTML, CSS, and JavaScript, also needs to understand JSX.
So, we need a JavaScript compiler ([Babel](https://babeljs.io/docs/en/babel-standalone.html)) that transforms JSX into JavaScript for the browser.
I'm using the standalone version, which works well in the browser without requiring extra tooling, like Webpack, Browserify or Gulp.

Now, `index.html` looks like:

```html
<html>
    <head>
        <link rel="stylesheet" href="style.css">
        <script crossorigin src="https://unpkg.com/react@17/umd/react.development.js"></script>
        <script crossorigin src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
        <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
    </head>
    <body>
        <div id="root"></div>
        <script src="App.js" type="text/babel"></script>
        <script src="index.js" type="text/babel"></script>
    </body>
</html>
```

But I ran into a couple of issues:

1. When it comes to reusing modules in other files/modules via export and import, I bump into this error `ReferenceError: require is not defined.`

2. Some JavaScript expressions, like [`spread`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) don't work. The browser's devtools' console shows

```javascript
...
Uncaught SyntaxError: http://127.0.0.1:5500/tutorial/App.js: Unexpected token (16:12)
...
```

To fix those issues, we need to tweak the script tags for our scripts.

## Keep script type as Babel and enable module import/export

### Problem

The example below shows `index.js` importing the `App` function in order to use it,
while `App.js` defines and exports an `App` function.

`index.js` looks like:

```jsx
// index.js
import App from "./App"

ReactDOM.render(<App/>, document.getElementById("root"))
```

`App.js` looks like:

```jsx
// App.js
export default function App() {
    return (
        <h1>Hello World!</h1>
    )
}
```

To make `App.js` into a [native ES module](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#applying_the_module_to_your_html),
we'd normally set the `type="module"`attribute on the script tag, like this:

```html
<html>
    <head>...</head>
    <body>
        <div id="root"></div>
        <script src="App.js" type="module"></script>
    </body>
</html>
```

But `App.js` contains JSX that the browser doesn't understand.
Babel, a JavaScript compiler, has to transform that to JavaScript, which the browser understands.
For that, we have to keep the `type="text/babel"` as an attribute of the script tag.

```html
<html>
    <head>...</head>
    <body>
        <div id="root"></div>
        <script src="App.js" type="text/babel"></script>
    </body>
</html>
```

The question remains: **how do we keep `App.js` type as `text/babel` while making it a module that can export functions?**

### Solution

This [stackoverflow question](https://stackoverflow.com/questions/63883713/using-es-modules-with-babel-standalone) hinted at the solution: 

> The only way around this I found was to use the transform-modules-umd plugin and include all my js files as scripts.

How do I add that plugin to `index.html` ?

I went to Babel's [CDN](https://unpkg.com/babel-standalone@6.26.0/babel.js) (take it from the `head` html element) and searched for the name `transform-modules-umd`.
That confirmed that name is right.
It has a corresponding [npm package](https://www.npmjs.com/package/babel-plugin-transform-es2015-modules-umd).

Because `index.js` is importing the `App` function, `index.js` also needs the same plugin.

Now the script tag has an additional `data-plugins` attribute.

```html
<html>
    <head>
        <link rel="stylesheet" href="style.css">
        <script crossorigin src="https://unpkg.com/react@17/umd/react.development.js"></script>
        <script crossorigin src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
        <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
    </head>
    <body>
        <div id="root"></div>
        <script src="App.js" type="text/babel" data-plugins="transform-es2015-modules-umd"></script>
        <script src="index.js" type="text/babel" data-plugins="transform-es2015-modules-umd"></script>
    </body>
</html>
```

## JavaScript `spread` syntax in Babel

### Problem

In the lesson of using React's `useState` to tell React to change something and display it in the browser,
we learnt that,
when the state is complex (we keep arrays or objects in state),
and
when we need to use an old version of state to determine the new value of state,
it's good practice to use JavaScript's [`spread` expression](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax).

The `spread` syntax (`...prevObj`) in our `App.js` looks like:

```jsx
// App.js
export default function App() {    
    // object saved in state
    const [objState, setObjState] = React.useState({
        firstName: "Jane",
        lastName: "Doe"
    })

    // called when text clicked on
    function handleClick() {

        setObjState(function(prevObj) {
            const randomNumber = Math.floor(Math.random() * 100)
            const newLastName = prevObj.lastName + "-" + randomNumber
            // uses spread syntax
            return {...prevObj, lastName: newLastName}
        })
    }

    return (
        <div>
            <h2 onClick={handleClick}>
                Click me to add suffix to previous last name
            </h2>
            <div>
                <h3>{objState.lastName}</h3>
            </div>
        </div>
    )
}
```

When running that, we get

```javascript
Uncaught SyntaxError: http://127.0.0.1:5500/til-state-with-spread/App.js: Unexpected token (13:20) (at babel.min.js:7:10099)
  11 |             const newLastName = prevObj.lastName + "-" + randomNumber
  12 |             
> 13 |             return {...prevObj, lastName: newLastName}
     |                     ^
  14 |         })
  15 | 
  16 |     }
    at J.raise (babel.min.js:7:10099)
    at X.unexpected (babel.min.js:5:27476)
```

### Solution

This [stackoverflow question](https://stackoverflow.com/questions/56890304/how-to-use-babel-standalone-to-transpile-es6-javascript-that-uses-the-spread-res) hinted at the solution:

> might you also need `"@babel/plugin-transform-spread"` to capture spreads in an array, since `"@babel/plugin-proposal-object-rest-spread"` is object-specific.

Go to Babel's [CDN](https://unpkg.com/babel-standalone@6.26.0/babel.js), search for `object-rest-spread`, and add it to the `data-plugins` attribute of the script tag.

```html
<html>
    <head>
        <link rel="stylesheet" href="style.css">
        <script crossorigin src="https://unpkg.com/react@17/umd/react.development.js"></script>
        <script crossorigin src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
        <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
    </head>
    <body>
        <div id="root"></div>
        <script 
            src="App.js" type="text/babel" 
            data-plugins="transform-es2015-modules-umd,transform-object-rest-spread">
        </script>
        <script src="index.js" type="text/babel" data-plugins="transform-es2015-modules-umd">

        </script>
    </body>
</html>
```

## References

What I came across that pointed me in the right direction.

- <https://stackoverflow.com/questions/63883713/using-es-modules-with-babel-standalone>
- What does `umd` suffix in Babel's plugin `transform-modules-umd` mean? See <https://github.com/umdjs/umd>
- <https://stackoverflow.com/questions/56890304/how-to-use-babel-standalone-to-transpile-es6-javascript-that-uses-the-spread-res>
- ECMAScript language specification - <https://tc39.es/ecma262/multipage/#sec-intro>