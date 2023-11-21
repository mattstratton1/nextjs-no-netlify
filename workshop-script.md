# Configure your app and data for Continuous Delivery with Netlify

> **Note** this is the script for the workshop that uses this repository.


## Show the app running

Start by showing the web site running, so the audience have some idea of what
it is about.


Use a pre-prepared fork of https://github.com/Aiven-Labs/nextjs-netlify (that is, a fork of the original repository).
URL is: https://netlify-workshop-demo.netlify.app/


Point out that this is *not* the version of the repository we’re going to be
using in the workshop - it’s an entirely different one we prepared earlier. In
the workshop we’ll start with a version of the web app that isn’t set up to
work with netlify. But by the end of the workshop, we should be able to
reproduce what we’re showing now.

Show the app actually doing something (look at a recipe) and maybe also look
in the database to show what’s there (for instance, using psql at the command
line).

Explain that it's using PostgreSQL as a database, and Redis as a cache to
speed up (repeated) database access.

Explain that the application comes with its own data - deploying it loads the
data into PostgreSQL.

## Getting set up

> Help/Setup: https://go.aiven.io/help-netlify-workshop

> The GitHub repository: https://github.com/Aiven-Labs/nextjs-no-netlify

Check people have forked the repository from  https://github.com/Aiven-Labs/nextjs-no-netlify
(see the [workshop help page](https://go.aiven.io/help-netlify-workshop) for
some guidance on this.)


> **Note** We’re using gitpod to work in - once we’ve forked the repository,
> then we can use  `https://gitpod.io/#https://github.com/USERNAME/nextjs-no-netlify`.

Check people have an Aiven login, and quickly show them how to get on.

> Aiven signup: https://go.aiven.io/signup-netlify-workshop

Lead them through creating the PostgreSQL and Redis services they’re going to need

> We recommend using `aws-us-east-1` in the North America region to minimise latency, as this region will be closest to where the Netlify free plan deploys its functions.

While those are running up, lead them through getting a Netlify account.

> Netlify signup: https://app.netlify.com/signup

1. Go to https://app.netlify.com/signup and do all the things
2. Use GitHub to sign in, as it’s so convenient.
3. Answer all the questions…
4. Skip the end bit (choosing GitHub or whatever - we'll set that up later), and that should be done

Check people have installed the netlify command line application

1. `netlify --version` to check it's there and working
2. Otherwise, refer back to the forum page

   * `brew install netlify-cli` on a Mac
   * `npm install netlify-cli -g` otherwise (needs Node.js to be installed)

Now we’re going to look at the application we’re working with.

* It’s a Javascript app

  * It uses Prisma as its ORM to talk to the databases
  * Show the `package.json` file and explain the build command we’ll be using, `build-deploy`.
  * Explain that this is a general npm target - it’s not “bound” to netlify - hence calling it `build-deploy`
  
> **Object-relational mapping** (ORM) is a technique that creates a layer between the language and the database, helping programmers work with data

Login to netlify
```
netlify login
```

This takes us to a webpage to authorize it doing things to GitHub

Back at the command line, use
```
netlify status
```
to show that this has worked and you’re logged in as the right user

Set up the repository to use netlify:
```
netlify init
```

And answer the questions:

* `What would you like to do?` - Use arrow keys to select **Create & configure
  a new site**
  ```
    ❯  Connect this directory to an existing Netlify site
    +  Create & configure a new site
  ```
  Choose **Create & configure a new site**
* `Team:`  - Use arrow keys to choose
* `Site name` - Leave blank to get a random name or make one up

* Then it asks for permission to access that repo on GitHub.

   > **Note** If the presenter is using gitpod, then they'll need to use a GitHub
   > access token to do this. There's no need to explain the details. The
   > audience will probably want to use the GitHub app.
   >
   > (It will show the token! So explain “not going to show you my token!” and
   > show a different tab (cute kitten (or dog) picture!) while pasting the token in, and then clear the terminal before going back, so the viewers don’t get to see.)

* Build command, which is `npm run build-deploy` (using that `build-deploy` target we mentioned earlier)
* Where to publish -  `.next` is OK
* is it OK to create the TOML file? - yes, it is

Show the content of the new netlify.toml file

1. `git status` should show that `netlify.toml` is untracked
2. `git add netlify.toml`
3. `git commit -m “Add netlify toml”`
4. `git push`

   ...which is the same as `git push origin main`. And yes, we’re pushing to main, that’s OK in this demo

> **Note** can use `netlify open` to go to the netlify dashboard for the site


## Define which PostgreSQL and Redis the app should use

* Go to the Aiven Console, and to the PostgreSQL service page
* Copy the SERVICE URI
* Use `netlify env:set DATABASE_URL ‘THE-POSTGRES-SERVICE-URI’` - remember
  we’ll need to quote the URL at the command line

* Go to the Aiven Console, and to the Redis service page
* Copy the SERVICE URI
* Use `netlify env:set REDIS_URI ‘THE-REDIS-SERVICE-URI’`

`netlify env:list` will show them back

Now use `netlify open` to go to the web application (or change to the tab you
had open for netlify earlier on)

Show the environment variables in the netlify web app as well

Kick off a new build using the web app - this time it should succeed

> **Trigger deploy** > **Deploy site**

Explain that the first deployment will have failed because it will have
happened before we set the environment variables! (it starts building as soon
as we did `netlify init`). This is OK! This last (just now) deployment should
succeed

<!--
  Presenter can take the time to create the services that are needed for the
  "and this is how we handle production" bit at the end -->
-->

Open the deployed app (from the netlify page) and show it works.

## Make a change to the app

* `git checkout -b wonderful` to make a new branch (other git commands
  for creating a new branch and switching to it exist)
* Edit `src/components/Navbar/Navbar.tsx` at line 22, put "Wonderful" just in
  front of "Recipes" (leave the indentation the same)
* `git status` (it's always useful to check what's going on with `git status`)
* `git add src/pages/index.tsx`
* `git commit -m 'Make everything wonderful'`
* `git push origin wonderful` to push our changes (and this new branch) to the
  remote repository.

Follow the link that is printed out to go back to GitHub, and make a PR for that change.

**In the new PR, remember to change the upstream URL so it doesn’t try to make
a PR for the place we forked from - we want it to merge back to `main` from
our forked repository**

* `netlify open` (or go back to the Netlify tab) and show how (eventually) the netlify deployment shows in the PR
* Go to the Netlify web app and see it building there 
* When it’s done, show it’s also done in the PR
* And the change is in the newly deployed app - even though we haven’t merged the PR or anything


While it’s doing all this, talk about why this is useful (automatic delivery, allowing the PR reviewer to check things against a live database without needing to spin the environment up themselves, the possibility of writing integration tests that are run in CI at this stage

When it's deployed the PR change, show that the testing app and the production
app are *talking to the same database* - changes in one reflect in the other.
This is Not a Good Thing.
  
## Production and testing

We’re using the same database(s) for both production and testing - this is a
Bad Idea. We’re doing that because we’re using the free PG and Redis, and a
user is only allowed one of each - which isn’t a problem for real use cases,
because the free tier is not suitable for production.

* Switch over to the prod url and show wait a minute, the data is changing when I change it in the test url? Whoops this isn’t good
* Explain that you probably want to use a different data services set up in pre-prod vs prod
* Explain that this part is just for demo as we don’t have time to set this all up for everyone
* Show that there is a new prod postgres and redis already created (presenter did this in the background earlier)
* Show that we need to update the env variables to have a different URL for the services for prod than for everything else
* Call out “oh wait, the build command seeds the database! We don’t want to do that every time in prod!”
* Then say “well wait, lets see what that really does…” look at line 38 in prisma/seed.ts and see that it only does that for a blank DB
* After changing the env variables to add for prod, trigger a deploy on prod and then show that the two different places (prod vs the pr one) have different data.


Talk about using a separate build configuration for production and for testing,
targetting different PostgreSQL and Redis instances.

## What's Prisma?

We may not address this in the workshop, unless asked, but
[Prisma](https://www.prisma.io/) is an ORM (object-relational mapping) for Node.js and TypeScript

An ORM, according to wikipedia:
  > Object-relational mapping (ORM) is a technique that creates a layer
  > between the language and the database
  
Prisma says it provides:
>  an intuitive data model, automated migrations, type-safety & auto-completion.

## Summary
* We’ve now shown how to integrate our application (and its data) with Netlify
* Which is one way of doing continuous delivery / deployment
* ...


## Addenda

SQL command from Francesco: `update recipes set is_liked=true where recipe_name ='Apricot Danish Coffee Cake';`

### Other links

Forums: https://go.aiven.io/forums-netlify-workshop

Feedback: https://go.aiven.io/feedback-netlify-workshop 

DevCenter: https://go.aiven.io/dc-netlify-workshop

Newsletter: https://go.aiven.io/newsletter-netlify-workshop

Upcoming Workshops: https://go.aiven.io/workshops-netlify-workshop

