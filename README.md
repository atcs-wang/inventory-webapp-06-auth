# Part 07: Authentication and Authorization  

This tutorial follows after:
[Part 06: First Deployment](https://github.com/atcs-wang/inventory-webapp-05-handling-forms-post-crud)



Some more detailed "next steps" for using Auth0 w/ Node/Express here: [https://github.com/auth0/express-openid-connect/blob/master/EXAMPLES.md#2-require-authentication-for-specific-routes](https://github.com/auth0/express-openid-connect/blob/master/EXAMPLES.md#2-require-authentication-for-specific-routes)


# (7.0) Get started with Auth0

Go to [https://auth0.com/](https://auth0.com/) and create an account.

Once logged into, you can follow the `Getting Started -> Create Application` wizard. 
Choose:
  * At  `Create Application`, give your app a Name and choose `Regular Web Applications` as your application type.

  * At  `What technology are you using for your project`, choose `Node.js (Express)`

  * At  `Choose your path`, click `Integrate Now` beneath `I want to integrate with my app`

  * At  `Configure Auth0`, the `Allowed Callback URL` box should contain `http://localhost:3000/callback` and the `Allowed Logout URLs` box should contain `http://localhost:3000`. Change the number from `3000` to whatever your PORT value is during development. (Mine is 8080)   

  * At `Integrate Auth0`, take the command 

    > `npm install express express-openid-connect --save`

    and run it in your Terminal. This will install the `express-opendid-connect` package.

    Then look at the code provided in the `Configure Router` section, which is already configured with your information. It contains 4 "lines", which we will add to our `app.js`
    
    * Copy the first `const { auth } = require('express-openid-connect');` line and add it among the other `require` statements at the top of the file.

    * Copy the `const config=...` and `app.use(auth(config))` lines except for the last section and add it after the helmet , before any of the other `app.use` lines. 
    
    * Then add the last line `app.get('/',...);` after the other `app.use` lines, but change the route path from `/` to `/authtest`.

    The added code, in context, should look like this:

    ```js
    //Other require lines
    ...
    const { auth } = require('express-openid-connect');
    ...

    // Helmet middleware
    app.use(helmet({...
    });

    // CODE FROM AUTH0:
    const config = {
      authRequired: false,
      auth0Logout: true,
      secret: 'a long, randomly-generated string stored in env',
      baseURL: 'http://localhost:<PORT>',
      clientID: '<YOUR CLIENT ID>',
      issuerBaseURL: 'https://<YOUR DOMAIN>.us.auth0.com'
    };

    // auth router attaches /login, /logout, and /callback routes to the baseURL
    app.use(auth(config));

    // Rest of the app.use(...) middleware
    ...

    // req.isAuthenticated is provided from the auth router
    app.get('/authtest', (req, res) => {
      res.send(req.oidc.isAuthenticated() ? 'Logged in' : 'Logged out');
    });

    // Rest of the code, e.g. routes
    ...
    ``` 

    Now go to the next section.
  
  * At `Test your login`, start up your server with `npm start`
    * Follow the instructions to visit the `/login` and `/logout` routes. Auth0's monitoring should show activity. 

    *  You should also visit `/authtest` after both logging in and logging out to see that the webapp is aware of your logged-in/out status.

  *  At `Get the user profile information`, copy the code provided, and add them to `app.js`:
    * The `const { requiresAuth } = require('express-openid-connect');` can be added to the top of the file.

    * The other line can be added anywhere after the `app.use(auth(config))` line, among the routing code.

    The added code, in context, should look like this:

    ```js
    //Other require lines
    ...
    const { requiresAuth } = require('express-openid-connect');

    //Other code including app.use(auth(config))
    ...

    app.get('/profile', requiresAuth(), (req, res) => {
      res.send(JSON.stringify(req.oidc.user));
    });

    // Rest of the code, e.g. routes
    ...

    ```
    * Restart your server and visit the `/profile` route, both logged in and logged out.
  * The wizard should be complete, and you can click `Go To Application Settings`.


## (7.2) Securing Auth0 secrets in `.env`, setting up for deployment

Our authentication basics work for our development code, but a few things need to be fixed up before we can push any changes and (re-)deploy our app.

The `config` object in `app.js` contains all the Auth0 info in plaintext. These should be kept secret via environment variables.

1. Add these lines to your `.env` file, replacing the values with the same values used in your `config` object.

```
AUTH0_SECRET=<SECRET>
AUTH0_BASE_URL=http://localhost:<PORT>
AUTH0_CLIENT_ID=<YOUR CLIENT ID>
AUTH0_ISSUER_BASE_URL=https://<YOUR DOMAIN>.us.auth0.com
```

The `<SECRET>` can technically be replaced with anything, but Auth0's `Application Settings` page provides a good `Client Secret` that you can copy and use.

2. Add this in the top section (among the `requires`) to your `app.js`:
  ```js
  const dotenv = require('dotenv');
  dotenv.config();
  ```

3. Change the `config` object in `app.js` to: 
  ```js
  const config = {
    authRequired: false,
    auth0Logout: true,
    secret: process.env.AUTH0_SECRET,
    baseURL: process.env.AUTH0_BASE_URL,
    clientID: process.env.AUTH0_CLIENT_ID,
    issuerBaseURL: process.env.AUTH0_ISSUER_BASE_URL
  };
  ```

Double check that your server is still able to do `/login` and `/logout` successfully.

Now you can push the code without worrying about exposing the secrets. 

However, if you have a deployed app, we need it to be given those environment variables too. 

The only one that needs to be changed for the deployed environment is the `AUTH0_BASE_URL`, which should be the URL of the actual hosted site, not `localhost`.

Then, you need to update the `Application Settings` on Auth0: In the `Application URIs` section, change the `Allowed Callback URL` box from just `http://localhost:PORT/callback` to :
  ```
    http://localhost:PORT/callback,
    https://<HOSTDOMAIN>/callback
  ```
  and the `Allowed Logout URLs` box from just `http://localhost:PORT` to:

  ```
    http://localhost:PORT,
    https://<HOSTDOMAIN>
  ``` 

Save your changes.
## (7.3) Adding a log-in / log-out button
# TODO

We obviously need a way to log in and out that's more intuitive for the user than changing the URL manually. 

We can easily add Login and Logout buttons to our pages, which are just links to `/login` and `/logout` 
```html
    <a href="/login" class="btn blue">Login</a></li>
    <a href="/logout" class="btn red">Logout</a></li>
```

They would look better on a navigation bar that is consistent at the top of the each page.
> We have neglected to make a nav bar thus far in our project. In retrospect, we should have just added one at the prototype stage. 
Materialize has some nice support for good looking nav bars, which can collapse for small screens like mobile devices, and provide an alternative side menu nav.

Go ahead and add one into each of our views at the top of the body... 

```html
...
</head>
<body>
    <!-- Nav bar -->
    <header>
        <nav>
            <div class="nav-wrapper">
                <a href="/" class="brand-logo">Stuff Manager</a>
                <a href="#" data-target="mobile-nav" class="sidenav-trigger"><i class="material-icons">menu</i></a>
                <!-- Main nav, hidden for small screens -->
                <ul id="desktop-nav" class="right hide-on-med-and-down">
                    <li><a href="/"><i class="material-icons left">home</i>Home</a></li>
                    <li><a href="/stuff"><i class="material-icons left">list</i>Inventory</a></li>
                    <li><a href="/login" class="btn blue">Login</a></li>
                    <li><a href="/logout" class="btn red">Logout</a></li>
                </ul>
            </div>
        </nav>
        <!-- Mobile nav menu, shown when menu button clicked -->
        <ul id="mobile-nav" class="sidenav">
            <li><a href="/"><i class="material-icons left">home</i>Home</a></li>
            <li><a href="/stuff"><i class="material-icons left">list</i>Inventory</a></li>
            <li><a href="/login" class="btn blue">Login</a></li>
            <li><a href="/logout" class="btn red">Logout</a></li>
        </ul>
    </header>

    <!-- rest of the body -->
    <div class="container">
      ...
```

> If not already included, add Materilize source and initialization scripts, which are needed for the mobile side-nav to work.
```html
    <!-- Materialize JavaScript -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js"></script>

    <!-- initialize Materialize elements -->
    <script src="/scripts/materializeAutoinit.js"></script>

```

You can test the buttons to make sure they work.

## (7.4) Changing view on authentication; displaying user information

However, we'd only want to show one of the two buttons at a time - Login if not yet authenticated, and Logout if already authenticated. We might also like to show, if you are logged in, some of your user information, making it clear who you're logged in as.

We've seen that the state of auth is included with all incoming requests via `req.oidc`, and that it could be used to determine how we respond.  The `/authtest` route from above demonstrated the `req.oidc.isAuthenticated()` method which tells us whether we are logged in, and the `/profile` route demonstrated `req.oidc.user`, which has user info included with the auth token.

We want our server to provide the state of authentication to EJS at render time. 

One way of doing this is to pass it as part of the context parameter given at `res.render(...)`.
For example we could change the `res.render` in the `/stuff` route handler from this:

```js
app.get( "/stuff", ( req, res ) => {
    ...
            res.render('stuff', { inventory : results });
    ...
} );
```

to this:

```js
app.get( "/stuff", ( req, res ) => {
    ...
            res.render('stuff', { inventory : results , loggedIn : res.oidc.isAuthenticated });
    ...
} );
```

However, we'd have to do this for each GET route handler - tedious, and easy to miss one as our app grows.

A better way is to create a single middleware that, for all GET routes, will add a "local" property to the `res` response object, providing the information to EJS for all page renders.

Add this to the `app.js` after the other middleware (`app.use`) but before any route handlers (`app.get`, `app.post`, etc.)

```js
app.use((req, res, next) => {
    res.locals.isLoggedIn = req.oidc.isAuthenticated();
    res.locals.user = req.oidc.user;
    next();
})
```

Now, we can assume the availability of two variables in all of our `.ejs` files:
* `isLoggedIn`, a boolean indicating our authentication status
* `user`, which is either `undefined` if not logged in, or the user info object from the auth token. 

Let's use them to customize our nav bar view:

```html
<header>
        <nav>
            <div class="nav-wrapper">
                <a href="/" class="brand-logo">Stuff Manager</a>
                <a href="#" data-target="mobile-nav" class="sidenav-trigger"><i class="material-icons">menu</i></a>
                <ul id="desktop-nav" class="right hide-on-med-and-down">
                    <li><a href="/"><i class="material-icons left">home</i>Home</a></li>
                    <li><a href="/stuff"><i class="material-icons left">list</i>Inventory</a></li>
                    <% if (isLoggedIn) { %>
                        <li><a href="/profile"><i class="material-icons left">person</i> Hello, <%=user.name%></a> </li>
                        <li><a href="/logout" class="btn red">Logout</a></li>                        
                    <% } else { %>
                        <li><a href="/login" class="btn blue">Login</a></li>
                    <% } %>
                </ul>
            </div>
        </nav>
        <ul id="mobile-nav" class="sidenav">
             <li><a href="/"><i class="material-icons left">home</i>Home</a></li>
             <li><a href="/stuff"><i class="material-icons left">list</i>Inventory</a></li>
             <% if (isLoggedIn) { %>
                <li><a href="/profile"><i class="material-icons left">person</i> Hello, <%=user.name%></a> </li>
                <li><a href="/logout" class="btn red">Logout</a></li>                        
            <% } else { %>
                <li><a href="/login" class="btn blue">Login</a></li>
            <% } %>
        </ul>
    </header>
```

## (7.5) Authorization for specific routes
TODO
## (7.6) Associating inventory data with users
TODO
## (7.7) Keeping a user table with roles/permissions
TODO
## (7.8) Adding a user page for admins only

