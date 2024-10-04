This directory contains utilities to help docker-odoo deployment.

<details>
<summary>

# Restore Snapshot

</summary>

To use restore snapshot, make sure the main directory (`../`) is has the same name of the archive file. For example, `snapshot-docker-odoo.tar.zst` the main directory name should be `docker-odoo` removing the `snapshot-` prefix and `.tar.zst` suffix.

Before you run the snapshot script, you need to prepare [snapshot utilities first](#snapshot-utilities).

</details>

<details>
  <summary>

  # Snapshot Utilities

  </summary>

  See the example file to create the snapshot utility (`./scripts/snapshot.sh.example`).

  1. Copy the example file. This will export the service name from your cloned repository dirname.
      ```bash
      export SERVICE_NAME=$(basename "$PWD")
      cp ./scripts/snapshot.sh.example ./scripts/snapshot-$SERVICE_NAME
      ```

  2. Edit your example file with your favorite text-editor (`vim` or `nano`, etc)
      ```bash
      vi ./scripts/snapshot-$SERVICE_NAME
      ```

  3. You need to find (`ctrl + f`) the `enter` word to see which value should be changed

  4. Save the file and change the permission.
      ```bash
      sudo chmod 755 ./scripts/snapshot-$SERVICE_NAME
      ```

  5. Create a soft-link to system-wide bin
      ```bash
      sudo ln -s $PWD/scripts/snapshot-$SERVICE_NAME /usr/local/sbin/snapshot-$SERVICE_NAME
      ```

  6. Add a new crontab to run your script
      ```bash
      sudo crontab -e
      ```

      Then add this cron:
      
      ```bash
      # run snapshot utility every 4 hour past 27 minutes in each day
      27 */4 * * * /usr/local/sbin/snapshot-$SERVICE_NAME
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
      sudo cat << EOF > ~/snapshot-$SERVICE_NAME
      /var/log/snapshot-$SERVICE_NAME.log {
          rotate 4
          su root syslog
          olddir /var/log/snapshot-$SERVICE_NAME.log-old
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

      sudo chown root: ~/snapshot-$SERVICE_NAME
      sudo chmod 644 ~/snapshot-$SERVICE_NAME
      sudo mv ~/snapshot-$SERVICE_NAME /etc/logrotate.d/snapshot-$SERVICE_NAME
      ```

  8. You can setup Google Cloud Storage for automatic rotate snapshot file or use `logrotate` on Ubuntu.

</details>