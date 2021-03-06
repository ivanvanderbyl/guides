## Getting Started with EmberJS

*Thanks to Robin Ward who has a lot of great Ember content on [his blog](http://eviltrout.com). Some of his posts, such as [Ember without Ember Data](http://eviltrout.com/2013/03/23/ember-without-data.html) have inspired this guide*.

EmberJS is one of approximately a thousand JavaScript frameworks that are available. The [website for EmberJS](http://emberjs.com) calls Ember "a framework for creating **ambitious** web applications." That's quite a claim to make! Ember is extremely opinionated (just like Rails!), and so a lot of the hard decisions have already been made, which is great.

Rather than go through all the other information about it that you can find out all over the web, let's just go ahead and get started with Ember. What we're going to do is to turn a very, very basic Rails blogging application called "blorgh" into one that uses Ember.

To begin, let's clone "blorgh":

    git clone git@github.com:radar/blorgh
    cd blorgh
    git checkout rails

What this application has at the moment are a `Post` and `Comment` model and if you're at all familiar with the [Getting Started with Rails](http://guides.rubyonrails.org/getting_started.html) guide, then you'll know how these things work.

This application also has an API, which can be accessed at routes like `/api/posts`, `/api/posts/1/comments`. It's this API that we'll be talking to in order to get the information that we need to make Ember happy.

## Installing Ember

To install Ember into this application, we'll need to install the `ember-rails` and `ember-source` gems, which we can do by adding these lines to our `Gemfile`:

    gem 'ember-rails'
    gem 'ember-source', '1.4.0'

Then we can run `bundle install` to install these gems.

Next, we need to set up the Ember structure for this application, which we can do with this command:

    rails generate ember:bootstrap

This creates a couple of sub-directories within `app/assets/javascripts`:

* `models`: These are the objects which will talk with our API to retrieve data from it.
* `controllers`: Used for decorating the models with display logic. Similar concept to helpers within Rails.
* `views`: A place to put complicated view logic.
* `components`: Reusable pieces of the application.
* `routes`: Code that tells Ember what to do when a specific route is requested.
* `templates`: Handlebar templates which are responsible for displaying the information from the models.
* `helpers`: A place to define Handlebar helpers for your templates.

You'll see how all these pieces tie together throughout this tutorial, so don't worry if you don't grok it just yet.

## Surveying the landscape

If we look at our `app/assets/javascripts` directory now, we'll have two versions of `application.js`, one called `application.js` and one called `application.js.coffee`. The latter was generated by `ember:bootstrap`, and is the one we'll keep around. Let's delete `app/assets/javascripts/application.js` and look at `app/assets/javascripts/application.js.coffee`.

```coffee
#= require handlebars
#= require ember
#= require ember-data
#= require_self
#= require blorgh

# for more details see: http://emberjs.com/guides/application/
window.Blorgh = Ember.Application.create()
```

This file requires *almost* everything that we need for Ember. It's actually missing a require for jquery, which we can just simply add at the top. In this guide I'm not going to use Ember Data because the API that the application provides does not comply with Ember Data's standards, so we will remove that line. The end result will be a file that looks like:

```coffee
#= require jquery
#= require handlebars
#= require ember
#= require_self
#= require blorgh

# for more details see: http://emberjs.com/guides/application/
window.Blorgh = Ember.Application.create()
```

This file now requires jQuery, Handlebars, Ember, itself and then finally `blorgh`. The `blorgh` file that it's pointing at is `app/assets/javascripts/blorgh.js` which contains this:

```coffee
#= require_tree ./models
#= require_tree ./controllers
#= require_tree ./views
#= require_tree ./helpers
#= require_tree ./components
#= require_tree ./templates
#= require_tree ./routes
#= require ./router
#= require_self
```

The `store` file here will set up an Ember Data store, but since we're not going to be using that we can remove that line. The rest of the file includes all the other parts of our application: the models, controllers, views, helpers, components, templates, routes and the all-important router. The router is what's going to tell our Ember app what to do, extremely similar to what `config/routes.rb` does in Rails apps.

## Starting our Ember app

We're going to host our Ember app *through* our Rails app. The Rails application is going to be providing the API that Ember will use to perform CRUD actions on our data. In this tutorial, we're going to replace all the Rails code that is currently responsible for displaying posts and comments, with some Ember code that does the same thing.

Let's ensure that the database is setup now by running `rake db:setup`. After that, we will start the Rails app with `rails s`. When we go to http://localhost:3000 we should see the application with a couple of posts.

![Posts](/ember/posts.png)

At this point, our Ember code should be up and running as well. We can see if that's true by opening the JavaScript console for our browser. If you're using Chrome, you'll see something like this:

![Chrome Console](/ember/chrome_console.png)

This is some helpful information that tells us the application is running. If you've installed the [Ember Inspector](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi?hl=en) within Chrome, you can go over to that tab in the JavaScript console and see an indication that our app is running too:

![Chrome Ember Inspector](/ember/chrome_ember_inspector.png)

If the application wasn't running, you'd see this:

![Ember Not Found](/ember/ember_application_not_found.png)

If you *are* seeing this, then make sure you've required the correct JavaScript files.

If you *aren't* seeing this, then you've correctly setup the Ember app and we can proceed.

The first thing we're going to do is to provide an outlet for Ember to put its content. Let's open `app/views/layouts/application.html.erb` and replace this line:

```erb
<%= yield %>
```

With these lines:

```erb
<script type='text/x-handlebars'>
{{outlet}}
</script>
```

This defines the application template for Ember. Ember will be rendering all
of its content into this area.

So what's the next step after this? Well, Ember doesn't really provide any
hints itself. However, there's some debug settings that we can set that will
give us some hints as to what to do. These are mentioned in the [Understanding Ember: Debugging](http://emberjs.com/guides/understanding-ember/debugging/) guide, but I'll repeat them here because why not.

Let's open `app/assets/javascripts/application.js.coffee` and set the `LOG_TRANSITIONS`, `LOG_TRANSITIONS_INTERNAL`, `LOG_VIEW_LOOKUPS` settings:

```cofeee
window.Blorgh = Ember.Application.create
  LOG_TRANSITIONS: true
  LOG_TRANSITIONS_INTERNAL: true
  LOG_VIEW_LOOKUPS: true
```

These will show us a lot more information in our console, which will tell us what Ember's doing during a request. If we refresh the page now, we'll see this on our console:

![Ember Transitions](/ember/ember_transitions.png)

## Rendering a template

On the page itself, it will now be blank. This is because we're not telling Ember to do anything. Where we are now -- on the homepage -- we want to show a list of posts. If we look at the transitions in our console, we can see that Ember cannot find the index template:

```
Could not find "index" template or view. Nothing will be rendered
```

Ember's attempted to find this template, but couldn't find it. We can define this template within `app/assets/javascripts/templates/index.hbs`. Let's start off simple:

```hbs
<h1>Posts</h1>
```

When we refresh that page, we should see that header now appearing:

![Posts header](/ember/posts_header.png)

This is our first Ember view rendering. That was easy!

## Fetching content

Our next couple of steps is to find the posts and add them to this page. In Ember, this is done by objects that extend from `Ember.Route`. We don't need to create these objects ourselves -- as we would have to do in other frameworks, like Backbone -- Ember will automatically create them itself. We can see the result of this magic when we switch to the "Routes" tab in the Ember Inspector in Chrome:

![Ember Routes](/ember/ember_routes.png)

Therefore we can know from this information that we need to define the fetching of the data within the `IndexRoute` object. Let's create a new file at `app/assets/javascripts/routes/index.js.coffee`:

```coffee
Blorgh.IndexRoute = Ember.Route.extend
  model: ->
    Blorgh.Post.findAll()
```

This `IndexRoute` object is responsible for the collection of data. This is similar in concept to an action within a Rails controller. This data fetching is done with the `Blorgh.Post.findAll()` function call. If we refresh our page at this point, we'll see that our Ember code is no longer happy:

```
Error while loading route: TypeError: Cannot call method 'findAll' of undefined
```

While Ember infers routes, it does not infer models. Therefore we must define the model ourselves, which we can do by creating a new file in `app/assets/javascripts/models/post.js.coffee`:

```coffee
Blorgh.Post = Ember.Object.extend({})
```

We need to define the `findAll` function on `Blorgh.Post` so that `Blorgh.IndexRoute` can happily collect its data. It's going to act very similar to a class method within a Ruby class, and so to do that, we need to call `reopenClass` and define our function:

```coffee
Blorgh.Post = Ember.Object.extend({})

Blorgh.Post.reopenClass
  findAll: ->
    posts = Em.A()
    $.getJSON('/api/posts').then (data) ->
      $.each data.posts, (post) ->
        posts.pushObject(data)
    posts
```

This code defines a simple Ember.Array object, which is the type of object that needs to be returned with the `model` call in `Blorgh.IndexRoute`. After defining that array, the code then makes a request to `/api/posts` which will query our API. Our API will dutifully return all the posts, and then the rest of the code in this function iterates through all of those posts and adds them to array. The final line in the method returns the list of posts.

When we refresh the page, we'll no longer see an error. Instead, at the very bottom of the console output, we'll see this:

```
XHR finished loading: "http://localhost:3000/api/posts".
```

This is a great indicator that shows that our Ember app is making a request to fetch all the posts. With all the posts being fetched, the next step is to display them. We can take care of this within the index template, over at `app/assets/javascripts/templates/index.hbs`:

```hbs
<h1>Posts</h1>

{{#each}}
  <h2>{{title}}</h2>
  {{text}}
{{/each}}
```

In this new code, we're iterating through the array returned by `Blorgh.IndexRoute`'s `model` function using the `{{#each}}` helper provided by Handlebars. This automatically knows what we want to iterate over, and it just does it. Within the scope of the `each` "block", we can reference the attributes of the posts very directly.

When we refresh this page, we'll see the posts now displaying with the power of Ember:

![Ember Posts](/ember/ember_posts.png)
