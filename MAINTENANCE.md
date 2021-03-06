# Building download latest.zip

Any time the configs, the Dockerfile or docker-compose.yml change, run this script to create a new latest.zip file.
This zip file contains only what is needed to run praeco for end users, and is linked to from documentation.

`./build_download.sh`

# Updating packages

Run `npm update` to install latest versions of packages per package.json.

# Checklist for new prod release

## Elastalert (if neccessary)

- Stash all changes in elastalert project
- Create docker image + push to servercentral/elastalert

## Praeco

- Run tests
- Recreate latest.zip if neccessary
- Stash all changes in this project; make sure in `master`
- `docker build -t praeco .`
- Create docker image + push to servercentral/praeco (Drone does this for us in master)
