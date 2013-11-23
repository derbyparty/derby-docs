## Environment

Let`s assume you have Linux Debian family OS (Ubuntu, Debian, etc.). Setting up the environment for other OSs: other Linux, Windows, Mac OS has sungularities, but is not completly different.  
To run the Derby application, we need: node.js, mongodb, redis (at least 2.6).

For node.js and redis, we will use the chris-lea repository, mongo has an official repo.

```
# Add the repository
# node.js
sudo add-apt-repository-y ppa: chris-lea/node.js
# mongodb
sudo apt-key adv - keyserver hkp :/ / keyserver.ubuntu.com: 80 - recv 7F0CEB10
sudo echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee / etc/apt/sources.list.d/10gen.list
# redis
sudo add-apt-repository-y ppa: chris-lea/redis-server

# Update the apt-get
sudo apt-get-y update

# Set
sudo apt-get-y install nodejs
sudo apt-get-y install mongodb-10gen
sudo apt-get-y install redis-server
```

## Application

We certainly can create application from scratch now.  
But Derby has tool that generates for us the application layout and saves time. Why do not we use it?  
First we need to install derby npm package globally:

```
sudo npm install -g derby
```

Let`s create an application called hello-derby (this is also the name of the folder):

```
derby new hello-derby
```

Tool creates the application and installs all dependencies. It will take some time and in the end you will see:

```
  Project created!

  Try it out:
    $ Cd hello-derby
    $ Npm start

  More info at: http://derbyjs.com/
```

We will examine Javascript application, but if you want Coffeescript, use - coffee,-c:

```
derby new -coffee my-cool-coffee-derby-app
```

You can also create an bare application (bare layout only):

```
derby bare my-bare-app
```

Or create an application, but does not install dependencies using - noinstall,-n:

```
derby new -n empty-node_modules-app
```

## Run

To start (you guessed it) you need:

```
cd hello-derby
npm start
```

See:

```
1234 listening. Go to: http://localhost:3000/
```

Now in your browser go here: http://localhost:3000/  
Hooray, it works!

## Structure

Let's quickly go over the project structure.

/lib - here is almost all js. If you are on the coffee, it will be a folder /src.  
/lib/app - this is a client application called 'app'. Here maybe some of them. It runs on client and server.  
/lib/app/index.js - here app itself is created, and two component libraries are added: 'derby-ui-boot' (bootstrap for derby) and 'ui' (some components in our Derby app). Then there are routes that will be executed on the server and on the client. At the end controller functions are created - functions that are executed only on the client and are associated with the manipulation of dom.  
/lib/server - server application. Can be only one. Code runs only on server and is not directly accessible form client.  
/lib/server/error.js - here we generate some custom static (only html and css, without the client application) error pages.  
/lib/server/index.js - this creates Express application, configures databases, creates store, adds Connect modules to Express app, some of which are parts of Derby application. At the end it creates a server-side Express route, which generates an error for requests that were not catched by client app router or server (Express) router.  
/node_modules - npm packages.  
/styles - styles are here. Default engine is Stylus (Less and css are supported also).  
/styles/app - styles that will be uploaded to client with client application named 'app'.  
/styles/ui - styles for ui component library.  
/ui - component library. Each component consists of js and html files.  
/ui/connectionAlert - an example of a component. If the client went offline, this component displays the label and button «Reconnect». If reconnect failed, it offers to restart the application «Reload».  
/ui/index.js - ui component library settings.  
/views - html templates.  
/views/app - templates that are loaded into client app.  
/views/app/home.html - home page.  
/views/app/index.html - layout for home.html and list.html.  
/views/app/list.html - list page.  
/views/error - templates for the static error pages for /lib/server/error.js  
.npmignore - you will need it if you publish your application as a package to npm.  
Procfile - this is for Heroku.  
README.md - read me  
package.json - setting for npm: which modules download on npm install, what to do on npm start, etc.  
server.js - main file. The entry point of your application. Derby starts Express application here.







