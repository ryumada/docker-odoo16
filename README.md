# Odoo Docker Image
A Dockerfile to create a custom Odoo docker image.

| Specification | Version |
|----|----|
|Python|`'3.7'` (recommended) or `'3.10'`|
|Odoo version|`'16'`|
|PostgreSQL|`'14'`|

| Python `'3.10'` has slower build time and not compatible with ks_dashboard.

| ⚠️ You need to read this README.md file thoroughly. ⚠️

There are some points you should know:

- First, you need to execute `sudo ./_RUNMEFIRST.sh` script to check if all of your files and directories are ready to build image.
  ```bash
  sudo ./_RUNMEFIRST.sh
  ```

- Please follow the instruction after you run that script above, before continue the porcess.

- You should add your Odoo base, whether it is Odoo Community, Odoo Enterprise, or your custom Odoo base, to the `odoo-base` directory (⚠️ Only add one directory to `odoo-base` as this will be read automatically by the `entrypoint.sh` script, for the name of the directory is no need to be `odoo` ⚠️).

- Add your custom Odoo Modules (Odoo Addons) to `git` directory and add the path to addons_path in `./conf/odoo.conf`. Don't add unused custom module directory to this directory as it will be added to your docker image and increased the image size.

- Odoo `datadir` is placed on `/var/lib/odoo` and Odoo `log` is placed on `/var/log/odoo`. These directories will be used by Odoo for static data storage and logging. It will be called in docker-compose. (⚠️ This directories are automatically created on your host machine after you run `sudo ./_RUNMEFIRST.sh` ⚠️)

- Build your docker image with this command below:

  ```bash
  docker compose build
  ```

  After the build completed, you can copy the image name and enter it in your `docker-compose.yml`.

- Run your odoo deployment with docker compose.

  ```bash
  docker compose up
  ```

  You can also run detach the docker compose stdout with this command:

  ```bash
  docker compose up -d
  ```
  

- If your Odoo module needs libreoffice you can install it using this command:

  ```bash
  docker exec -itu root $CONTAINER_ID apt --no-install-recommends -y install libreoffice
  ```

  or you can uncomment this `RUN` syntax on dockerfile to include the installation of libreoffice on your docker image.

  ```dockerfile
  ...
  # install libreoffice only be needed if there is a module need to use libreoffice featrue
  # RUN apt --no-install-recommends -y install libreoffice
  ...
  ```

- If you want to commit changes of your config, make sure to change the ownership to your user first before create a new commit.
  ```bash
  sudo chown -R $USER: ./
  ```

# Maintenance
The image build using the dockerfile in this repository installed some utility scripts.

## Check the version of Odoo base
You can check the Odoo version and its git hash by running this command:

```bash
docker compose exec $SERVICE_NAME getinfo-odoo_base
```

<details>
  <summary>You can get <code>$SERVICE_NAME</code> by looking at your <code>docker-compose.yml</code> file. </summary>

  ```dockerfile
  ...
  services:
    # Enter the correct the service name, you can use company name (example: sudoerp)
    enter_the_correct_service_name: <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
      # Enter the correct image name below (format: username/repo:tag, example: odoo:16.0)
      image: username/repo:tag
      build:
        context: .
        dockerfile: dockerfile
      # Because we use host ne
  ...
  ```
</details>

## Check the git repository used by Docker Image
You can check the git repository information by running this command:

```bash
docker compose exec $SERVICE_NAME getinfo-odoo_git_addons
```

<details>
  <summary>You can get <code>$SERVICE_NAME</code> by looking at your <code>docker-compose.yml</code> file. </summary>

  ```dockerfile
  ...
  services:
    # Enter the correct the service name, you can use company name (example: sudoerp)
    enter_the_correct_service_name: <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
      # Enter the correct image name below (format: username/repo:tag, example: odoo:16.0)
      image: username/repo:tag
      build:
        context: .
        dockerfile: dockerfile
      # Because we use host ne
  ...
  ```
</details>

## Data Backup Utilities
See the example file to create the backup utility (`./scripts/backup_utility.sh.example`).

1. Copy the example file. This will export the service name from your cloned repository dirname.
    ```bash
    export SERVICE_NAME=$(basename "$PWD")
    cp ./scripts/backup_utility.sh.example ./scripts/backup-$SERVICE_NAME
    ```

2. Edit your example file with your favorite text-editor (`vim` or `nano`, etc)
    ```bash
    vi ./scripts/backup-$SERVICE_NAME
    ```

3. You need to find (`ctrl + f`) the `enter` word to see which value should be changed

4. Save the file and change the permission.
    ```bash
    sudo chmod 755 ./scripts/backup-$SERVICE_NAME
    ```

5. Create a soft-link to system-wide bin
    ```bash
    sudo ln -s $PWD/scripts/backup-$SERVICE_NAME /usr/local/sbin/backup-$SERVICE_NAME
    ```

6. Add a new crontab to run your script
    ```bash
    sudo crontab -e
    ```

    Then add this cron:
    
    ```bash
    # run backup utility every 4 hour past 27 minutes in each day
    27 */4 * * * /usr/local/sbin/backup-$SERVICE_NAME
    ```
    
    > ⚠️ Replace `$SERVICE_NAME` to the value of your root repository name (`basename "$PWD"`).

    Make sure that the crontab is added:

    ```bash
    sudo crontab -l
    ```

7. Rotate the logfile.
    > ⚠️ Make sure you are in the root repository.
    ```bash
    export SERVICE_NAME=$(basename "$PWD")
    sudo cat << EOF > ~/backup-$SERVICE_NAME
    /var/log/backup-$SERVICE_NAME.log {
        rotate 4
        su root syslog
        olddir /var/log/backup-$SERVICE_NAME.log-old
        weekly
        missingok
        #notifempty
        nocreate
        createolddir 775 odoo root
        renamecopy
        compress
        compresscmd /usr/bin/xz
        compressoptions -ze -T 0
        delaycompress
        dateext
        dateformat -%Y%m%d-%H%M%S
    }

    EOF

    sudo chown root: ~/backup-$SERVICE_NAME
    sudo chmod 644 ~/backup-$SERVICE_NAME
    sudo mv ~/backup-$SERVICE_NAME /etc/logrotate.d/backup-$SERVICE_NAME
    ```

8. You can setup Google Cloud Storage for automatic rotate backup file or use `logrotate` on Ubuntu.


---

Copyright © 2024 ryumada. All Rights Reserved.

Licensed under the [MIT](LICENSE) license.
