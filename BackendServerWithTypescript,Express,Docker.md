# How to Make a Backend Server using Typescript, Express, and Docker
You can use any editor you want, but for this tutorial I will be using VSCode.

## Step 1: initializing the project
First, we're going to initialize a new node project.

Create a new folder called MyServer and open it in VSCode.

Open a terminal and navigate to the folder.
* If you're using VSCode, you can open a new terminal in the "Terminal" menu in the title bar, or with the shortcut Ctrl+Shift+`.
* If you're using Webstorm or Intellij, you can open a new terminal by clicking the "Terminal" menu on the bottom bar or with the shortcut Alt+f12.
* If you're using a plain text editor, you can open a new cmd and typing `cd [path-to-your-folder]`.

Run the command `npm init`. This will take you through initializing a new node project. Click enter to continue through the setup, and at the end you should see something like this:
```
{
    "name": "myserver",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC"
}

Is this OK? (yes)
```
Press enter again to finish the process.
4. Open up the `package.json` file that was just created, and add the following lines to "scripts"<br>
`"start": "node dist/server.js",`<br>
`"build": "docker build . -t myserver",`<br>
`"prebuild": "npx tsc",`<br>
`"deploy": "docker run -it --rm -p 3000:3000 myserver"`

Then, delete the `"test"` script. Your `"scripts"` section should now look like this:
```json
"scripts": {
    "start": "node dist/server.js",
    "build": "docker build . -t MyServer",
    "prebuild": "npx tsc",
    "deploy": "docker run -it --rm -p 3000:3000 myserver"
}
```

## Step 2: install npm packages
Next, we need to install some npm packages in order for our project to work. An npm package or module is a public (or private) building block that someone else published to make our lives easier. Instead of having to write a ton of boilerplate to get a simple server working, we can use npm packages to make things a lot simpler.

In terminal, run the following commands:<br>
`npm install typescript express`<br>
`npm install --save-dev @tsconfig/node18 @types/express`<br>
`typescript` and `@tsconfig/node18` will allow us to use TypeScript to write our code instead of JavaScript, and `express` is a package that makes creating APIs very simple. 

## Step 3: scaffold our project and add configuration files
There are many different ways you can scaffold your project, but this is usually how I do it and how I see it done in other projects.

Create a folder called `src` and a folder called `dist` in the `MyServer` folder. Now, the project structure should look like this:
```
├── MyServer
│   ├── dist
│   ├── node_modules
│   ├── src
│   ├── package-lock.json
│   ├── package.json
```

## Step 4: initialize git repository (optional)
If you plan on using a git repository or another source control now would be a good time to initialize it.

In terminal, run the command `git init`. This will create an empty git repository.

Create a file called `.gitignore` in the MyServer folder. Inside this file, add the following lines:

```
node_modules
dist
```
You can also add any other files you don't want to include in your git repo.

Track all the files we've created using the command `git add .`.

Create your first commit using the command `git commit -m "initial commit"`. You can replace the commit message with whatever you want.

## Step 5: add configuration files
Because we're using TypeScript, we need to create a tsconfig file to tell the TypeScript engine how to compile our code.

Create a file called `tsconfig.json`. In this file, add the following code:
```json
{
    "extends": "@tsconfig/node18/tsconfig.json",
    "include": [
        "src/**/*"
    ],
    "exclude": [
        "node_modules"
    ],
    "compilerOptions": {
        "outDir": "dist",
        "lib": [
            "ES6",
            "DOM"
        ]
    }
}
```
By modifying this tsconfig file, you can change the configuration of the TypeScript compiler for your project.

Create a file called `Dockerfile`. This is the "instructions" that tell docker how to create an image from your code. In it, add the following code:
```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install --omit=dev
COPY . .
CMD npm start
```
In this Dockerfile, we first install the base Node.js@18 image, and set the working directory to `/app`. In this directory, we add the `package.json` and `package-lock.json` files that specify what npm packages we're using. Next, we install the npm packages, copy our `dist` folder over, and start the server. Finally, we expose port 3000 so we can access it from our local machine.

Create a file called `.dockerignore`. In this file, we will add any files or folders that we do not want to be copied to our docker image. In it, add the following code:
```
node_modules
src

.gitignore
tsconfig.json

**/*.ts
```
This will prevent these files and folders from being copied over to our docker image, which will cause our image size to be smaller and our build step to be faster.

The line `**/*.ts` means "ignore any file in any folder that ends in ".ts".

## Step 6: creating our server
Create a file called `server.ts` in the `src` folder. This is where we will put the code for our server. 

At the top of our `server.ts` file, add the following code:
```ts
import express from 'express';
import http from 'http';
```
This will import the some of the packages we need to run our server.
    
Next, we need to create an express app. To do this, add the following lines under the code we just added:
```ts
const app = express();
const server = http.createServer(app);
```
In this code, `app` is an express object, which is request listener, and `server` is the object that will listen for requests and pass them to the request listener.

Next, we will make our first endpoint using the following lines:
```ts
app.get('/', async (req, res) => {
    res.send("Hello world");
});

server.listen(3000, () => console.log('server listening on', 3000));
```
This will create an endpoint at `/`, and then turns on the server on port 3000.
### Quick Checkpoint
By now, the file structure of your project should look like this:
```
├── MyServer
│   ├── dist
│   ├── node_modules
│   ├── src
│   │   ├── server.ts
│   ├── .dockerignore
│   ├── .gitignore
│   ├── Dockerfile
│   ├── package-lock.json
│   ├── package.json
│   ├── tsconfig.json
```

## Step 7: testing our server
Run the command `npm run build`. This will first compile your .ts code into .js code, and then build the docker image.

Then, run the command `npm run deploy` to run the container.

Finally, navigate to `http://localhost:3000` on a web browser, and you should be greeted with "Hello world".

To stop the server, use the shortcut Ctrl+c while in the terminal.

## Step 8: adding more routes
Now, you can add more routes similarly to how we created the `/` route.

Remember, each time you update the code you will have to rebuild it before deploying.