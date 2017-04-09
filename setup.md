---
layout: page
title: Setup
permalink: /setup/
---
The first thing will need is an account on <a href="https://imgur.com" target="_blank">Imgur</a> if you don't already have one. Go to the web site 
and select the <a href="https://imgur.com/register?invokedBy=regularSignIn" target="_blank">sign up</a> link. Follow the the instructions to register.

Once you have created an account, you will need to register 
<a href="https://api.imgur.com/oauth2/addclient" target="_blank">an application</a>. First, enter a name
for the application, which can be anything, but probably needs to be unique (e.g. containing your Net ID). Then select 
"`Anonymous usage without user authorization`" for the **`Authorization type`**. Next, enter "`https://imgur.com`" for the 
**`Authorization callback URL`**. Finally, enter an email addres and click on **`submit`**.

You should now see a new page containing a **`Client ID`**. You will need to copy this ID for use later.
