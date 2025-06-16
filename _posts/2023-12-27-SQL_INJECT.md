---
title: SQL Injection on Vulnerable Login Page
date: 2023-12-27 14:52:00 -0500
categories: [Cybersecurity]
tags: [sql, javascript, html, css, cybersecurity] # TAG names should always be lowercase
img_path: /assets/img/
---

# Intro

Before I get into any of the juicy bits, I\'d first like to apologize for what will likely be a rather
bland first post. I\'m very new to this whole blogging thing. The source code for everything in this project can be found [here](https://github.com/kaiserjd/login-form)

<p>Anyways, this whole project started as an attempt to make a simple web application
using SQL, in order to practice what I had been learning in my database course at the time.
Quicky thereafter I had the bright idea of introducing an SQL injection vulnerability for fun,
making the choice for a web application obvious: a simple login page.</p>

# Website Construction

The entire website is very simple, featuring just two pages:

![Login Page](assets/img/login-page.png){: width="1915" height="408" }
_The Login Page_
<br>

![User Page](/assets/img/user-page.png)
_The User Page_

The technology stack behind this website is pretty much what you\'d expect, with the frontend being handled by vanilla JS, HTML, and CSS, and the backend being a mixture of Node, Express, and EJS. And, as this was originally a project meant to improve my SQL skills, it also uses a mySQL server for the login credentials.

While I can all but guarantee that nearly every part of this website could be built better, (and why this project should probably serve as my certificate of disqualification from ever working in web dev), it does its primary job very well: to serve a single SQL query.

# SQL Shenanigans

Seeing as the whole point of SQL injection attacks is to manipulate a poorly constructed SQL query, I\'d like to show you the one I came up with. First, however, I think it would be a good idea to show what a well constructed query looks like in comparison.

```javascript
"SELECT * FROM users WHERE username = ? AND pword = ?",
[username, password],
```

Above is the query as it appeared in the [login.js](https://github.com/kaiserjd/login-form/blob/main/login.js) file when I originally made this site. The whole point of an SQL injection attack is to escape from wherever the user entered data is stored and execute commands. In this example, the variables `username` and `password` are _parameterized_, or stored in such a way that their values are taken literally rather than being potentially executed. This eliminates the chances of a user entering valid SQL code and having it executed by the server.

Unfortunately this way of doing things makes it rather difficult to perform an SQL injection, so we\'ll need a \'better\' query.

```javascript
 'SELECT * FROM users WHERE username ="' +
  username +
  '" AND pword ="' +
  password +
  '"',
```

The above example does not use parameterization, and as such the strings entered by the user could, if they are properly escaped, execute as part of the SQL query itself. To demonstrate this, we\'ll first look at an example of a mundane input with this query.

For the input of `johnsmith123` for the username and `jsmith123` for the password, our new query would look like:

```sql
SELECT * FROM users WHERE username = "johnsmith123" AND pword = "jsmith123"
```

This query would run correctly and all would be well. However, looking at the generated query above, we may notice that the only thing separating SQL code from the user's input is a single double quote. We can then simply close the double quote and start writing our own SQL code.

For a devious example of this in action, perhaps we intuit that a common username for administrator accounts is 'admin'. If we use `admin` as our username, we could then use what we learned previously to make the password requirement moot. Using the SQL statement `" OR ""="` as our password will result in the following query to be sent to the database:

```sql
SELECT * FROM users WHERE username = "admin" AND pword = "" OR ""=""
```

This query can effectively bypass the need for a password for any account. Because the SQL statement `OR ""=""` is always true, no password check will ever take place, and the user will be logged into the admin account without ever knowing the password.

![Admin Page](assets/img/admin-page.png){: width="1915"}
_The Admin Page_

# Goodbye

Hopefully this didn\'t bore you too much. This blogging thing is pretty neat, and honestly I find sharing what I\'ve learned to be very helpful in cementing my learnings. Please let me know if there\'s anything you think I can improve on as I continue writing, and thank you very much for reading.
