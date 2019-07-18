

*One of the developers on your team has put together a very simple prototype for a system that writes and reads to a database. The developer is using Postgres for the backend database, the Python Flask framework as an application server, and nginx as a web server. All of this is developed with the Docker Engine, and put together with Docker Compose.* 

# Steps to run the system

Assuming you have the Docker Engine and Docker Compose already installed, steps for running the system are the following: 

Open a terminal, `cd` into this repo, and then enter these two commands:

    docker-compose up -d db
    docker-compose run --rm flaskapp /bin/bash -c "cd /opt/services/flaskapp/src && python -c  'import database; database.init_db()'"

This "bootstraps" the PostgreSQL database with the correct tables. After that you can run the whole system with:

    docker-compose up -d

At that point, the web application should be visible by going to `localhost:8080` in a web browser. 

## Thought processes and problems found
*System overview.*: This simple Flask application has 3 components: The nginix proxy server, the flask web server and the database server(Postgres). Each of these components is a Docker container. Client requests first go to the nginix server(a proxy) and get redirected to the web server. The web server reads and writes from/to the database server to support users queries and data persistence.

My first thought is to get the app running and make it accessible from my computer and learn enough about the underlying technologies(specifically Docker, Flask, nginix) to let me do just that. 

As the developer mentioned, the app will run on port 8080 on localhost. I need to map port 8080 to nginix port 80 which is the default port nginix listens to for incoming http request.

To do that I fix the port mapping of the nginix service in docker-compose.yml to use "8080:80" instead of "80:8080"(this maps host port 80 to nginix container port 8080 which is not what we want). 

Another bug I found is in the nginix config file, the proxy passes requests to port 5001 of the web server. Since the developer did not set the port for the flask web server to 5001, the web server will listen to port 5000 only(the default port). To connect them properly, I made it explicit in the web server to use port 5000 and updated the nginix config file to pass requests to port 5000 on the web server.

The app now shows up correctly in my browser with the url localhost:8080. But it doesn't show all items added so far after a client enters a new item. (This seems to be the intention of the developer as she/he made a query to get all items). I added some code to loop over the query result and made the server to return all items in a list. 
