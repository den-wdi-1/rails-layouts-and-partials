<!--Hook: Raise your hand if you love writing out thousands of lines of static HTML.  Raise your hand if you love doing tons of DOM manipulation manually.  Ruby, and more specifically Rails, has a lot of power to do this work for us, and today we'll dive into that power, saving us countless hours of coding, and allowing us to change only what we need to change easily and quickly.  Today, we'll be talking about views in Rails.
-->

<!--1:00 5 minutes -->

# Layouts, Partials, and Views

### Objectives
*After this lesson, students will be able to:*

- **Describe** how templates & views work together
- **Use** partials for static content and rendering dynamic content

### Preparation
*Before this lesson, students should already be able to:*

- **Write** HTML
- **Describe** how to yield a template in a layout
- **Use** Ruby instance variables in templates
- **Use** params to store data in instance variables

<!-- 1:05 5 minutes -->

<!-- Bring up Sinatra layout.erb -->

## Views in Rails - Intro

In this lesson we will cover the details of rendering views and best practices for keeping the views DRY.

With Sinatra, we've seen that if there is a file called `layout.erb`, this file will be used as the app layout by default.

If this file `layout.erb` contains a yield statement `<%= yield %>`, then the template rendered for the current route will be "injected" where the yield statement is in the layout.  Combined, the templates and layouts will be sent back to the client.

<!-- Make new project with `rails new books_demo` and `rails g scaffold book title:string author:string` -->

<!-- Bring up this project's application.html.erb -->

This logic is pretty much the same with Ruby on Rails. When the app is created, Rails will automatically add a layout `application.html.erb` in `app/views/layouts/application.html.erb`. This layout already contains a yield statement and all the links to css and js files in the head part of the html document.

Take a minute and discuss the views files with your partner.  What is different from Sinatra?  What is the same?

<!-- 1:10 15 minutes -->

## Using views with Rails - Demo

In Rails, the logic for rendering a view is quite straightforward. Given that every route in Rails will execute a method inside a controller, when the method is executed, Rails will look for:

1. A folder inside `views` corresponding to the controller's name (folder `books` for `BooksController`).
2. A file with the method's name and `.html.erb`.

For example , if we call `http://localhost:3000/books`, Rails will execute the method `index` in the controller `books` and then look for a template located in `app/views/books/index.html.erb`  This works when the method always renders the same template.

In some cases though, you may want to render a template with a different name than the current method. Let's take a look at an example action:

```ruby
  def create
    @book = Book.new(book_params)

    respond_to do |format|
      if @book.save
        format.html { redirect_to @book, notice: 'Book was successfully created.' }
        format.json { render :show, status: :created, location: @book }
      else
        format.html { render :new }
        format.json { render json: @book.errors, status: :unprocessable_entity }
      end
    end
  end
```

Based on the result of `@book.save`, the method will execute either the code in the `if` or in the `else`.  The code `format.html` or `format.json` means that Rails will understand the format asked by the user, html or JSON.  For the moment, we will dive deeply only on the lines starting with `format.html`.

In the `if` case , the code executed is the block `redirect_to @book, notice: 'Book was successfully created.'` This code will redirect the request to the show method. `redirect_to` the `@book` object in Rails means "go to the method that will show only this object".

In the `else` case, the code executed is `render :new` - this means, Rails will show the template `app/views/book/new.html.erb`.  This template uses an instance variable `@book`. In this case, it will use the instance variable defined at the start of the create method.

To sum it up, Rails will, by default, render the template that has the name of the current method in the controller, unless there is a `render` statement in the method that tells Rails to use a different template.

There are different syntaxes for render, and they all do the same action described above but the rule of thumb is to use the simplest one that makes sense for the code you are writing.

<!-- Note: Review and explain each of the render functions below. -->

```ruby
render :edit
render action: :edit
render "edit"
render "edit.html.erb"
render action: "edit"
render action: "edit.html.erb"
render "books/edit"
render "books/edit.html.erb"
render template: "books/edit"
render template: "books/edit.html.erb"
render "/path/to/rails/app/views/books/edit"
render "/path/to/rails/app/views/books/edit.html.erb"
render file: "/path/to/rails/app/views/books/edit"
render file: "/path/to/rails/app/views/books/edit.html.erb"
```

<!-- Raise your hand if this freedom excites you.  Raise your hand if it scares you. -->

<!-- 1:25 25 minutes -->

## Integrating Layouts - Codealong

Create a new Rails app "views_and_layouts" and scaffold the resource posts:

```bash
rails new views_and_layouts
cd views_and_layouts
rails g scaffold post title:string content:text
rake db:migrate
```

Open the posts controller and look at how each method renders the templates: some of them, like index and show, are abstract because the name of the template is the name of the method, but for some other methods, like create or update, we need to explicitly tell Rails what to do at the end of the method.

<!-- Half-mast -->

#### Different Layouts

By default, Rails will render the layout `application.html.erb` - just like `layout.html.erb` in Sinatra - but sometimes, you want to render a template in a different layout.

For instance, let's create a layout called `sidebar.html.erb`

```bash
touch app/views/layouts/sidebar.html.erb
```

Take this template and add it into the `sidebar.html.erb`:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Sidebar Template</title>
</head>
<body>
  <header>
    <h1>My Website</h1>  
    <ul>
      <li>Menu 1</li>
      <li>Menu 2</li>
      <li>Menu 3</li>
      <li>Menu 4</li>
    </ul>
  </header>
  <main>
    <%= yield %>
  </main>
  <footer>
    <ul>
      <li>About us</li>
      <li>Team</li>
      <li>Terms and conditions</li>
    </ul>
  </footer>
</body>
</html>
```

This will help us to differentiate the layouts.

In the controller method `index`, add this to the end of the method:

```ruby
render layout: "sidebar"
```

This line will just tell Rails to use the same logic of template rendering, but instead of using the default `application.html.erb`, it will render the template inside `sidebar.html.erb`.

Now make your own sidebar.  Create at least two distinct sections.  Customize the HTML to whatever you want to show up in your app.  Don't forget the `<%= yield %>` tag!

<!-- Catch-up / Half-mast -->

#### Using partials

A best practice is to always keep every template as small as possible. A rule of thumb would be to keep shorter than 50 lines, for the sake of readability.

So for example, if your website has a layout with a top menu, the content and then the footer, this should all be in the layout, as it is rendered on every page.  But if all this html is in one file, the layout will end up being 200+ lines of code.

When this happens, you should think about splitting the template into partials. Partials are just chunks of a template that will be rendered into another one, like children of the layout.

This layout in `sidebar.html.erb` will soon be too long and unreadable, thus we need to split into different parts:

Create a new file: 

```bash
mkdir app/views/application
touch app/views/application/_header.html.erb
```

And move the following inside from `sidebar.html.erb`:

```erb
<header>
  <h1>My Website</h1>  
  <ul>
    <li>Menu 1</li>
    <li>Menu 2</li>
    <li>Menu 3</li>
    <li>Menu 4</li>
  </ul>
</header>
```

Now let's create another file:

```bash
touch app/views/application/_footer.html.erb
```

And inside move the following from `sidebar.html.erb`:

```erb
<footer>
  <ul>
    <li>About us</li>
    <li>Team</li>
    <li>Terms and conditions</li>
  </ul>
</footer>
```

Now the main `sidebar.html.erb` file is back to a normal size, we just need to include the partials and the final result will then be the same than before.

**Note:** Every partial needs to have an underscore as the first character - this way Rails knows that it is not a proper template but only a partial that will be included in a template.

Let's now call the partials in the layout:

```erb
<!DOCTYPE html>
<html>
<head>
  <title>Sidebar Template</title>
</head>
<body>
  <%= render "header" %>
  <main>
    <%= yield %>
  </main>
  <%= render "footer" %>
</body>
</html>
```

Rails will automatically look in the folder `app/views/application/` for a file that is called by the name given to the method `render` with an underscore prefix.

Now split up your template with two partials that you render inside your template.

<!-- Catch-up -->

<!-- 1:50 20 minutes -->

##Independent Practice (You Do)

Create a Rails app called `fanzine`. This app will have a controller called `home` and four routes called:

- Homepage
- Info
- About
- redirect_home

You'll have to match these routes with these templates:

- Homepage renders the template "homepage"
- Info renders the template "information"
- About renders the template "about"
- redirect_home redirect the request to "/homepage"

This app only has one layout, but the layout should have a menu with links to the four routes. These links should be inside a partial called `menu` and the file should be in `app/views/application`. The menu should appear on every page.

<!--2:10 5 minutes -->

## Conclusion

#### Questions

- What are the two main ways we used the method `render` today?
- What is the easiest way to go to the show page of a restful controller from another method in the same controller?
- How do we render a different layout only for one method?


## Licensing
All content is licensed under a CC­BY­NC­SA 4.0 license.
All software code is licensed under GNU GPLv3. For commercial use or alternative licensing, please contact legal@ga.co.
