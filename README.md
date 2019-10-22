# FileMaker WebDirect Record
This is so far the easiest way to go to a specific record in FileMaker WebDirect from web application using Javascript.

## What this solves?
In a previous Filemaker verion, from an HTML page, we were able to link to a:
- specific database
- specific database, and layout,
- specific database, layout, and record
- particular database and run a script

Now the only options available are linking to a database, or linking to a database and running a script.

## Prerequisites
1. FileMaker Advanced 16,17, or 18
2. Basic knowledge on JavaScript, HTML, CSS
3. Basic FileMaker knowledge

# How to go to a specific record in Filemaker WebDirect from an HTML page
First I will give the basic steps needed to go to a specific record from an HTML page, then I will give a couple of examples incase you are using a FORM.

## Basic Steps
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

# How to get a users ID into the url to pass to filemaker.
The most common reason someone wants to go to a specific record from an HTML page is that they have a login form and wants the user that logs in to go straight to the FileMaker record specific to them.
###### So Let's Do That.

## Things to keep in mind.
1. We are only going to be using email to login the user. You can use the same logic to sign them in with email and pass, or whatever credentials of your choosing, but for now we will use just email.
2. When a user logs in, they access the FileMaker database using the credentials of that privilege set. So if necessary, create a guest account with a limited privilage set.
3. I will not be going over how to make this more secure, that is on you. However with that being said, FileMaker has made this very secure already.

## Steps to get a users ID into the url to pass to filemaker.
1. Create a form.
- like I said in the Thing to keep in mind section, I will only be using email because I am lazy.
- here is a very basic example:
![login.png](/img/login.png)

- keep in mind you can name your html page whatever you like this is just an example.
- this form has an email input and a submit button. Notice the form does not have an action or method.
- also I have a script that includes a JavaScript file. Your path and name of file can be whatever you like, but do create a JavaScript file and include it.

2. 

```javascript
//---// Data needed to Create SESSION. IMPORTANT to pull token from response //---//
var session = {
  "url": "https://ws.skdata.cloud/fmi/data/v1/databases/WSdummydb/sessions",
  "method": "POST",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Basic YWRtaW46YWRtaW4="
  }
}

$( "form" ).submit(function( event ) {
  // console.log( $( this ).serializeArray() );
  event.preventDefault();

var num = 0
function x() {
  return num++;


}
  var email = $( this ).serializeArray()[x()].value;
  // var password = $( this ).serializeArray()[x()].value;




  // console.log(email);
  // console.log(password);

  //---// Ajax makes request and gives response then pulls token from response) //---//
  $.ajax(session).done(function (response) {
    token = (response.response.token);

  //---// Data needed to GET records //---//
    var customerList = {
      "url": "https://ws.skdata.cloud/fmi/data/v1/databases/WSdummydb/layouts/CustomerDetails/records",
      "method": "Get",
      "headers": {
        "Content-Type": "application/json"

      }
    }
  //---// Insert correct Authorization into header of eventRecords "Bearer token" //---//
  customerList.headers.Authorization = "Bearer " + token;

  //---// Ajax makes request for Event records) //---//
    $.ajax(customerList).done(function (response) {
      var data = (response.response.data)


    for (var i = 0; i < data.length; i++) {
      if (email.toUpperCase() == data[i].fieldData.Email.toUpperCase() ) {
        // console.log(data[i].fieldData.Email);
        var id = response.response.data[i].recordId;
        // console.log("RecordID = " + id);

        // localStorage.setItem("id",id);
        // window.open("./page.html", "_self");

        localStorage.setItem("id",id);
        window.open("./CustomerWebDirect.html", "_self")
        break;
      }
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
