Oldnews
-----------------

Tutorial on using Carrierwave with Heroku and Cloudinary to create a simple news site.

Intro
===============================

We are going to make a news site. It will display a list of news articles. Each article will have tags linked to it to categorise it.
Each article will have a thumbnail and a full page image. We will use Carierwave to upload the files to our server.
CarrierWave is more flexible than Paperclip. It is based on Rack.
We will use the Cloudinary gem to upload files to their service and have a link to them.

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
```

This is as good a time as any to add Git to our project

1. Go to Github.com and create a new repo there
2. Open your terminal and go to the oldnews folder
3. Initialize a local git repository by running `git init`
4. Add all files to git to start tracking them with `git add .`
5. Commit this state of the project to your local repository with `git commit -m "first commit"`
6. Add the Github repository as remote `git remote add origin git@github.com:hezus/oldnews.git`
7. Push to the remote repository `git push -u origin master`

Let us start our local server
```
$ rake db:create
$ rake db:migrate
$ rails s
```

And navigate your browser to `http://0.0.0.0:3000/news_items`

Adding Carrierwave
---------------

Add carrierwave to our gemfile


```gemfile.rb
gem 'carrierwave'
```


Then run `bundle` to install the gem. Carrierwave has generators that makes a uploader for us.

```
$ rails g uploader image
	create  app/uploaders/image_uploader.rb
```

The generator creates a new directory called `uploaders` under the app directory and in it a file called `image_uploader.rb`.
We need to add the uploader to the NewsItem model. You add a column to the NewsItem table. 

```
$ rails g migration add_image_to_news_items image:string
$ rake db:migrate
```

Open your model file and mount the uploader

```ruby
class NewsItem < ActiveRecord::Base
  mount_uploader :image, ImageUploader
end
```

```
  <%= form_for(@news_item, :html => {:multipart => true}) do |f| %>
  
  ...
  
  <p>
    <%= f.label :image %><br>
    <%= f.file_field :image %>
  </p>
```

```
def news_item_params
  params.require(:news_item).permit(:title, :sub_header, :content, :image)
end
```

```.gitignore
/public/uploads
```