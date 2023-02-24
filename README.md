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
TODO
## (7.4) Displaying user information
TODO
## (7.5) Authorization for specific routes
TODO
## (7.6) Associating inventory data with users
TODO
## (7.7) Keeping a user table with roles/permissions
TODO
## (7.8) Adding a user page for admins only

