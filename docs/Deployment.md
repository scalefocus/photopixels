# Deployment steps for the backend and frontend projects

## URLs
### Backend
- Repo: https://github.com/scalefocus/photopixels-backend-net
- Dockerhub: https://hub.docker.com/r/scalefocusad/photopixels-backend-net
### Frontend
- Repo: https://github.com/scalefocus/photopixels-web
- Dockerhub: https://hub.docker.com/r/scalefocusad/photopixels-web

## Deployment Instructions
1. Create and merge the pull request.
- *For the frontend application you may create Tags. See the "Add Tags" section at the bottom of this file*

1. **(BACKEND ONLY!)** In order to run the pipeline manually go in the Backend repo. Then press **"Actions"**. Click on the **"dotNet Cl for Push"** link in left. Press on **"Run workflow"** dropdown button in top right and choose **"Run workflow"**. The pipeline should deploy the build to dockerhub.

2. Go to the dockerhub url. In **"Tags"** you can see the latest tag. You will need them in order to deploy the correct version
 - In order to have the latest version numbers, make sure there are no running pipelines

3. Open Terminal and connect to the server:
```shell
ssh <USERNAME>@photopixels.scalefocus.dev
```
- To obtain the username ask the devops or Boris to create account and credentials to photopixels.scalefocus.dev. You will also need and SSH keys for this

1. Obtain administrator privilege
```shell
sudo su <admin password provided by the devops>
```

1. Go to the dev server apps
```shell
cd /home/photopixeladmin/docker-containers/dev2.photopixels.io
```
- **dev2** can be changed to another dev environment if needed

1. Stop the application by running
```shell
docker compose down <app name>
```
- app name can be **"backend"** or **"frontend"** (see the docker compose file for the app names)

1. Open the docker compose file
```shell
vim docker-compose.yml
```

1.  Find the desired application image name and modify the tag in order to point to the latest deployment you want to run
      1. To go in edit mode, press **"i"**, then edit the text
      2. In order to save it, exit the edit mode by pressing **ESC** and then type **"w"**
      3. In order to exit after the changes has been made, pres **ESC** and then **":q"** to quit
      4. If you mess up, press **ESC** and then type **":q!"** to exit without changes

2.   Start the container by running
```shell
docker compose up - d <app name>
```
- app name can be **"backend"** or **"frontend"** (see the docker compose file for the app names)

## Add Tags
In order to have different deployment versions, each deploy should has its unique tag, which corresponds to its version number. These tags are part from the image name which later is used to run the application.

### Backend
For the backend you don't need to do anything. The tag is automatically assigned after completing a pull request.

### Frontend
You need to add tags manually. After correct tag assignment, a pipeline would be triggered, that should deploy the build to dockerhub
1. Take latest main
```bash
git fetch origin
git checkout main
git pull origin main
```

2. Create the tag
```bash
git tag -a v1.0.0 -m "Release v1.0.0"
```
- v1.0.0 should be the proper version you want to use. The text after -m is the comment

3. Push the tag
```bash
git push origin v1.0.0
```
- Here after the origin, you should type the tag name you created before