
# Using Foundation Gem in Rails

## Learning Goals
-  Be able to install Foundation using the Foundation Gem
-  Be able to use Foundation specific styles and widgets in your hmtl/css code.
-  Be able to adjust the Foundation variables to style Foundation apps

## Installing Foundation-Rails

First we will install the Foundation Gem and then configure it.  **Please remember to restart your rails server after installing Foundation.**  To install the Foundation-Rails gem add the following lines to your Gemfile.

```ruby
# Foundation Rails Gem
gem 'foundation-rails'
# Required due to an incompatibility between TurboLinks and Foundation.
gem 'jquery-turbolinks'
```

Then run `bundle install`.  

Lastly you can run an installer the the gem provides with: `rails g foundation:install`

Rails will provide a warning that the app/views/layouts/application.html.erb file will be overridden.  This is because the installer will add the required Javascript & css files.  

```bash
$  rails g foundation:install
Running via Spring preloader in process 7609
      insert  app/assets/javascripts/application.js
      append  app/assets/javascripts/application.js
      create  app/assets/stylesheets/foundation_and_overrides.scss
      create  app/assets/stylesheets/_settings.scss
      insert  app/assets/stylesheets/application.css
    conflict  app/views/layouts/application.html.erb
Overwrite /Users/Chris/ada/StudentWork/Foundation/foundation_example/app/views/layouts/application.html.erb? (enter "h" for help) [Ynaqdh] Y
       force  app/views/layouts/application.html.erb
   identical  app/assets/stylesheets/foundation_and_overrides.scss
   identical  app/assets/stylesheets/_settings.scss
```

The installer adds the following files to your app/assets folder.  

![foundation files](images/foundationfiles.png)


### Then make sure to restart your rails server, if it's running. 

## A Little Configuration

There are a few things to get setup before we can take full advantage of Foundation.  

1.  Go to `app-->assets-->stylesheets` and rename `application.css` to `application.scss`
	- This will make the CSS file a scripted CSS file called a [Syntactically Awesome Stylesheet](http://sass-lang.com/) giving us variables we can use to set fields in our stylesheets.  
1.  Then go to `app-->assets-->stylesheets-->foundation_and_overrides` and uncomment out 3 lines for the `motion-ui` library.
	- `@import 'motion-ui/motion-ui';`
	- `@include motion-ui-transitions;`
	- `@include motion-ui-animations;`

1.  Lastly add the following code to the bottom of `app-->assets-->javascript-->application.js`.
  
```javascript
//= require foundation
$(document).foundation();

$(document).on('turbolinks:load', function() {
  $(function(){ $(document).foundation(); });
});
```
	- This makes sure that the Foundation Javascript is called whenever you enter a new page.  Rails 4 has a feature called turbo links which loads views without refreshing the page.  


## Example Styling

### Drop-Down Menus

You can create drop-down menus for your app with code like this:


```erb
    <!-- Navigation row for top Menu -->
  <nav class="row">
      <section class="small-12 columns">
        <!-- Make this unordered list horizontal at medium and larger screens & verticle on small ones  -->
      <ul class="vertical menu medium-horizontal" data-responsive-menu="accordion medium-dropdown">
        <li>
          <%= link_to "Home", root_path %>
          <ul class="menu vertical">
            <li>
              <%= link_to "Orbit Example", "/orbit" %>
            </li>
          </ul>
        </li>
    <li>
      <%= link_to "Tasks", tasks_path %>
      <!-- Submenu under "Tasks" -->
      <ul class="menu vertical">
        <li>
          <%= link_to "New Task", new_task_path %>
        </li>
      </ul>
    </li>
    <li><%= link_to "Google", "https://www.google.com" %>
      <ul class="menu">
        <li><%= link_to "Google Drive", "https://drive.google.com" %></li>
      </ul>
    </li>
    <li><%= link_to "Facebook", "https://www.facebook.com" %></li>
    <li><%= link_to "Log in via Github", "auth/github" %></li>
  </ul>
  </section>
  </nav>
```

### Orbits

Orbits are a fancy way to provide a carousel effect in your page, it can be a mix of text and images.

```erb
<!-- Wrapper Around Orbit -->
<div class="orbit" role="region" aria-label="Chuck Norris Quotes" data-orbit>
  <!-- Previous and Next Buttons -->
  <button class="orbit-previous"><span class="show-for-sr">Previous Slide</span> &#9664;&#xFE0E;</button>
<button class="orbit-next"><span class="show-for-sr">Next Slide</span> &#9654;&#xFE0E;</button>

<!-- Unordered List of photos or text for carousel -->
  <ul class="orbit-container">
    <% num = 0 %>
    <% @orbits.each do |quote| %>
      <li class="orbit-slide <%= 'is-active' if quote == @orbits[0] %>">
        <%= image_tag "Orbit#{num}.jpg", alt: "Space" %>

        <figcaption class="orbit-caption"> <%= quote %></figcaption>
      </li>
      <% num += 1 %>
    <% end %>
  <!-- More slides go here... -->
</ul>
<!-- Navigation Buttons for Accessibility -->
<nav class="orbit-bullets">
  <button class="is-active" data-slide="0"><span class="show-for-sr">First slide details.</span><span class="show-for-sr">Current Slide</span></button>
  <button data-slide="1"><span class="show-for-sr">Second slide details.</span></button>
  <button data-slide="2"><span class="show-for-sr">Third slide details.</span></button>
</nav>
</div>

```

## Foundation Variables

In the `App-->assets-->stylesheets-->_settings.scss` file you can find a variety of variables in Foundation.  These variables change how the framework works in terms of color, and styling.

They are set by:  $VARIABLE_NAME:  VALUE;

So you can change if the Accordion Menu has arrows with this variable:
```scss
$accordionmenu-arrows: false;
```

And you can change the body font-family with:
```css
$body-font-family: 'Helvetica Neue', Helvetica, Roboto, Arial, sans-serif;
```

Mostly however you will use these variables rarely to turn off or on features.   You can override styling with your own styles in `application.scss`.  


