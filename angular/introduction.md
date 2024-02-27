- [What is Angular](#what-is-angular)
- [Prerequisites](#prerequisites)
- [Tooling and preparation](#tooling-and-preparation)
- [Hello World](#hello-world)
- [File structure](#file-structure)
- [Architecture](#architecture)
- [Components](#components)
  - [how to generate a new component:](#how-to-generate-a-new-component)
  - [how to insert a component into a module](#how-to-insert-a-component-into-a-module)

# What is Angular
Angular is a modular framework for frontend development. 
# Prerequisites
- Javascript
- Typescript
- HTML
- CSS
# Tooling and preparation
Install:
- Node.Js
- Npm
- Agular CLI
# Hello World
create a new angluar project by running
```
ng new helloworld
```
Now you can start the default angular project by running 
```
cd helloworld; ng serve
```
pressing `o`+`enter` will open the project in your default browser. It should say something like `Congratulations! Your app is running. `
# File structure
When creating the hello world project, `ng` creates a directory called helloworld(or whatever name you gave to your project). This directory contains a bunch of files. We will mainly focus on the src-subdirectory. Here, we find main.ts, where the execution starts. 

# Architecture
Angular is contains several modules, and each module contains several components TODO insert image

# Components
Every angular component is build like its own mini-website: It contains an html, a css and a typeScript File (It also contains a `.spec.ts`-file, but we can ignore this for now.)

In order to understand this better, we will have a look at the app-component. You can find it in helloworld/src/app. Open `app.components.ts` to see how a component normally looks like. We will focus on the decorator (The thing that starts with @). It should look something like this:
```@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
```
`selector` is the name of the custom html-tag you can use to insert a component into html.

`templateUrl` is the html-file of this component. Try to edit it and see what happens!

`styleUrl` is the css-file. It should be empty for now, but you can fill it with css-tags if you need.

## how to generate a new component:
Decide on a name for your new component, then run 
```ng generate component [name] ```
Now, a new folder is created inside the app-folder called [name]. It contains the ts, css and html-files of the component:
- [name].component.html
- [name].component.css
- [name].component.ts
- [name].component.spec.ts

The html-file should only contain the text `<p>[name] works!</p>`. We can insert this into the main html-file (app.component.html) in any place we want. Let's try this!

## how to insert a component into a module
Step 1: import it into our app.component.ts-file as a typescript class. It should look something like this:

`import { [name] } from './[name]/[name].component';`[^1]


Step 2: add it to the import list from the component. It will look something like this:

`imports: [RouterOutlet, name],`

Step 3: `[name].component.html` is represented by a custom tag called `app-[name]` [^2] which can be imported into `app.component.html` at any place(s) we want.

So we need to add `<app-name></app-name>` to a proper place in the html-file.  The statement `[name] works!` should now be displayed in the browser. You can also try to change the content of `[name].component.html`, which should then be updated in the browser.

Instead of inserting as an html-tag, there are 2 more ways to do this:
- prepending the selector with a dot, which will turn it into an html-class
- enclosing it with square brackets, which will turn it into an html-attribute

[^1]: Of course you can use any name you want for the tyescript class.

[^2]: If you want, you can change the name of the html-tag in `[name].component.ts.Component.selector` 