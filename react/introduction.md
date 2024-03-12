# What is React?
React is a JavaScript libaray with which we can create SPA-websites.
# Prerequisites
HTML, CSS and JavaScript
# Tooling and preparation
Node.js
# Architecture
## File structure
A react project consists of the following folders:
- node_modules: all dependencies are stored here. If you want to change them, do not tamper with this folder, but change `package.json`, then run `npm install`.
- public: contains index.html, which is the start page for our app.
- src: source folder for react code
Note: In normal projects, we will only ever touch the src-folder and ignore node_modues and public.
## Components
A react website consists of several components that are arranged in a tree. A component consists of a template and logic.
# Hello World
you can create a react project by running 
```bash
npx create-react-app hello-react
```
This command should create a new folder called hello-react. Change to this folder and launch the website by running
```sh
npm run start
```
This should open your browser at `http://localhost:3000` and show this image:
![atom](startpage.png)
You can now change the html-code [^1] of `src/App.js`. Right now it looks complicated, but we could reduce most of the code to this:
```jsx
function App() //every component is defined in a JS-function
{
  //here you can add java script code if you need it
  return (
    <div className="App">
      <div>
        <h1>Hello world</h1>
      </div>
    </div>
  );
}
export default App;//make our function available to the ouside world
```
Save the file and head over to your browser (Saving the file is enough, no rebuilding required). It should have been updated to show just `Hello world` and nothing else.

[^1]: Technically it's jsx-code, not html. More on that later.

# First steps
## What is JSX?
So far, we only used App.js, which acts as the root of the component tree. Now let's add a component to App.js as a child. For this, we use **jsx**. jsx is basically JavaScript code extended with HTML. You can write HTML code in return statements, which you can then extend by JavaScript expressions enclosed in curly braces.

What does that mean in practise? Loot at line 4 of the code above. There, we could generate a random number called `rand`.[^2] We can now insert it into the HTML code below `Hello world!` like this:
```jsx
<p>The generated random number is: {rand}</p>
```
Your browser should now show a random number that changes on every reload.

[^2]: How do we generate random numbers in JavaScript? Google that yourself!
## How to build the component tree
Like `App.js`, every component must export a jsx-function that returns HTML. This exported function can then be imported by other components and inserted into their HTML as custom tags.

Let's try this! A simple component creating a Button and a text could look like this:
```jsx
//function name must start with uppercase
const SimpleComponent = () => {
    return ( 
        <div>
            <button>Click here!</button>
            <p>Button was not clicked</p>
        </div>
     );
}
export default SimpleComponent;
```
In App.js, we can import it by
```js
import SimpleComponent from './whatevernameyougaveyourfile';
```
and insert it into HTML as a self-closing tag at a suitable place [^3] like this:
```html
<SimpleComponent/>
```
Like mentioned above, react components are a tree. So you can create another component and add it to SimpleComponent the same way we added SimpleComponent to App.js.

[^3]: In fact, you can insert it in several different places
## How to handle events
Modern websites cannot function without events. You can interact with it by typing, hovering, clicking etc. In most cases, the website should not just change some internal variables, but actually **react** to the event and change its appearance somehow. In order for that to happen, we need to specify which elements of the website might change. We do this by creating them as **UseState**-objects.

So how does this work? We will now create a component that conists of a button and a text that is updated once the button is clicked. We can reuse our SimpleComponent from above.

First, we need to tell the button, which function to run when clicked:
```jsx
<button onClick={handleClick}>Click here!</button>
```
The function `handleClick`, of course, does not yet exist, so this will throw an error. To solve this, create the function `handleClick`. This function could log something to the console in the function, so you can easily check if it works.

Now we can add a string after the button indicating that the button was clicked:
```jsx
<p>{clicked}</p>
```
This, of course, will also throw an error, since we have not yet defined the variable `clicked`.

So how do we do that? Well, clicked must be a stateful variable, so we need to import the useState on top of our file:
```jsx
import {useState} from 'react'
```
Now we can define our useState in the component function like this:
```jsx
const [clicked, setClicked] = useState("initial value")
```
Ok, what is happening here? Well, useState returns 2 objects: clicked is the String we can change, and setClicked is a setter function. Whenever we want to change `clicked` we msut do so by calling `setClicked`. So all we need to do now is edit `handleClick` so it calls `setClicked` to update its value. Now you should see a reaction.

This is the whole component code:
```jsx
import {useState} from 'react'
const SimpleComponent = () => {
    const [clicked, setClicked] = useState("Button was not clicked")
    const handleClick=() => {
        setClicked("Congratulations! You're able to click a button!")
    }
    return ( 
        <div>
            <button onClick={handleClick}>Click here!</button>
            <p>{clicked}</p>
        </div>
     );
}
export default SimpleComponent;
```