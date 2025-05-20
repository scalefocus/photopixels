# Photopixels
Main repo for the Photopixels

## Documentation
1. Install python3
    - Windows - [Python Download](https://www.python.org/downloads/)
    - Linux - Look online for installation methods for your disribution.

2. Install pip 
    - Windows  - 'python get-pip.py'
    - Linux  - Something like 'sudo apt-get install python3-pip' for debian descendants OR 'sudo dnf install python3-pip' for Fedora
3. Install mkdocs - 'pip install mkdocs'
4. Install desired theme - 'pip install mkdocs-material'
5. Navigate to the repo docs folder and execute - 'mkdocs new .'. It will create **mkdocs.yaml** and **docs** folder with **index.md**
5. Edit **mkdocs.yml** according to the documentation found at [mkdocs user guide](https://www.mkdocs.org/user-guide/)
6. Create the desired MD files in the docs folder and adjust **docs/docs/index.md**
7. Execute in terminal 'mkdocs build' - it will create **docs/site** directory which is EXCLUDED in gitignore.
8. Publish the docs/site directory to your web server. 