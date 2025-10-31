## In order to run the backend:
1. Modify the **appsettings.json** by adding proper Admin's email and password. They can be taken from somebody else who already has them. You can change the EmailConfiguration and the ConnectionString as well
2. In order to prepare your docker configuration you can run the following commands:
   1. ```docker volume create photopixels-dev``` to create the volume (if needed)
   2. ```docker run --name photopixels-dev -e POSTGRES_PASSWORD=mysecretpassword -p 5435:5432 -v photopixels-dev:/var/lib/postgresql/data -d postgres``` to run the container
      - **POSTGRES_PASSWORD** value should be the password from the ConnectionString
3. Run the backend and make sure it runs well by checking if the database schema has been created successfully (In the provided from the connection string database, there should be **photos** schema full with tables which starts with **mt_doc_**).

## In order to run the frontend
1. Change the .env file with the **REACT_APP_SERVER** equal to the url of your backend. (it should be something like https://localhost:7290/)
2. Type ```npm start ``` to run the app
3. Open **http://localhost:3000/login** to login. Use the admin credentials from the backend's appsettings.json file (if you don't have them, request such from someone who already has them)

## Possible issues
If you are unable to push the backend due to TUS repo rights, run
```shell
git submodule update --init --recursive
```