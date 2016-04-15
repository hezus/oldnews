#Oldnews

Tutorial on using Carrierwave with Heroku and Cloudinary to create a simple news site.

###Intro
We are going to make a news site. It will display a list of news articles. Each article will have a link to the page with the full text.
Each article will have a thumbnail and a full page image. We will use Carierwave to upload the files to our server.
CarrierWave is more flexible than Paperclip. It is based on Rack.
We will use the Cloudinary gem to upload files to their service and have a link to them.

###Create a project

Create a new rails project with postgres

```
$ rails new oldnews --database=postgresql
```

go into the directory of the new project

```
$ cd oldnews
```

open the project with atom

```
$ atom .
```

We will start with a model for NewsItems

```
$ rails g scaffold NewsItem title:string sub_header:text content:text
$ rake db:create
$ rake db:migrate
```

###Adding Git
This is as good a time as any to add Git to our project

1. Go to Github.com and create a new repo there
2. Open your terminal and go to the oldnews folder
3. Initialize a local git repository by running `git init`
4. Add all files to git to start tracking them with `git add .`
5. Commit this state of the project to your local repository with `git commit -m "first commit"`
6. Add the Github repository as remote `git remote add origin git@github.com:hezus/oldnews.git`
7. Push to the remote repository `git push -u origin master`

###Adding Carrierwave

Lets add [Carrierwave](https://github.com/carrierwaveuploader/carrierwave). Carrierwave is a way to handle fileuploads easy.
You have to add the gem to your gemfile.
`Gemfile`
```
gem 'carrierwave'
```


Then run `bundle` to install the gem. Carrierwave has generators that makes a uploader for us.
```
$ rails g uploader image
	create  app/uploaders/image_uploader.rb
```
The generator creates a new directory called `uploaders` under the app directory and in it a file called `image_uploader.rb`.


We need to add the uploader to the `NewsItem` model. You add a column to the `news_items` table. 
```
$ rails g migration add_image_to_news_items image:string
$ rake db:migrate
```


Open your model file add the mount_uploader method to use the `ImageUploader` we just generated.
`app/models/news_item.rb`
```ruby
class NewsItem < ActiveRecord::Base
  mount_uploader :image, ImageUploader
end
```


You have to add a file_field to the form.
`app/views/news_items/_form.html.erb`
```
  <p>
    <%= f.label :image %><br>
    <%= f.file_field :image %>
  </p>
```
adding a file_field makes the form automatically add the html options `{:multipart => true}`


You have to add the image to the controller permitted parameters
`app/controllers/news_items_controller.rb`
```
def news_item_params
  params.require(:news_item).permit(:title, :sub_header, :content, :image)
end
```
TODO: something about params hash


Add some layout to the whole thing. After the create, the app redirects you to the show action. 
Make the image span the entire page and so we can scroll over it with our news article.
`app/views/news_items/show.html.erb`
```html
<%= image_tag @news_item.image_url.to_s, class: 'bg' %>
```


Add css to make the image span the entire page
`app/assets/stylesheets/news_items.scss`
```css
img.bg {
  min-height: 100%;
  min-width: 1024px;
  width: 100%;
  height: auto;
  position: fixed;
  top: 0;
  left: 0;
}
```


Off course the news text needs to be displayed aswell.
`app/views/news_items/show.html.erb`
```
<%= image_tag @news_item.image_url.to_s, class: 'bg' %>

<div class="content">
<h1><%= @news_item.title %></h1>
<h2><%= @news_item.sub_header %></h2>
<p>
  <%= @news_item.content %>
</p>
</div>
```


and expand the CSS rules for the content
`app/assets/stylesheets/news_items.scss`
````
img.bg {
    min-height: 100%;
    min-width: 1024px;
    width: 100%;
    height: auto;
    position: fixed;
    top: 0;
    left: 0;
  }
  .content {
    position: relative;
    width: 500px;
    margin: 50px auto;

    padding: 20px;
    background: white;
    h1 {
      font-size: 40px;
      line-height: 50px;
    }
  }
```


Time to start your local server to check the progress.
```
$ rails s
```
And navigate your browser to `http://0.0.0.0:3000/news_items`

###Commit to git

Time to commit to git again. The uploads directory should be added to the git ignore file.
`.gitignore`
```
/public/uploads
```


Then in the terminal
```
$ git add .
$ git commit -m 'added carrierwave'
$ git push origin master
```

###Adding [Cloudinary](http://cloudinary.com/documentation/rails_integration#getting_started_guide)

Carrierwave is nice, but if you deploy to a service like Heroku, it will not work. Cloudinary solves this problem.
You upload the files to their servers so they don't disappear after reboot.

Start by making a account on cloudinary.com and then adding the gem to the gemfile.
`Gemfile`
```
gem 'cloudinary'
```


Run bundle to install the new gem
```
$ bundle
```


Update the image uploader class to work with cloudinary
`app/uploaders/image_uploader.rb`
```ruby
class ImageUploader < CarrierWave::Uploader::Base
  include Cloudinary::CarrierWave
end
```
TODO: include


Set the cloudinary URL in the terminal 
```
$ export CLOUDINARY_URL=cloudinary://874837483274837:a676b67565c6767a6767d6767f676fe1@sample
```
TODO: environment variables

You can test your progress.


Voeg een thumbnail optie toe voor de homepage
`app/uploaders/image_uploader.rb`
```ruby
  version :thumbnail do
    eager
    resize_to_fit(300, 300)
    cloudinary_transformation :quality => 80
  end
```

Change the homepage to display a overview of images
`app/views/news_items/index.html.erb`
``` html
<h1>News!</h1>
<% @news_items.each do |news_item| %>
    <a href="<%= news_item_path news_item%>" class="summary">
      <%= image_tag(news_item.image.thumbnail.url, :class => "thumbnail inline") %>
      <h2><%= news_item.title %></h2>
      <h3><%= news_item.sub_header %></h3>
    </a>
<% end %>
```


And add some css to make it look slightly better.
```css
.summary{
  background: none;
  text-decoration: none;
  width: 300px;
  height: 350px;
  overflow: hidden;
  float: left;
  &:hover{
    background: none;
    text-decoration: none;
    h2, h3{
      color: black;
    }
  }
}
```


Add the routes file
`config/routes.rb`
```ruby
root 'news_items#index'
```


Time for another commit to git
```
$ git push origin master
```


### Add Heroku
```
$ heroku create
```
```
$ git push heroku master
```
```
$ heroku run rake db:migrate
```
```
$ heroku restart
```
```
$ heroku config:set CLOUDINARY_URL=cloudinary://874837483274837:a676b67565c6767a6767d6767f676fe1@sample
```

