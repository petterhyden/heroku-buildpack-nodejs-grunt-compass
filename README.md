Heroku buildpack: Node.js with grunt, bower and sass (via compass) support
============================================

Supported Grunt versions: 0.3 and 0.4.
See the Grunt [migration guide](https://github.com/gruntjs/grunt/wiki/Upgrading-from-0.3-to-0.4) if you are upgrading from 0.3.

This repo is a fork of [alexaivars fork](https://github.com/alexaivars/heroku-buildpack-nodejs-grunt-compass). This fork added 
modified the order in which compass is installed. This way we make sure that we have SASS before grunt tasks are ran. Also,
added the option to install npm packages in non production mode. This means that devDependencies will be installed. This was added
since we also used Heroku for testing and rapid prototyping. Just set the environment variable `DEPLOY_TYPE` in Heroku config:

```
$ heroku config:set DEPLOY_TYPE="dev"
```

The alexaivars fork added support for installation of bower dependencies from private repositories. Todo this you will need to generate a 
ssh key pair, add the public RSA key to GitHub or BitBucket and then add the private to Heroku:

```
$ heroku config:set SSH_KEY="`cat your_rsa_private`"
```

This is a fork of a fork of a fork of [Heroku's official Node.js buildpack](https://github.com/heroku/heroku-buildpack-nodejs) 
with support for installing dependencies from private repositories from BitBucket and GitHub. Also, install 

For this build pack to run you will need to have `grunt` and `bower` as `dependencies` in your package.json.

After all the default Node.js and npm build tasks have finished, the buildpack checks if a Gruntfile (`Gruntfile.js`, `Gruntfile.coffee`or `grunt.js`) exists and executes the `heroku` task by running `grunt heroku`. For details about grunt and how to define tasks, check out the [offical documentation](http://gruntjs.com/getting-started). You must add grunt to the npm dependencies in your `package.json` file.
If no Gruntfile exists, the buildpacks simply skips the grunt step and executes like the standard Node.js buildpack.


How it Works
------------

Here's an overview of what this buildpack does:

- Uses the [semver.io](https://semver.io) webservice to find the latest version of node that satisfies the [engines.node semver range](https://npmjs.org/doc/json.html#engines) in your package.json.
- Allows any recent version of node to be used, including [pre-release versions](https://semver.io/node.json).
- Uses an [S3 caching proxy](https://github.com/heroku/s3pository#readme) of nodejs.org for faster downloads of the node binary.
- Discourages use of dangerous semver ranges like `*` and `>0.10`.
- Installs `compass`, caching it for future use.
- Uses the version of `npm` that comes bundled with `node`.
- Puts `node` and `npm` on the `PATH` so they can be executed with [heroku run](https://devcenter.heroku.com/articles/one-off-dynos#an-example-one-off-dyno).
- Caches the `node_modules` directory across builds for fast deploys.
- Doesn't use the cache if `node_modules` is checked into version control.
- Runs `npm rebuild` if `node_modules` is checked into version control.
- Check if `DEPLOY_TYPE` is set to `dev`, if true will run `npm install` instead of `npm install --production`
- Always runs `npm install` to ensure [npm script hooks](https://npmjs.org/doc/misc/npm-scripts.html) are executed.
- Always runs `npm prune` after restoring cached modules to ensure cleanup of unused dependencies.
- Runs `grunt` if a Gruntfile (`Gruntfile.js`, `Gruntfile.coffee`or `grunt.js`) is found.
- Doesn't install grunt-cli every time.


For more technical details, see the [heavily-commented compile script](https://github.com/stephanmelzer/heroku-buildpack-nodejs-grunt-compass/blob/master/bin/compile).

Usage
-----

Create a new app with this buildpack:

    heroku create myapp --buildpack heroku config:add BUILDPACK_URL=https://github.com/stephanmelzer/heroku-buildpack-nodejs-grunt-compass.git

Or add this buildpack to your current app:

    heroku config:add BUILDPACK_URL=https://github.com/stephanmelzer/heroku-buildpack-nodejs-grunt-compass.git

Set the `NODE_ENV` environment variable (e.g. `development` or `production`):

    heroku config:set NODE_ENV=production

Create your Node.js app and add a Gruntfile named  `Gruntfile.js` (or `Gruntfile.coffee` if you want to use CoffeeScript, or `grunt.js` if you are using Grunt 0.3) with a `heroku` task:

    grunt.registerTask('heroku:development', 'clean less mincss');

or

    grunt.registerTask('heroku:production', 'clean less mincss uglify');

Don't forget to add grunt to your dependencies in `package.json`. If your grunt tasks depend on other pre-defined tasks make sure to add these dependencies as well:

    "dependencies": {
        ...
        "grunt": "*",
        "bower": "*"
        "less": "*"
    }

Push to heroku

    git push heroku master
    ...
    ----> Fetching custom git buildpack... done
    -----> Node.js app detected
    -----> Requested node range:  0.10.x
    -----> Resolved node version: 0.10.25
    -----> Downloading and installing node
    -----> Found Gruntfile
    -----> Augmenting package.json with grunt and grunt-cli
    -----> Restoring node_modules directory from cache
    -----> Pruning cached dependencies not specified in package.json
           npm WARN package.json mealgen@0.0.0 No repository field.
    -----> Installing dependencies
           npm WARN package.json mealgen@0.0.0 No repository field.
    -----> Caching node_modules directory for future builds
    -----> Cleaning up node-gyp and npm artifacts
    -----> Installing Compass
    -----> Restoring ruby gems directory from cache
    Updating installed gems
    Nothing to update
    -----> Caching ruby gems directory for future builds
    -----> Building runtime environment
    -----> Running grunt heroku: task

Debugging
---------

npm can be run with a verbose flag to help debugging if something fails when installing the dependencies.

* if the `VERBOSE` environment variable is set, npm is always run with verbose logging.
* if `BUILDPACK_RETRY_VERBOSE` is set, npm is relaunched in verbose mode if npm failed.

Thanks to [mackwic](https://github.com/mackwic) for these extensions.

Further Information
-------------------

For more information about using Node.js and buildpacks on Heroku, see these Dev Center articles:

- [Heroku Node.js Support](https://devcenter.heroku.com/articles/nodejs-support)
- [Getting Started with Node.js on Heroku](https://devcenter.heroku.com/articles/nodejs)
- [Generating SSH keys](https://help.github.com/articles/generating-ssh-keys)
- [Buildpacks](https://devcenter.heroku.com/articles/buildpacks)
- [Buildpack API](https://devcenter.heroku.com/articles/buildpack-api)
- [Grunt: a task-based command line build tool for JavaScript projects](http://gruntjs.com/)
- [Compass: SCSS with batteries](http://compass-style.org/)
- [Bower: A package manager for the web](http://bower.io/)
