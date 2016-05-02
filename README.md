#Vue-Cli Template for Larvel + Webpack + Hotreload (HMR)

I had a really tough time getting my workflow rocking between Laravel and VueJS projects. I found myself building my VueJS projects independently in the Webpack dev server and then weirdly porting it into my Laravel projects. This was especially painful on existing projects where I was just trying to add a new component to the project.

I love Evan's approach and setup that you get with vue-cli. I'm sure there are many ways to do this but this is how I get started when building components that are going to live and communicate with a Laravel application. It allows me to build in real time, maintain Vue's data-store state' and just have less headaches all around.

This vue-cli template will hopefully help you get off the ground with your VueJS projects. Certainly, any feedback is greatly appreciated. 

##Requirements

You will need the following installed - If you're new I'll try and walk you through this so give it a go.

* [Homestead](https://laravel.com/docs/5.2/homestead)
* [NodeJS](https://nodejs.org/en/)
* [vue-cli](https://github.com/vuejs/vue-cli)

####Recommended

* [Laravel](https://laravel.com/docs/5.2) Command Line Installer 
* Vue Dev-tools extension for Chrome

##Install Laravel

I won't go into all the steps to make this happen [Laravel Install Docs](https://laravel.com/docs/5.2) but essentially, here are the steps I take with [Homestead](https://laravel.com/docs/5.2/homestead).

```
laravel new superproject (or whatever your project name is)
```

##Configure Laravel

I then create a database for this project and edit the ```.env``` file to point to the database by changing ```DB_DATABASE=homestead``` to ```DB_DATABASE=superdatabaseformyproject```

##Configure Host Machine

Edit your host file so you can render the site:

```sudo vi /etc/hosts```

And add the following

```127.0.0.1               superproject.app```

Edit your Homestead.yaml 

```vi ~/.homestead/Homestead.yaml```

Add the following to your list of sites:

```
    - map: superproject.app
      to: /home/vagrant/Code/superproject/public
```

Finally provision the site with Vagrant from your Homestead directory (most likely ```~/Homestead```) with this command

```vagrant provision```

##Install Laravel NPM Dependencies

Navigate to your projects root Laravel directory (probably ```~/Code/superproject```) . Update your ```package.json``` with the following:

```
{
  "private": true,
  "devDependencies": {
    "babel-core": "^6.7.7",
    "gulp": "^3.9.1",
    "laravel-elixir-browsersync2": "^0.1.0",
    "laravel-elixir-webpack": "^1.0.1"
  },
  "dependencies": {
    "bootstrap-sass": "^3.0.0",
    "laravel-elixir": "^5.0.0",
    // "laravel-elixir-webpack-ex": "0.0.4" // This has been removed pending some pull requests for both webpack config overrides and known bugs in the defined version of webpack-stream. For now use the fork below
    "laravel-elixir-webpack-ex": "dolbex/laravel-elixir-webpack-ex"
  }
}
```

and run the following:

```
npm install
```

Update your gulpfile.js with the following and be sure to scan it real quick and change any settings based on what you called your vue js project and what your server address is.

```
/*
 |--------------------------------------------------------------------------
 | Elixir Asset Management
 |--------------------------------------------------------------------------
 |
 | Elixir provides a clean, fluent API for defining some basic Gulp tasks
 | for your Laravel application. By default, we are compiling the Sass
 | file for our application, as well as publishing vendor resources.
 |
 */

// Just making paths a little shorter
var jsAssetsPath = './resources/assets/js'

// Requirements
var BrowserSync = require('laravel-elixir-browsersync2');
var elixir = require('laravel-elixir');
var gulp = require('gulp');

require('laravel-elixir-webpack-ex');


// Elixir extension to clean up for multiple Vue projects
elixir.extend('buildVueProject', function(mix, projectName, entryPath, configPath) {

  var project = {}
  project[projectName] = entryPath
  mix.webpack(project, require(configPath), elixir.config.publicPath);

});

elixir(function(mix) {
    mix.sass('app.scss');

    if(elixir.config.production) {
	    mix.buildVueProject(
	    	mix,
	    	'test-vue-app',
	    	'/my-vue-project/src/main.js',
	    	jsAssetsPath + '/my-vue-project/build/webpack.prod.conf.js'
	    );

	    // Let's let elixer take care of the hashing
	    mix.version([
	    	'public/js/css/test-vue-app.css',
	    	'public/js/test-vue-app.js'
	    ])
	} else {
		BrowserSync.init();
    	mix.BrowserSync({
	        proxy           : "testproject.app:8000/",
	        logPrefix       : "Project Name",
	        logConnections  : false,
	        reloadOnRestart : false,
	        notify          : false,
	        files			: ["**/*.php"]
	    });
	}
});
```

##Create new Vue-CLI project using this template

Ok, so at this point we could go to http://localhost:8000 and see the Laravel 5 welcome screen. Great. Now, let's get some awesome Vue JS in there! 

Head to (and you probably have to make this directory on a new project) ```~/Code/superproject/resources/assets/js/```

Bang out this command and change ```my-vue-project``` to whatever your Vue JS project name is. Remember, this is the component or widget that is going to live within your laravel app. It could be a map, slider, game, or, maybe it's the entire site accessing Laravel API endpoints you are going to make.

```vue init dolbex/webpack-laravel  my-vue-project```

Call the directory that was just made (in my case ```my-vue-project```) and run:

```npm install```

##Inserting your Vue JS app in your blade template

So, let's assume you've followed my steps and are working from a fresh Laravel installation. I'm going to crack open ```~/Code/superproject/resources/views/welcome.blade.php``` and replace the body with:

```
<body>
        <div class="container">
            <div class="content">
                <div class="title">Laravel 5</div>
            </div>
        </div>

        <app></app>

        <script src="http://localhost:8080/app.js"></script>

    </body>
```

All that we've done here is add the ```<app></app>``` which will be the container for our Vue JS root component and a script tag that is pointing to a server that isn't running yet. What is that?! That *will be* our webpack server. So, let's get that up and running.

### Pretend we're developing

Ok, so we're going to have three servers running at the same time. Homestead is already running. Let's start the other two. Remember, these are run on your machine *outside* of Homestead. I open a new terminal tab (I use Total Terminal) for each of these.

##### Webpack:

If you're not already there go to on your main machine (again, not Homestead): ```~/Code/superproject/resources/assets/js/vue-example``` and run ```npm run dev```

Now, if all you're working on is your Vue app you could go to ```http://superproject.app:8000``` and see the Vue logo next to the Laravel 5 logo. If you open an editor and edit ```~/Code/superproject/resources/js/vue-example/src/App.vue``` and make a change you should see it live update in the browser. 

You can test this by installing Vue Dev Tools extension for Chrome and pressing the button. You'll see on the App component has a 'test' property that will change from false to true (I had some redraw issues on dev tools. Just click another component and click back - this is something I need to figure out). Now change something in your ```{project-root}/resources/assets/js/vue-example/src/App.js``` file and the display will update but the state of the app will remain.

##### Elixir / Browsersync

Ok, but what if we want to take this a little further and live-update blade stuff as well? Not a problem. 

Open another terminal and head to your project root and run ```gulp```

This should open a new browser. What should be happening is that when you update any .php files (like a Blade file) Browsersync will inject updates into the browser. When you update your Vue application webpack will do the same.

### Production

So, what about when you're ready to go to production? Elixir and Laravel-Elixir-Webpack-Ex make this easy and you're all setup to go. If you want you can edit the names of your javascript by editing your ```gulpfile.js```

From the root of your project:

```gulp --production```

Add the following to the ```welcome.blade.php``` just before the ```</body>```:

```
<script src="{{ elixir("js/TestVueApp.js") }}"></script>
```

Add this in the ```<head>```:

```
<link rel="stylesheet" href="{{ elixir("js/css/TestVueApp.css") }}">
```

 These are using elixir for cache busting so you don't have to worry about cache on the server or in the client. 

### Adding in SASS Node Dependencies

If you want to add in something like bourbon / neat (a sass library) to your project you'll need to edit the webpack.base.conf.js file inside your vue js project. 

In the case of bourbon / neat add the following requirements to the top of the file.

```
...
var projectRoot = path.resolve(__dirname, '../')

var bourbon = require('bourbon').includePaths;
var neat = require('bourbon-neat').includePaths;

module.exports = {
...
```

Then add the sassloader section in the module:

```
  ...
  resolveLoader: {
    fallback: [path.join(__dirname, '../node_modules')]
  },
  sassLoader: {
      includePaths: [bourbon, neat]
  },
  module: {
  ...
```

Don't forget to restart your webpack dev server when you're done editing the config file.


### Feedback / Questions

Certainly, I am not a webpack / elixir professional yet so I am certainly open to any feedback that folks have concerning these technologies. Please post any feedback to the issues and I'll see if I can give you a hand.

### Sources

* [Browsersync.io Docs](https://www.browsersync.io/docs/options/)
* [Browsersync 2 Package for Elixir 3+](https://github.com/anheru88/laravel-elixir-browser-sync2)
* [Webpack Hot Middleware Multientry Example](https://github.com/glenjamin/webpack-hot-middleware/blob/master/example/webpack.config.multientry.js)
* [Stepan Parunashvili's answer to hot middleware from outside client](http://stackoverflow.com/a/35446292/215726)
* [Laravel-Elixir-Webpack-Ex](https://github.com/farrrr/laravel-elixir-webpack-ex)
* [Jeremy Dagorn's Blog on setup for something similar with React](http://jeremydagorn.com/posts/how_to_setup_react-hot-loader_in_your_laravel_application)
* [Matt Stauffer's Elixir + Vue Post](http://blog.tighten.co/setting-up-your-first-vuejs-site-using-laravel-elixir-and-vueify)
* [Amazing Answer by @vincentfretin with regards to multiple entry points - thank you sir.](https://github.com/gaearon/react-transform-boilerplate/issues/47#issuecomment-151909080)
* [Installing Boubon Github issue on](https://github.com/vuejs/vue-loader/issues/95)
