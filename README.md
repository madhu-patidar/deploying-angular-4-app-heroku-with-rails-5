
Angular4 Setup

	1. To install Node.js, type the following command in your terminal:

		sudo apt-get install nodejs

	2. Then install the Node package manager, npm:

		sudo apt-get install npm

	3. Install Angluar CLI via npm:

		npm install --save-dev @angular/cli@latest

Deploying an Angular 4 app on Heroku with Rails 5

1. Convert/copy package.json

In order for Heroku to build, it needs to know how to build angular, as well as build rails. This means at very least it needs a package.json file with a heroku-postbuild script. In my experience it needed more than this. I ended up copying all the dependencies and dev dependencies into the package.json file of the root of the directory. I also did not remove the package.json from the /angular directory. When I did this, it caused other troubles. I don't like part of the solution, but it's what is working for now. Iteration!



So in the end you should have a package.json at the root of your directory that looks something like this:

	{
	  "name": "railstest",
	  "scripts": {
	    "dev": "NODE_ENV=\"development\" && cd angular && ng serve",
	    "prod": "ng build --prod --output-hashing=none",
	    "heroku-postbuild": "npm run prod"
	  },
	  "dependencies": {
	    "@angular/cli": "1.0.0",
	    "@angular/compiler-cli": "^4.0.0",
	    "@angular/common": "^4.0.0",
	    "@angular/compiler": "^4.0.0",
	    "@angular/core": "^4.0.0",
	    ...
	  }
	}


Note: The heroku-postbuild step will be run after Heroku runs npm install. This makes sure that the Angular CLI is available so that ng build can be run.

2. Convert/copy .angular-cli.json

The next step is to copy the Angular CLI json file (.angular-cli.json) to be at the root level of your angular app.

Once copied from the /angular/.angular-cli.json, I had to change the "root" and "outDir" and a few other bits of the file to look inside the angular subdirectory:

	{
	  ...
	  "apps": [
	    {
	      "name": "myapp",
	      "root": "./angular/src",
	      "outDir": "./public",
	      ...
	    }
	  ],
	  ...
	  "lint": [
	    {
	      "project": "./angular/src/tsconfig.myapp.json"
	    },
	    {
	      "project": "./angular/src/tsconfig.spec.json"
	    },
	    {
	      "project": "./angular/e2e/tsconfig.e2e.json"
	    }
	  ],
	  ...
	  "test": {
	    "karma": {
	      "config": "./angular/karma.conf.js"
	    }
	  },
	}
	
Note: This tells Angular CLI where to look for its files. At this point, run npm run prod and confirm that the CLI compiles your project and dumps it in the /public directory of rails.



3. Setup Rails 5 to serve the Angular 4 App

This part isn't Rails 5, nor Angular 4 specific, you can use it on a multitude of types of front-end applications.

Edit config/routes.rb to add a root and an '*unmatchedroute' route.

	
	Rails.application.routes.draw do
	  root 'pages#index'

	  scope '/api' do
	    # my other api stuff
	  end

	  get '*unmatchedroute', to: 'pages#index'
	end

These two routes will send / to the front-end app, and any other routes that aren't dedicated to Rails (hence why I scope everything else under /api) get swallowed up by the front-end as well, and Angular handles the routing.

This route does no good without a controller to match. Create a file at app/controllers/pages_controller.rb, and paste the following (it'll be the easiest controller you ever write).

	class PagesController < ApplicationController
	  def index
	  end
	end

Now we need a view to load up the front-end app. I'm going to copy a fair bit of code from the generated index.html built by ng build --prod. Create a file at app/views/pages/index.html.erb and paste in the following:

	<% content_for :head do %>
	  <base href="<%= root_url %>">
	  <link href="/styles.bundle.css" rel="stylesheet"/>
	<% end %>

	<body>
	  <myapp-root>Loading...</myapp-root>
	  <script type="text/javascript" src="/inline.bundle.js"></script>
	  <script type="text/javascript" src="/polyfills.bundle.js"></script>
	  <script type="text/javascript" src="/vendor.bundle.js"></script>
	  <script type="text/javascript" src="/main.bundle.js"></script>
	</body>

You may now need to go edit app/views/layouts/application.html.erb and add the following to the head section:

	<%= yield :head %>


# deploying-angular-4-app-heroku-with-rails-5
