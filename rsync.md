## ⏬ backup WP files from remote > local: 

``` bash
rsync -ah --progress --exclude="*.git" --delete  USER:/home/USER/htdocs/WWW.SITEWEB.INFO/ Desktop/WWW.SITEWEB.INFO/
```

## ⏫ Restore WP files from local > remote:

```bash
rsync -ah --progress  --exclude=".git" --delete  Desktop/WWW.SITEWEB.INFO/ USER:/home/USER/htdocs/WWW.SITEWEB.INFO/
```