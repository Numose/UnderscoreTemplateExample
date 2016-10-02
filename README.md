#Coding Simple Data Views with Underscore.js Templates

![Underscore Template Example 'Twittling'](http://i.imgur.com/NQQ2M2b.png)

**JavaScript, HTML, CSS knowledge for this project is Beginner.**

[Link to Blog Post](https://medium.com/p/c6fa41fba099)

Sometimes you need to build a simple view with some structured data and you don’t want or need the overhead of one of the big frameworks.

Maybe it’s a weekend hacking project that needs a messaging feed component. Or a bare-bones internal tool for your startup that displays a list of user information that you can filter and sort.

You could use React or Angular, but these frameworks can take a lot of time to get setup and running and create a lot of overhead. jQuery makes DOM insertion easy, but even the most simple views can create tons of messy appends and removes and force you to craft incredibly long .html() insert strings to get the data on the page.

To solve these issues, I’d like to propose taking a look at [Underscore.js’s  _.template() method](http://underscorejs.org/#template).

Many front-end developers include jQuery and Underscore as standard libraries for useful tools like `_.each()` and `_.filter()` but don’t know about `_.template()`, which is a shame.
`_.template()` is powerful because you can compose reusable view components using HTML and CSS that are write-once-reuse-many, making it a breeze to make changes to the layout and styling down the line.

I’ve personally used `_.template()` on many small projects and love it, both for hobby code and even some production cases. Let’s dive in with an example and see what you think.

For those who want to get started right away or follow along in the code, here’s an example repo I put together. 

##Step 1: Source & Include jQuery.js + Underscore.js

First and foremost, you’ll need to import and reference the [jQuery.js](http://jquery.com/download/) and [Underscore.js](http://underscorejs.org/) JavaScript libraries. There are several ways you can accomplish this.

The easiest way by far is to use a CDN. If you go this route, make sure you use a trusted provider. If they go down regularly or decide to stop hosting the code, your page or even entire site could stop working. Not something you want.

You can also host them yourself within your own site by downloading the source or importing them through a package manager like Bower or NPM.

Once you’ve sourced them, include the scripts in your HTML. Make sure to load jQuery first since Underscore relies on it and your code could break otherwise.

##Step 2: Get the Data You Want to Display

To progress from here in a meaningful way, you’ll want to get your hands on the data to display.

In my example code I used a local array of objects that represent tweets, but you’ll most likely get your data from a web API. If for some reason you aren’t able to get real data you can always create a mock from specs and set it up the way I did until it becomes available.

If the data is available, you can use [jQuery’s $.ajax method](http://api.jquery.com/jquery.ajax/) to retrieve it asynchronously then simply hand it to your engine.

##Step 3: Build Out the Engine

Next we need to set up the Underscore template engine. 

Place these two blocks above the rest of your engine code but below the jQuery and Underscore imports. I prefer to have most if not all of my scripts at the bottom of the body element since it makes the page ‘feel’ faster to users and won’t cause any headaches with the template script being in the body of the document.

```javascript
_.templateSettings.variable = 'item'; 
var template = _.template(
  $('script.template').html()
);
```

Assign the `_.templateSettings.variable`, to something that fits with your data source; I’ve used ‘item’ in my example project, but it can be pretty much anything. ‘tweet’, ‘user’, or ‘account’ all work well. 

You’ll be accessing properties attached to this variable later so it makes sense to name it something intuitive.

We need to give the engine somewhere to insert the data, so create a container in the body of the HTML where you want the data items to display. It’s best not to have any other content in this container since it will probably be overwritten or cause general annoyance down the line.

```html
<body>
  <div id="feed" class="feed"></div>
</body>
```

Id’s over classes are preferred when an element is the target of selection / mutation. I try to only style elements using classes in CSS so that if style naming changes later in the project life-cycle our jQuery and JS won’t break unexpectedly.

Lastly, use an iterator of your preference to hand each data chunk into the templating function. If you are using local data like my example array, you can simply call this function with the array as the argument.

```javascript
var tweets = [{...}, {...}, {...}];

var renderTweets = function(arr) {
  _.each(arr, function(elem) {
    $('#feed').prepend(template(elem));
  });
};

renderTweets(tweets);
```

If you are using AJAX to get data from an API you can call the render function by passing the data object in as a step in the success callback. Using a function expression allows you to call render whenever you get new data to update the view without reloading the page.

```javascript
$.ajax({
  url: 'http://www.example.com/api-endpoint',
  type: 'GET',
  success: function(data) {
     renderTweets(data);
  },
  error: function(err) {
    console.error('something broke with the ajax request');
  }
});
```

##Step 4: Construct the Template

The final step in the templating process is to construct the actual template that each data chunk will be rendered into.

To start, insert some script tags into the `<body>` element of the html. It is really important to make sure to use the text/template script type in the `<script>` tag attribute signature. Classifying the script this way will cause the browser’s parser to skip over this section of your html when it loads initially. It won’t recognize it and so simply avoid it and move on, a handy technique used by many of the larger frameworks as well.

```html
<!-- other content, header, nav, etc. -->

<script type="text/template" class="template">
  <!-- template body -->
</script>

<!-- other content in the body, footer div, etc. -->
```

Compose the body of the template as you would any component of your page using regular HTML elements and attributes.

```html
<script type="text/template" class="template">
  <div class="post">
    <img src="" />
    <div class="post-data">
      <a class="post-name" href="#"><!--post name--></a>
      <span class="post-msg"><!--message body--></span>
    </div>
  </div>
</script>
```

Next, have a look at your data and figure out what display logic is needed. `_.template()` allows you to both insert variables as well as run JavaScript right from within the template at render time.
`<%- // variable.name %>` notation is how to insert variable values into the template that are handed in as an object by the iterator function we wrote earlier. You can use these variables as text values like the user name or the main content of the component, or as attribute values for an image source or data-x attributes which are useful for hooking into event listeners. 

```html
<script type="text/template" class="template">
  <div class="post" data-id="<%- item.id %>">
    <img src="<%- item.userImg %>" />
    <div class="post-data">
      <a href="#" class="..."><%- item.userName%></a>
      <span class="..."><%- item.msg %></span>
    </div>
  </div>
</script>
```

`<% JS logic %>` notation is how to insert JavaScript logic into the template. You can use this built-in logic, for example, to create fall-backs for missing or malformed data.

```html
<script type="text/template" class="template">
  <div class="post" data-id="<%- item.id %>">
    <img src="<%- item.userImg %>" />
    <div class="post-data">
    <% if (item.userName) { %>
      <a href="#" class="..."><%- item.userName %></a>
    <% } else { %>
      <span class="...">Unknown User</span>
    <% } %>
      <span class="..."><%- item.msg %></span>
    </div>
  </div>
</script>
```

Note: There is an alternate syntax you can use instead of `<% %>` and `<%- %>` that is commonly referred to as mustache syntax or handlebars. You will see it written as `{{ }}` in code. I went with the native syntax here for ease of use but get mustache syntax implemented if you are interested since it’s much easier to type.

##Step 5: Take It Out for a Spin

And that’s it. With your template built, everything should work. Exciting!

If you are getting any console errors or erratic behavior check your code against the code in the example repo. Common errors include forgetting the text/template designation in the template script tag, errors in the `<% %>` logic syntax, and malformed data objects being handed into the template.

I hope you enjoyed this walk-through and have success using Underscore templates in your own projects.

Cheers.