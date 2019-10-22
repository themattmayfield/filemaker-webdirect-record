# FileMaker WebDirect Record
This is so far the easiest way to go to a specific record in FileMaker WebDirect from web application using Javascript.

## Table of Contents
[What this solves?](#what-this-solves)
[Prerequisites](#Prerequisites)

### What this solves?
In a previous Filemaker verion, from an HTML page, we were able to link to a:
- specific database
- specific database, and layout,
- specific database, layout, and record
- particular database and run a script

Now the only options available are linking to a database, or linking to a database and running a script.

### Prerequisites
1. FileMaker Advanced 16,17, or 18
2. Basic knowledge on JavaScript, HTML, CSS
3. Basic FileMaker knowledge

## How to go to a specific record in Filemaker WebDirect from an HTML page
First I will give the basic steps needed to go to a specific record from an HTML page, then I will give a couple of examples incase you are using a FORM.

### Basic Steps
1. In Filemaker go to Manage Scripts / Scripts Workspace and create a new script.

2. In your HTML page, link to the newly created Filemaker script:
- this is how the link is set up. 
http://<host>/fmi/webd/<database name>[?script=<script name>[&param=<script parameter>]]
- where <script parameter> = the Recordid you want to go to
- if you want to know how to PUSH a Recordid into the script parameter click HERE.
This is directly from FileMaker WevDirect 18 Guide
https://fmhelp.filemaker.com/docs/18/en/fmwd/#accessing_linking
  
3. In Filemaker go to script you created.
- below is an image of what should go into your script.
- you can change and manipulate to match your environment, and situation, but this is a basic outline.
![Script](/img/go_to_record.png)

4. As you can see this will get your id parameter from the url and go to that record.

## How to get a users ID into the url to pass to filemaker.
The most common reason someone wants to go to a specific record from an HTML page is that they have a login form and wants the user that logs in to go straight to the FileMaker record specific to them.
###### So Let's Do That.

### Things to keep in mind.
1. We are only going to be using email to login the user. You can use the same logic to sign them in with email and pass, or whatever credentials of your choosing, but for now we will use just email.
2. When a user logs in, they access the FileMaker database using the credentials of that privilege set. So if necessary, create a guest account with a limited privilage set.
3. I will not be going over how to make this more secure, that is on you. However with that being said, FileMaker has made this very secure already.

### Steps to get a users ID into the url to pass to filemaker.
1. Create a form.
- like I said in the Thing to keep in mind section, I will only be using email because I am lazy.
- here is a very basic example:
![login.png](/img/login.png)

- keep in mind you can name your html page whatever you like this is just an example.
- this form has an email input and a submit button. Notice the form does not have an action or method.
- also I have a script that includes a JavaScript file. Your path and name of file can be whatever you like, but do create a JavaScript file and include it.

2. Below is what goes in the javascript file you just created. I have solid comments within the code, but there are a few things I want to explain that took me a while to figure out. So PLEASE READ
- where we create the variable session, make sure you fill out the url properly with the host being prefixed with "https://".
- also in session.Authorization it is going to have the work Basic followed by the base64-encoded string of your account name and password of the database sepereated by a colon which looks like... "Basic accountname:password". So if your account name is "admin" and your password is "admin" your authorization should look like below. You can find out the base64-encoded string of your account name and password by some online converter. Documentation of this is in the FileMaker 18 DATA API Guide https://fmhelp.filemaker.com/docs/18/en/dataapi/#connect-database_log-in
- when creating a session, we use the authorization that uses Basic accountName:password like we talked about, but when getting a range of records, we use a token. We are going to get the token from the response we get when we create the session, and insert it into the variable we will create to get the range of records.

```javascript
//---// Data needed to Create SESSION. IMPORTANT to pull token from response //---//
//---// Fill in with correct FileMaker Credentials //---//
var session = {
  "url": "<YOUR HOST>/fmi/data/v1/databases/<YOUR DATABASE>/sessions",
  "method": "POST",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Basic YWRtaW46YWRtaW4=" //this is if your username and password are both admin
  }
}
//--// When the user clicks submit, it will execute this code. //--//
$( "form" ).submit(function( event ) {

//stops form from doing normal form duties
event.preventDefault();

//simple function for if you have more form data, so you call this funciton
//whenever you set a variable to what the user input.
//it incrememnts to the next piece of data.
var num = 0
function x() {
  return num++;

}

// sets a new variable email to the value the user typed in.
//called the funciton x() so it gets the first data in the form.
//call whenever there is more form data.
  var email = $( this ).serializeArray()[x()].value;


  //---// Ajax makes request and gives response then pulls token from response) //---//
  $.ajax(session).done(function (response) {
    token = (response.response.token);

  //---// Data needed to GET records //---//
    var records = {
      "url": "<YOUR HOST>/fmi/data/v1/databases/<YOUR DATABASE>/layouts/<YOUR LAYOUT>/records",
      "method": "Get",
      "headers": {
        "Content-Type": "application/json"

      }
    }
  //---// Insert correct Authorization into header of RECORDS "Bearer token" //---//
  records.headers.Authorization = "Bearer " + token;

  //---// Ajax makes request for records) //---//
    $.ajax(records).done(function (response) {
      //Set variable data to all FIleMaker data we got
      var data = (response.response.data)

      // loop to see if email exist within the FIleMaker records 
    for (var i = 0; i < data.length; i++) {
      //if email exist we execute code
      if (email.toUpperCase() == data[i].fieldData.Email.toUpperCase() ) {
      //set variable id to the FIleMaker RecordID associated with that email
        var id = response.response.data[i].recordId;

      //store the id ans essentially send it to the new webpage we will open.
        localStorage.setItem("id",id);
        //open a new HTML that will get the id to send to webdirect.
        window.open("./WebDirect.html", "_self")
        break;
      }
      //if email does not exist, it will give an error message.
      else {
        console.log("nope");
        if(i == data.length - 1){
          console.log("stopped");
          const messages = document.getElementById('messages');
          messages.textContent = "Wrong Email";
        };
      }

    }

        })


      });


  });

```
3. Create a new HTML page. 
- as you can see in my example above I called it WebDirect.html
- when the user clicks submit with a valid email in the login.html, the JavaScript page will get the Filemaker ID associated with that email, and pass it to the new HTML page we just created.
- what this page does is it sets the ID to a new variable, and we set the form action to go to the correct script where FIleMaker will be able to go to the correct record. Then automatically submits the form.
- we discussed everything about scripts earlier assuming you read if not click HERE.

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>

<form id="myForm" method="post">
  <input type="hidden" name="user" value="admin" />
<input type="hidden" name="pwd" value="admin" />
</form>


<script type="text/javascript">
var id = localStorage.getItem("id");
document.getElementById('myForm').action = ("<YOUR HOST>/fmi/data/v1/databases/<YOUR DATABASE>?script=<YOUR FILEMAKER SCRIPTNAME>&param=" + id);
document.getElementById('myForm').submit();
</script>

  </body>
</html>

```
4. You are done!
- Created a login form
- JavaScript creates a FileMaker session that checks the credentials.
- If credentials are good, it gets that users FileMaker RecordID associated with it.
- Sends the ID to another form.
- That form automatically submits and sends the ID to FilemakerScript.
- Script handles everything and sends user to appropriate record in WebDirect.
