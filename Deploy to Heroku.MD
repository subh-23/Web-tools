
# Deploy to heroku

Install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-command-line)

Download and install the Heroku CLI.

#### If you haven't already, log in to your Heroku account and follow the prompts to create a new SSH public key.

```bash
$ heroku login
```

#### Create a new Git repository

#### Initialize a git repository in a new or existing directory

```bash
$ cd my-project/
$ git init
$ heroku git:remote -a login-weather
```

#### Deploy your application

#### Commit your code to the repository and deploy it to Heroku using Git.
```bash
$ git add .
$ git commit -am "make it better"
$ git push heroku master
```
