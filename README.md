#Vue-Cli Template for Larvel + Webpack + Hotreload (HMR)

I had a really tough time getting my workflow rocking between Laravel and VueJS projects. I found myself building my VueJS projects independently in the Webpack dev server and then weirdly porting it into my Laravel projects. This was especially painful on existing projects where I was just trying to add a new component to the project.

Here is a vue-cli template that will hopefully help you get off the ground with your VueJS projects. Certainly, any feedback is greatly appreciated. 

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
    "laravel-elixir-webpack-ex": "0.0.4"
  }
}
```

and run the following:

```
npm install
```

Update your gulpfile.js with the following:

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

var elixir = require('laravel-elixir');
var BrowserSync = require('laravel-elixir-browsersync2');

// Just making paths a little shorter
var jsAssetsPath = './resources/assets/js'

require('laravel-elixir-webpack-ex');

elixir(function(mix) {
    mix.sass('app.scss');

    if(elixir.config.production) {
	    // This is going to use the production configuration
	    // which will place the build files in /public/js/static/
	    mix.webpack({
	    		TestVueApp: '/vue-example/src/main.js',
	    	},
	    	require(jsAssetsPath + '/vue-example/build/webpack.prod.conf.js')
	    );

	    // Let's let elixer take care of the hashing
	    mix.version([
	    	'public/js/css/TestVueApp.css',
	    	'public/js/TestVueApp.js'
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

