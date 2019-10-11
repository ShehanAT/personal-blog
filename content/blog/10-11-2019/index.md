---
title: JavaScript AJAX Form Submission in Ruby On Rails 
date: "2019-10-11T22:12:03.284Z"
description: "How to submit forms using AJAX in Ruby On Rails"
---

Form submission in Ruby On Rails can be done in one of two ways: locally or remotely. To submit locally means to specify a url determined by predefined routes then submit one request to be received by an action, but to submit remotely means to send two requests: one from the form itself via Rails routing and the other with unobtrusive JavaScript AJAX calls. Today we will cover how to implement remote form submission with JavaScript AJAX calls in Ruby On Rails.

![JavaScript AJAX joke ](./img1.jpg)
<p align="center">Figure 1: Some JavaScript Humor</p>

### Introduction 
As usual we will be using the Article application made in earlier posts which will hold our form. If you are feeling lost you can generate a scaffold application by running the command:```rails generate article```. This will generate a scaffolded application with an ```article``` model, some migrations, and appropriate controllers and views so you can build your RESTFUL application with minimal setup time. We will build a login form with form submission handled through unobtrusive JavaScript AJAX calls.  

### Setting Up
These are the files and folder structure we will need:
```
|--app
    |--controllers
        |-- sessions_controller.rb
    |--views
        |-- sessions
            |-- new.html.erb
            |-- new.js.erb
            |-- index.html.erb
    |--models
        |-- article.rb
        |-- session.rb
```

In the ```sessions_controller.rb``` file we will need the ```new```, ```create``` and ```index``` actions:
```
def new 
    respond_to do |format|
        format.html { render "new" }
        format.js { render "new" }
    end 
end 
 
def create 
    if User.authenticate(params[:username], params[:password])
        redirect_to sessions_path      
    else     
        respond_to do |format|
            format.html { render "new" }
            format.js { render "new" }
            format.json { render json: { message: "Invalid Credentials!" } }
        end     
    end 
end 


def index
end 
```

If a render call is not used in an action, Rails automatically looks to render a html.erb file corresponding to the action's name, in this case ```new.html.erb``` will be rendered upon navigating to ```/sessions/new```. 

Next lets setup up our view files: in ```sessions/new.html.erb``` add this form:
```
<%= form_with url:"/sessions" do |form|%>
    <div id="alert_message"> 
    </div>
    <%= form.label(:username, "Username:") %>
    <%= form.text_field(:username)  %>
    <%= form.label(:password, "Password:") %>
    <%= form.text_field(:password)  %>
    
    <%= form.submit "Login", data: { "disable-with": "Loading..." } %>
<% end %>
```
and for the ```sessions/index.html.erb``` file, add this:
```
<h1>Welcome</h1>

```
The ```index.html.erb``` file is rendered upon a successful login attempt. 

Note the ```remote: true``` option specified in the ```form_with``` helper of ```new.html.erb```. This serves to tell Rails that we will handle the form submission remotely using JavaScript AJAX calls. Although Rails assumes remote form submission by default it is best to explicitly specify the method of submission to improve readability and maintenance.  

Now comes the part where we define the AJAX call in the ```new.js.erb``` file:
```
$.ajax({
    url: "/sessions",
    method: "post",
    data: {
    },
    dataType: "json",
    success: function(data){
        $("#alert_message").text(data.message);
    }
});
```
This is the AJAX call that will send the POST request to the ```create``` action in the sessions controller. If the authentication is successful(we will leave out the implementation of User.authenticate for now) the browser redirects to the ```index``` action, if not the ```new.html.erb``` is re-rendered and the AJAX callback function is executed. This function will display the message "Invalid Credentials!" in the ```div#alert_message``` element to notify the user that authentication has failed. 

Note: this implementation assumes you have the ```jquery-rails``` gem already installed, if not go their [github page](https://github.com/rails/jquery-rails) to install it. 

To test this form navigate to ```/sessions/new``` and enter the credentials of a registered user and the browser should redirect to ```/sessions```. Enter some invalid credentials and you should see the message "Invalid credentials" rendered in the form, which was done through the AJAX callback function. 

So as you see JavaScript AJAX calls are very useful for form submissions and validations. Another plus of using AJAX calls are that browser does not have to refresh in order to submit the form, which gives a smoother and faster experience for end-users. 


### Closing Notes 
Although this is only a minor implementation of using JavaScript AJAX calls for form submissions I hope this encourages you to look at and implement more advanced examples which will surely improve the applications we build using Ruby On Rails, whether that be through making the application more RESTFUL or creating a smoother end user experience.


Well that's all for today, I hope you found this review helpful. I would greatly appreciate if you could check out my [Youtube channel](https://www.youtube.com/channel/UCtxed_NljgtAXrQMMdLvhrQ?), follow me on [Twitter](https://twitter.com/Shehan_Atuk), [LinkedIn](https://www.linkedin.com/in/shehan-a-780622126/), [Github](https://github.com/ShehanAT) and [Instagram](https://www.instagram.com/shehanthewebdev/).
