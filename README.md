# The Problem

When you make changes to your models, you need to reload the server.js file with the {force:true} setting on sequelize in order for the model changes to take place. But doing that also wipes away any seeding you've done. So you have to run the server file with force:true, quit and run the seed file, then run the server.js file AGAIN *without* force:true applied. Major pain.
You can avoid all this by doing the following. It's easier than it seems:

1. Create a file in the root of your project called sync.js, and inside it put this code:
```
const db = require("./models");
db.sequelize.sync().then(function() {
  console.log("models synced")
  process.exit()
});
```

2. Make sure you have `sequelize-cli` as a dependency. You'll need it.

3. Make sure your `config/config.json` file has all the correct database credentials etc. The sequelize-cli package will look for this.

4. In your `server.js` file, make sure that the `{force:true}` config option is REMOVED from the sync call at the bottom of the file.

5. In your package.json file, your scripts object should have the following:
```
"start": "node server.js",
"deploy": "npx sequelize db:drop && npx sequelize db:create && node sync.js && npm run seed && node server.js",
"seed": "npx sequelize db:seed:all"
```

6. You can have other scripts if you want, but you MUST have these.

7. You must have the database you are using created already in Workbench, but only for the very first time you do the next step. Thereafter sequelize-cli will delete it and create a new one each time.

8. Anytime you make a change to your models, Control-C to quit the server and then run `npm run deploy`<br>

# Here's What Happens

Here's what happens:
  - It deletes the existing database completely
  - It creates a new one, getting the name from your config file 
  - It loads up your models and creates the table structure (that's what sync.js does)
  - It seeds your database 
  - It then starts up the server so you're ready to go.

**IMPORTANT**: Once you have real user data in the database, don't run `npm run deploy` unless you are ok with losing that user-generated data.

You can go ahead and start using this process right away no matter how you have things set up -- none of your existing code gets deleted/changed in any way.

You'll no longer need to do any un-seeding. This happens by default when you delete the database and rebuild it.

Once you're completely happy with your models, you can just run `npm run start` instead of `npm run deploy`

Make sure you indicate in your README.md file that the deploy script should be run when first launching the project.