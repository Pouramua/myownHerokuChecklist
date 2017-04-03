# Deployment Checklist

## Heroku setup

- [ ] Create a Heroku account.
- [ ] Install the Heroku Toolbelt.
- [ ] Now login to Heroku on your command line with `heroku login`.


## Setup checklist

- [ ] Heroku sets a dynamic port which you can access with the port environment variable using `process.env.PORT`. If you are explicitly stating your port in your app without using the environment, Heroku can't expose your app. A good practice is to listen on a dynamic port or default to a local one: eg make sure your server/index.js has the code below in it.
```js
var port = process.env.PORT || 3000
```

*If you have a database:*

- [ ] You have a `production` option in the config file (if you are using knex this will be in the `./knexfile.js`).
- [ ] Install Postgres `npm install pg --save`.
- [ ] You are using `process.env.NODE_ENV` to dynamically choose the Knex environment. For example, in the module where you're using Knex.js (e.g. `db.js`):

  ```js
  var environment = process.env.NODE_ENV || 'development'
  var config = require('./knexfile')[environment]
  var db = require('knex')(config)
  ```

- [ ] You have defined the structure of your database with some migrations and they run locally without error. Example: npm run knex migrate:latest
- [ ] If you are seeding your database, the seed files run locally without error. Example: npm run knex seed:run
- [ ] You have configured the production database connection. We need to make sure Knex has the correct configuration for connecting to the Postgres database in the production environment. We do this in the `knexfile.js` in the `production.connection` property. The `DATABASE_URL` environment variable will be provided by Heroku and contain all the information Knex needs to make the connection. Example: paste the code below into your knexfile and overide the old production with the one below.

  ```js
  production: {
    client: 'postgresql',
    connection: process.env.DATABASE_URL,
    pool: {
      min: 2,
      max: 10
    },
    migrations: {
      tableName: 'knex_migrations'
    }
  }
  ```

- [ ] You are applying the migrations as the last step of deployment. We need our migrations to run after Heroku runs `npm install`. To do this, we add an npm script called `postinstall` to run the migrations.

  ```js
  "postinstall": "knex migrate:latest"
  ```


## Create app
 
*From the command line*

* Create a Heroku app with `heroku apps:create NAME_OF_YOUR_APP`.
  - This will create an app on Heroku from your terminal, and automatically add it as a remote in your local repo. Run `git remote -v` in your terminal to see this.

*Or, from heroku.com*

* From the dashboard, click the 'New' button in the top right corner. Create a name and a region and press 'create app'. Scroll down to the 'deploy using heroku git' section and copy the line that starts 'heroku git:remote -a YOUR_HEROKU_APP'. When you run this line in your terminal, it will add `heroku` as a new remote to your repo, similar to `origin`. Type `git remote -v` to see it.

## Setting up an exsiting App.

`heroku info <appName>` will give you the Git URL and then you wanna run `git remote add heroku {Git Url}` and this will set up your heroku remote the equivalent to `heroku git:remote -a <appName>` 


## Provision and deploy

1. Provision a Postgres DB using the postgresql addon
  - `heroku addons:create heroku-postgresql:hobby-dev`
  - This can also be done on heroku.com from the 'addons' section. Look for 'heroku postgres'.

2. Deploy to Heroku with `git push heroku master` or `git push heroku <localBranch>:master` for example `git push heroku dev:master`.

3. Now it's time to seed your database with any data you'd like it to have, so we need to login to the Heroku server. `heroku run bash` will open the terminal for your app hosted on Heroku. You will notice that it will be quite slow!
 - Apply the seed file by running `knex seed:run`.

4. Share and enjoy! If you see the application error page, type `heroku logs` into your command line in order to debug what may have gone wrong.


## Common gotchas

- Ensure that all required packages are in the `dependencies` part of your `package.json`. Heroku does **not** install anyting in `devDependencies`. Also, if a package is working globaly on your machine you may have forgotten to add it to your project explicitly with `--save`, which means it will break on a remote server. A best practice is to always install locally and use npm scripts.

- Any references to 'localhost' within your app will break unless they are provided with a production environement alternative.
