---
title: Basic Integration Testing For Ruby On Rails Apps Part 2
date: "2019-09-24T22:12:03.284Z"
description: "Implementing basic integration tests using Capybara."
---

Capybara is a powerful testing framework for Ruby on Rails applications. This post will explain how to write basic integration tests using Capybara. So let's get right to it! 

![Testing diagram](./img1.jpg)
<p align="center">Side note: Capybaras are the largest living rodents in the world.</p>

### Check for presence of elements in page 
Building on the previous article application, let's say we have a login form and want to check if the clicking of the ```Login``` button redirects to a different page. We can test this using Capybara. I will show what to do first then explain my steps later as this keeps these posts nice and to the point. 

Let's create a controller file: ```app/session/session_controller.rb``` which will hold the login action needed to authenticate users. I will skip the implementation of the User model and the login functionality because it is beyond the scope of this post. The controller file will contain three actions ```new```, ```create``` and ```welcome```, which are shown below:
```
def new 
end 

def create
    @userToRender = User.authenticate(params[:username], params[:password])
    respond_to do |format|
        if @userToRender
            session[:username] = @userToRender.username
            session[:user_id] = @userToRender.id
            format.html { redirect_to "/welcome" }
        else 
            format.html { render "new" }
        end 
    end  
end

def welcome
end 
```

Upon calling the create action the user is redirected to the route: ```/welcome```. Now let's add three routes to the ```config/routes.rb``` file:
```
  get "new", to: "session#new"
  post "new", to: "session#create"
  get "welcome", to: "session#welcome"
```
We will also need two view files:```app/views/session/new.html.erb``` and ```app/views/session/welcome.html.erb```.  Let's add a login form to the ```new.html.erb``` file:
```
<%= form_with( method: "post", id: "login_form") do %>
   
    <div id="alert_message"> 
    </div>
    <%= label_tag(:username, "Username:") %>
    <%= text_field_tag(:username)  %>
    
    <%= label_tag(:password, "Password:") %>
    <%= password_field_tag(:password)  %>
    
    <%= submit_tag "Login", data: { "disable-with": "Loading..." }, remote: true %>
<% end %>
```
and a simple welcome message to the ```welcome.html.erb``` file:
```
<h1>Welcome to My Application!</>
```

Now with the login functionality setup up let's write the integration test to test for a redirection on button click. In the ```app/test/controllers/session_controller_test.rb``` file add this:
```
test "should redirect from /new to /welcome on login button click" do 
    Capybara.visit("/new")
    Capybara.fill_in 'username', :with => 'admin'
    Capybara.fill_in 'password', :with => 'admin'
    old_path = Capybara.page.current_path
    Capybara.page.first("input[type='submit']").click
    sleep 0.1
    new_path = Capybara.page.current_path
    assert_not_equal(old_path, new_path)
end 
```

Run this test file specifically with ```rails test test/controllers/session_controller_test.rb``` and you should see 
```
3 runs, 3 assertions, 0 failures, 0 errors, 0 skips
``` 

So with our test passing let's go over what we just did. First we created the login functionality, that part is pretty self explanatory. Then in the writing of the test we declared ```Capybara.visit('/new')``` which indicates to Capybara to visit the url ```localhost:3000/new```. Now the ```new.html.erb``` file, containing the login form, will be rendered by the browser. The ```Capybara.fill_in``` method does exactly what it sounds like: it fills in the input for a specified element on the page. Note that filling in the form fields at this point won't actually validate the user as the purpose of this example is to demostrate the functionality of Capybara and not login authentication. Before we click the submit button we have to save the current path of the url so that we have a reference to compare to if the redirection actually works, which is done through the variable ```old_path```. By the way, ```Capybara.page.current_path``` gets the path of the current url. Now, the ```Capybara.page.first("input[type='submit']").click``` line tells Capybara to find the first element of type ```input[type='submit']``` and click it. This should trigger the redirection event to occur. In order to check for this we first need to tell Capybara to pause its execution for 0.1 seconds. This is to ensure the application has time to process the click event and respond to it before we check for the url path again. Then the path of the new url is saved to the ```new_path``` variable and is compared with ```old_path``` through the use of ```assert_not_equal```. Since the test passed, we know that ```new_path``` did not equal ```old_path``` meaning the redirection was successful.  

I will stop here for the sake of brevity but I encourage you to look further into Capybara. It's documentation can be found [here](https://rubydoc.info/github/teamcapybara/capybara/master). 

Well that's all for today, I hope you found this review helpful. I would greatly appreciate if you could check out my [Youtube channel](https://www.youtube.com/channel/UCtxed_NljgtAXrQMMdLvhrQ?), follow me on [Twitter](https://twitter.com/Shehan_Atuk), [LinkedIn](https://www.linkedin.com/in/shehan-a-780622126/), [Github](https://github.com/ShehanAT) and [Instagram](https://www.instagram.com/shehanthewebdev/).
