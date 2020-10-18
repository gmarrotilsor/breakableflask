# Providing database servers via Docker

Breakableflask can provide an SQL injection test bed using a number of different database engines (with only one engine being active for any one running iteration of the program).

The database servers can be setup in any of the usual ways if you prefer, but its also possible to quickly spin up workable servers using Docker, using the instructions below.

## Setup a PostgreSQL server

You can use Docker to create a suitable PostgreSQL server like so, using the official Docker image.

    docker run -d --name postgres -e POSTGRES_PASSWORD=password -p 127.0.0.1:5432:5432 postgres

You can now connect to the server using the credentials `postgres:password` on port `5432` at `127.0.0.1`.

After the container has been successfully launched once, you can use the following command to start it up again (e.g. if you reboot or otherwise kill the container).

    docker start postgres


## Setup a MySQL server

Setting up a MySQL server in Docker requires a few extra steps as by default the Docker image only sets a temporary root pasword and wont allow TCP connections, so there are a few extra steps after launching the container. Start like so:

    docker run -d --name mysql -p 127.0.0.1:3306:3306 mysql/mysql-server

Once this has launched and is running, run the following to get the temporary root password for the database, which you will need to immediately change (as shown in a later step).

    docker logs mysql 2>&1 | grep GENERATED | cut -d' ' -f 5

Now, run the `mysql` command within the container. Provide the password obtained in the previous step once prompted. 

    docker exec -it mysql mysql -uroot -p

Once in the mysql shell, enter the following commands to change the root password and create a new `mysql` user with a password of `password`.

    ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';
    CREATE USER 'mysql'@'%' IDENTIFIED BY 'password';
    GRANT ALL ON *.* TO 'mysql'@'%';
    QUIT

The server is now ready to use with your new credentials and is contactable on port `3306` at `127.0.0.1`.

After the container has been successfully launched once, you can use the following command to start it up again (e.g. if you reboot or otherwise kill the container).

    docker start mysql


## Setup a MSSQL server

You can use Docker to create a suitable MSSQL server like so, using the official Docker image.

    docker run -d --name mssql -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=Password1!' -p 127.0.0.1:1433:1433 mcr.microsoft.com/mssql/server:2019-latest

You can now connect to the server using the credentials `sa:Password1!` on port `1433` at `127.0.0.1`. There is a minimum complexity requirement for the SA password in this container, which the example password above meets. Make sure if you vary this when you run your container that your password meets this too, otherwise the container will refuse to run (you can see if its running using `docker ps`). 

You can check the output of `docker logs mssql` if the container fails to see if password complexity is the reason.

After the container has been successfully launched once, you can use the following command to start it up again (e.g. if you reboot or otherwise kill the container).

    docker start mssql


## Stopping and deleting Docker images

To stop a running Docker instance find the container id by checking the output of `docker ps`, and stop like so:

    docker kill [ID]

Delete the container by running the following, where [CONTAINER_NAME] is the value set when the container was created  (e.g. `mysql` from the MySQL example above):

    docker container rm [CONTAINER_NAME]