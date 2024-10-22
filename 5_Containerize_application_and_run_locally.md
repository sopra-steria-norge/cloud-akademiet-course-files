# Add Docker support for your application and run container locally with its own database - HOW TO

## Starting branch
[workshop-run-container-local-dev](https://github.com/sopra-steria-norge/WhoOwesWhat-net8/tree/workshop-run-container-local-dev)

What we are going to do in this part of the course:
- Add Dockerfile to our application in the root folder
- Verify Dockerfile
- Inspect relevant scripts and YAML-files related to containerization
- Run MS SQL database in separate container in the same network as the service so that the service and database can communicate
- Run application WhoOwesWhat in its own container and make sure the health endpoints work and that you can execute read & write operations

## Add Docker support...
In Visual Studio we can use built-in functionality to generate a Dockerfile that fits our project.
![Add Docker support...](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/run-container-local-dev/add-docker-support.png)
![Dockerfile in root folder](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/run-container-local-dev/docker-support-on-root-folder.png)

## Inspect scripts and docker compose files
Go through the files related to docker compose, the Dockerfile and .bat scripts. 

## Create MS SQL image and run in a container
- Our script will run this file: [Docker compose YAML for database (with database credentials)](https://github.com/sopra-steria-norge/WhoOwesWhat-net8/blob/main/database/docker-compose.yml)
- Script to create mssql image and run in container: [docker-compose.up.bat](https://github.com/sopra-steria-norge/WhoOwesWhat-net8/blob/main/database/docker-compose.up.bat)

## Verify that you can connect to the database from your local environment
- Enter Microsoft SQL Server Management Studio (SSMS). 
- Log into your database that runs in the container you set up. 
- From your local environment the container is seen as localhost, with the port we specified in our docker-compose setup - port `1434`. 
- Use the user and password that we specified in our docker-compose setup files (same as the variables in the connectionstring our containerized application uses).
[Docker compose YAML for database (with database credentials)](https://github.com/sopra-steria-norge/WhoOwesWhat-net8/blob/main/database/docker-compose.yml)
- The WhoOwesWhat database wil not exist yet, since the application needs to run, seed database and apply migrations to create the database. We will come to this later. The most important thing in this section is verifying that you can connect to the database. 

**Login to containerized database via SSMS:**
![Login - containerized database](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/run-container-local-dev/login-container-db.png)

## Create WhoOwesWhat image and run our application in a container
- Our script will run this file: [Docker compose YAML for application WhoOwesWhat](https://github.com/sopra-steria-norge/WhoOwesWhat-net8/blob/main/docker-compose.yml)
- Script to create mssql image and run in container: [docker-compose.up.bat](https://github.com/sopra-steria-norge/WhoOwesWhat-net8/blob/main/docker-compose.up.bat)

## Verify that contiainer runs and enter the Swagger API for WhoOwesWhat
1. Verify that migrations has been run and that the database for WhoOwesWhat is created

**Verify database connection and that database is created:**
![Verify database connection and that database is created](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/run-container-local-dev/verify-database-created.png)

2. Test WhoOwesWhat endpoints:
- Test health endpoints
- Create a user
- Authorize / Get the user you created
- This URL should work now: [Swagger URL for localhost:5025](http://localhost:5025/swagger/index.html) 

