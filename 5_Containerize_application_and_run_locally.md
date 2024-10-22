# Add Docker support for your application and run container locally with its own database - HOW TO

## Starting branch
[workshop-run-container-local-dev](https://github.com/sopra-steria-norge/WhoOwesWhat-net8/tree/workshop-run-container-local-dev)

What we are going to do in this part of the course:
- Add Dockerfile to our application in the root folder
- Verify Dockerfile
- Inspect relevant scripts and YAML-files related to containerization
- Run MS SQL database in separate container in the same network as the service so that the service and database can communicate
- Run application WhoOwesWhat in its own container and make sure the health endpoints work and that you can execute read & write operations


![Add Docker support...](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/run-container-local-dev/add-docker-support.png)

![Verify database connection and that database is connected](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/run-container-local-dev/verify-database-created.png)



Pull .net 8 branch
Move to docker-workshop branch
Add docker file (docker support...)
Verify docker file
Open visual studio code
Inspect scripts and docker compose files
Run database --> docker.bat, verify data base container by logging on to database `localhost,1434`

Run whooweswhat container --> .bat, verify that the database has been created and that you can use the endpoints

