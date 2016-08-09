+++
date = "2016-07-29T16:18:23+05:30"
draft = true
title = "Heroku Connct : Sync Heroku app with Salesforce"

+++

This workshop shows how to **Create** and **Run** a Python app with psycopg2 which uses PostgreSQL based Heroku Connect

<img src="images/heroku-connect-flow-flask-psycopg2.png" width="70%" height="70%">
   
Figure 1 show the  how HerokuConnect Add-On interacts with Heroku Postgres and force.com behind the scenes
Make sure you have Python installed.  Also, install the [Heroku Toolbelt](https://toolbelt.heroku.com/)

## Install Virtual Environment

Create a folder `flask-psycopg2-sample` and install a virtualenvironment in it.

  ``` bash

    $ mkdir flask-psycopg2-sample
    $ cd flask-psycopg2-sample
    $ virtualenv venv
    $ source venv/bin/activate

  ```

Install Dependencies

  ``` bash

  $ pip install flask gunicorn

  ```
  
## Creating a Simple Flask App

1. First Create a base Flask app with simple REST endpoint/ in a file `app.py` in the folder created above.
  
    ``` python
    
    from flask import Flask
    app = Flask(__name__)

    @app.route('/')
    def hello_world():
        return 'Hello World!'

    if __name__ == '__main__':
        app.run()
    ```


2. Run the app using the following command
 
    ``` bash
    $ python app.py
    ```

  Your app should now be running on [localhost:5000](http://localhost:5000)

## Initialize git

Initialize the git repository as shown by commands below.

  ``` bash
  $ git init
  $ git add .
  $ git commit -m "initial commit"

  ```
## Create a Requirements File

  ``` bash
  $ pip freeze > requirements.txt
  ```
  
## Create a Procfile

Create a file name `Procfile` in the root of the app and add following content. This specifies that the app uses a `web` dyno with `gunicorn` as http server.

  ``` bash
  web: gunicorn app:app --log-file -
  ```
  
## Deploying to Heroku

  ``` bash
    $ heroku create
    $ git push heroku master
    $ heroku open
  ```
## Add PostgreSQL Add-On

Add Postgress Add-On as shown below

  ``` bash
  $ heroku addons:create heroku-postgresql:hobby-dev
  ```

## Add Heroku Connect Add-On

Configure Heroku Connect Add-On. Command below configures Herok-Connect Add-On to the application.

  ``` bash
  $ heroku addons:create herokuconnect
  ```

## Configure Heroku Connect Add-On

1. Setup Connection
   
<img src="images/setup-connection.png" width="700" height=250> 

2. Enter Schema Name : This is the schema name underwhich database will be created.

<img src="https://github.com/dbhasuru/df16workshops/blob/master/content/workshop/pythonapp/images/enter-schemaname.png" width="700" height=250> 

3. Trigger OAuth 

   <img src="https://github.com/dbhasuru/df16workshops/blob/master/content/workshop/pythonapp/images/trigger-oauth.png" width="700" height=200>  
4. Enter Salesforce.com developer account credentials

   <img src="https://github.com/dbhasuru/df16workshops/blob/master/content/workshop/pythonapp/images/sfaccountdetails.png" width="300" height=400> 
5. Create Mappings

   <img src="https://github.com/dbhasuru/df16workshops/blob/master/content/workshop/pythonapp/images/create-mappings.png" width="700" height=400>   
6. Create Mappings Contacts : Choose the fields in Salesforce Schema which need to be mapped to Postgres Database in the application.

   <img src="https://github.com/dbhasuru/df16workshops/blob/master/content/workshop/pythonapp/images/create-mapping-contacts.png" width="700" height=500>  
7. Explore Contacts in the Dashboard

   <img src="https://github.com/dbhasuru/df16workshops/blob/master/content/workshop/pythonapp/images/contacts-explorer.png" width="700" height=500> 

## Add Code for contacts endpoint 

  First Add following lines which configure Connection object conn to PostgreSQL Database.
  
  ```python
    url = urlparse.urlparse(os.environ.get('DATABASE_URL'))
    db = "dbname=%s user=%s password=%s host=%s " % (url.path[1:], url.username, url.password, url.hostname)
    schema = "schema.sql"
    conn = psycopg2.connect(db)
    cur = conn.cursor()
  ```
  
  Add code for the Getting the Contacts.
  
  ```python
    @app.route('/contacts')
    def contacts():
        try:
          cur.execute("""SELECT name from salesforce.contact""")
          rows = cur.fetchall()
          response = ''
          my_list = []
          for row in rows:
              my_list.append(row[0])
          return render_template('template.html',  results=my_list)
        except Exception as e:
          print e
          return []
```
Complete Code listing

  ```python
    import os
    import psycopg2
    from flask import Flask, render_template
    import urlparse
    from os.path import exists
    from os import makedirs
    
    url = urlparse.urlparse(os.environ.get('DATABASE_URL'))
    db = "dbname=%s user=%s password=%s host=%s " % (url.path[1:], url.username, url.password, url.hostname)
    schema = "schema.sql"
    conn = psycopg2.connect(db)
    
    cur = conn.cursor()
    
    app = Flask(__name__)
    
    @app.route('/')
    def hello():
        return 'Hello World!'
    
    @app.route('/contacts')
    def contacts():
      try:
        cur.execute("""SELECT name from salesforce.contact""")
        rows = cur.fetchall()
        response = ''
        my_list = []
        for row in rows:
          my_list.append(row[0])
        return render_template('template.html',  results=my_list)
      except Exception as e:
        print e
        return []
    if __name__ == '__main__':
      app.run()
  ```
  
## Add Jinja Template 

  The code shown in previous section uses template.html file which is a Jinja template. Add this file under folder templates
  
  ```html
    <html>
      <head>
        <title>Flask Template Example</title>
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link href="http://netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap.min.css" rel="stylesheet" media="screen">
        <style type="text/css">
          .container {
            max-width: 500px;
            padding-top: 100px;
            }
        </style>
      </head>
      <body>
        <div class="container">
          <p>Contacts:</p>
          <ul>
            {% for r in results %}
            <li>{{r}}</li>
            {% endfor %}
          </ul>
        </div>
        <script src="http://code.jquery.com/jquery-1.10.2.min.js"></script>
        <script src="http://netdna.bootstrapcdn.com/bootstrap/3.0.0/js/bootstrap.min.js">
        </script>
      </body>
    </html>
```

## Update python packages

  `$ pip install psycopg2`
  
## Add Requirements file

  `$ pip freeze > requirements.txt`
  
## Create a Procfile

  Create a Procfile which will be used to identify the web dyno and type of runtime
  
  ` web: gunicorn app:app --log-file=-`

## Update Changes in Heroku

  ```
    $ git add .
    $ git commit -m "Added code for contacts"
    $ git push heroku master
  ```
  
  Open the App again in Heroku
  
  `$ heroku open`
  
## Show Contacts

  Browse to URL `http://{your-app-name}.herokuapp.com/contacts` to see the list of contact names.
  <img src="https://github.com/dbhasuru/df16workshops/blob/master/content/workshop/pythonapp/images/show-contacts.png" width="700" height=500> 
## Summary

  In this tutorial we learnt how to configure a Python Flask Application to work with Heroku Connect. We used Psycopg2 driver for talking to the PostgreSQL database deployed on Heroku.

  

  
  

