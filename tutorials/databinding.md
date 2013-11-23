## Data binding

First let`s create and run bare Derby application:

```
derby bare habr
cd habr
npm start
```

Type in the browser: http://localhost:3000/

Can you see lable 'Bare'?  
What just happened? Your request reached server, where it was processed by all Connect middleware from /lib/server/index.js, until:

```
.use(app.router())
```

Which is client app router from app/lib/app/index.js:

```
app.get('/', function (page) {
  page.render();
});
```

Here for the path '/' we generate html from template /views/app/index.html. Why index.html? This is by default. Nothing will change if we change to:

```
app.get('/', function (page) {
  page.render('index');
});
```

In index.html we have only section: Body. Also can be: Head, Header, Footer, Scripts, Title, etc. Derby template engine generating html, will find Body section and will put it`s value in the appropriate place, and instead of the other sections (since they are not set), it will put an empty or default values​​. As a result, next html will be returned back to the client:

```
<! DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title></title>
    <style id="$_css"></style>
  </head>
  <!-$_page->
  <body>
    Bare
    <!-$$_page->
    <script defer="" async="" src="/derby/lib-app-index.js"></script>
  </body>
</html>
```

What can we see ? Title is empty. No styles. Head - also almost empty. As we might expect. And there is something in the body.  
All styles from /styles folder will be combined, compressed and showed inline. You can put links to css files to Head section (for example) if you want connect style files.  
Script /derby/lib-app-index.js - this is our client application. As soon as it will be loaded to client it will take everything in it`s hands. So if you change url, client app router will catch it in browser and Derby template engine will generate html also in browser.  
<!-$_page-> and <!-$$_page-> - Derby uses html comments like this as internal tags for dynamic binding between data and html. If data changes, only part of html will be regenerated, not entire page. In this case, when router triggered, only Body section will be updated, because we have not set any other section and they all same for all pages.

You can change text in the templates (and styles) and see the changes instantly in your browser. It has no relation to any dynamic data synchronization. This is for the development convenience. If you change the html and css, it will be automatically compiled, uploaded to the client and replaced the old one. If you change js, application restarts.

Let's separate view from data. For this purpose, there are two ways in Derby. Let's start with Context. This is object, which we can be added as next (after template name) argument in page.render() and the data from which we can display in html.

```
app.get('/', function (page) {
  page.render({text: 'text from Context'});
});
```

```
<Body:>
  {{text}}
```

Double '{{ }}' template brackets mean that html is not dynamically binded to data. Derby template engine does not track html or data changes. It`s not necessary for Context, because Context - just js-object and does not change, therefore usually use-case for Context is static pages. For dynamic app use Model.

Model is data manipulation api-object. Also it stores some state inside itself.  
Where to get the Model? There are many ways (depending on whether you are on the client or on the server), but in the router it goes the second argument (after page) and it is very convenient for us in this situation :

```
app.get('/', function (page, model) {
  page.render();
});
```

Let's put data into Model:

```
app.get('/', function (page, model) {
  model.set('_page.text', 'text in model');
  page.render();
});
```

_page.text - this is path in the model, where our text will be stored. Path corresponds to json. In this case :

```
var obj = model.get('_page');
// obj === {text: 'text in model'}
```

Underscore means that this path is local. That is, it exists only in this Model. And is not synchronized with the database and other Models (other clients). You can create any local path, but _page object is a little bit special. It cleans every time router is triggered, so it is convenient to store data associated with the page in the _page path.  
In order to see the data from the model, change the template:

```
<Body:>
  {{_page.text}}
```

Let's try to make dynamic binding between data and html:

```
<Body:>
  <input type="text" value={_page.text} />
  {_page.text}
```

Single brackets {} mean that the data in the Model is dynamically binded to html. If you change the value in the input, it will change the value in the Model, which in turn changes the text next to the input. That's an example of two-side dynamic binding between data and html.

Well, there was not anything complex? 3 lines of code? It`s not serious?  
Let's make a really serious thing! Now we will create a web application, whose clients are synchronized with each other. We change the html on one client, this dynamically changes the data on the client, then the data flyes to the server, where conflict resolution algorithm merges data to database and send it to all clients that are subscribed to it, finally on each client data will be converted into html. Suitable? How long does it take to write this using your favorite (before Derby) framework? How many lines of code?

```
app.get('/', function (page, model) {
  model.subscribe('page.text', function (err) {
    if (!model.get('page.text')) {
      model.set('page.text', 'text in model');
    }
    page.render();
  });
});
```

```
<Body:>
  <input type="text" value={page.text} />
  {page.text}
```

That's it? o_O Well, in general, yes. Open http://localhost:3000/ in several browser windows and play with it for a while.

page.text - this is remote path. Unlike the local path, it points to the database. In this case, we created 'page' collection and object with id 'text'. In real life remote paths look like: 'users.8ddd02f1-b82d-4a9c-9253-fe5b3b86ef41.name', 'customers.8ddd02f1-b82d-4a9c-9253-fe5b3b86ef41.properties.isLead', 'products.8ddd02f1-b82d-4a9c-9253-fe5b3b86ef41.prices.1.value'.  
model.subscribe - this is way we subscribe client for path 'page.text'. If any other client will change data in the path 'page.text', server send us new version of data.