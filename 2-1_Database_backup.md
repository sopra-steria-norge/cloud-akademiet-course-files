# Restore Database - HOW TO

## Connect to the SQL Server
Open Microsoft SQL Server Management Studio 2020.
You will be prompted with a login screen, check `Trust server certificate` and click `Connect`

![Connect server](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-backup/server-connect.png)


## New database
### Step 1
After connecting to your database, right click the server and select `New Database...`

![New database step 1](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-backup/new-database.png)

### Step 2

Give it the name `WhoOwesWhat` and click `OK`

![New database step 2](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-backup/new-database2.png)

## Restore database

### Step 1
Right click the `Databases` folder and select `Restore Database...`

![Restore database step 1](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-backup/restore-database.png)

### Step 2
Select `Device` and navigate to where the `whooweswhat.bak` file is located. It should be in `cloud-akademiet-course-files/files/whooweswhat.bak`. 

Clone the repo if you do not have it locally. 

Use the command `git clone https://github.com/sopra-steria-norge/cloud-akademiet-course-files.git`

It should automatically select the newly created database for you.

![Restore database step 2](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-backup/restore-database2.png)

### Step 3
Click on `Options` on the left hand side of the window, and select `Overwrite the existing database (WITH REPLACE)`.

**Make sure `Take tail-Log backup before restore` is not checked if you just created the database**

Click `OK` and you should be good to go.

![Restore database step 3](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-backup/restore-database3.png)