# FileMaker WebDirect Record
This is so far the easiest way to go to a specific record in FileMaker WebDirect from web application using Javascript.

## What this solves?
In a previous Filemaker verion, from an HTML page, we were able to link to a:
- specific database
- specific database, and layout,
- specific database, layout, and record
- particular database and run a script

Now the only options available are linking to a database, or linking to a database and running a script.

# How to go to a specific record in Filemaker WebDirect from an HTML page
First I will give the basic steps needed to go to a specific record from an HTML page, then I will give a couple of examples incase you are using a FORM.

## Prerequisites
1. filemaker
2. html, js, css knowledge

## Basic Steps
1. In Filemaker go to Manage Scripts / Scripts Workspace and create a new script.

2. In your HTML page, link to the newly created Filemaker script:
- this is how the link is set up. 
http://<host>/fmi/webd/<database name>[?script=<script name>[&param=<script parameter>]]
- where <script parameter> = the Recordid you want to go to
- if you want to know how to PUSH a Recordid into the script parameter click HERE.
This is directly from FileMaker WevDirect 18 Guide
https://fmhelp.filemaker.com/docs/18/en/fmwd/#accessing_linking
  
3. In Filemaker go to script you created
![GitHub Logo](/images/go_to_record.png)
Format: ![Alt Text](url)
 
