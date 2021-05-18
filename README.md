# rsync deployments with ssh command

This GitHub Action deploys files in `GITHUB_WORKSPACE` to a remote folder via rsync over ssh. 

Use this action in a CD workflow which leaves deployable code in `GITHUB_WORKSPACE`.

The underlaying base-image of the docker-image is very small (Alpine (no cache)) which results in fast deployments.

---

## Inputs

- `switches`* - The first is for any initial/required rsync flags, eg: `-avzr --delete`

- `rsh` - Remote shell commands

- `path` - The source path. Defaults to GITHUB_WORKSPACE

- `remote_path`* - The deployment target path

- `remote_host`* - The remote host

- `remote_port` - The remote port. Defaults to 22

- `remote_user`* - The remote user

- `remote_key`* - The remote ssh key

- `remote_command` - The ssh remote command

``* = Required``

## Required secret

This action needs a `DEPLOY_KEY` secret variable. This should be the private key part of a ssh key pair. The public key part should be added to the authorized_keys file on the server that receives the deployment. This should be set in the Github secrets section and then referenced as the  `remote_key` input.

## Example usage

Simple:

```
name: DEPLOY
on:
  push:
    branches:
    - master

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: rsync deployments
      uses: iamapinan/rsync-deployments-with-ssh-command@1.0
      with:
        switches: -avzr --delete
        path: src/
        remote_path: /var/www/html/
        remote_host: example.com
        remote_user: debian
        remote_key: ${{ secrets.DEPLOY_KEY }}
```

Advanced:

```
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: rsync deployments
      uses: iamapinan/rsync-deployments-with-ssh-command@1.0
      with:
        switches: -avzr --delete --exclude="" --include="" --filter=""
        path: src/
        remote_path: /var/www/html/
        remote_host: example.com
        remote_port: 22
        remote_user: root
        remote_key: ${{ secrets.DEPLOY_KEY }}
        remote_command: docker exec ${{ secrets.CONTAINER_NAME }} php composer install && php artisan migrate:fresh --seed && php artisan cache:clear
```

For better security, I suggest you create additional secrets for remote_host, remote_port and remote_user inputs.

```
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: rsync deployments
      uses: burnett01/rsync-deployments@4.1
      with:
        switches: -avzr --delete
        path: src/
        remote_path: /var/www/html/
        remote_host: ${{ secrets.DEPLOY_HOST }}
        remote_port: ${{ secrets.DEPLOY_PORT }}
        remote_user: ${{ secrets.DEPLOY_USER }}
        remote_key: ${{ secrets.DEPLOY_KEY }}
```

