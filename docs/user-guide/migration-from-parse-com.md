## Configure MongoDB
#### Step 1: Enable SSL Connection
Parse provides a migration tool for existing applications. In order to make use of it, we need to open MongoDB to external connections and secure it with a copy of the TLS/SSL certificate from Let's Encrypt. Start by combining `fullchain1.pem` and `privkey1.pem` into a new file in `/etc/ssl` (remember to change !!text-danger|domain_name!! with yours):

    sudo cat /etc/letsencrypt/archive/domain_name/{fullchain1.pem,privkey1.pem} | sudo tee /etc/ssl/mongo.pem
    
!!! note-warning ""
    You will have to repeat the above command after renewing your Let's Encrypt certificate. If you configure auto-renewal of the Let's Encrypt certificate, remember to include this operation.

Make sure `mongo.pem` is owned by the **mongodb** user, and readable only by its owner:

    sudo chown mongodb:mongodb /etc/ssl/mongo.pem
    sudo chmod 600 /etc/ssl/mongo.pem

Now, open `/etc/mongod.conf`:

    sudo nano /etc/mongod.conf
    
Here, we'll make several important changes.

First, look for the `bindIp` line in the `net`: section, and tell MongoDB to listen on all addresses by changing `127.0.0.1` to `0.0.0.0`. Below this, add SSL configuration to the same section:
```yaml
# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0
  ssl:
    mode: requireSSL
    PEMKeyFile: /etc/ssl/mongo.pem
```

!!! note-warning ""
    You can skip above commands and configs if you skip Let's Encrypt installation. You still need to edit `/etc/mongod.conf` to continue with the migration.

Finally, the migration tool requires us to set the `failIndexKeyTooLong` parameter to `false`:
```yaml
setParameter:
  failIndexKeyTooLong: false
```
!!! note-warning ""
    **Note:** Whitespace is significant in MongoDB configuration files, which are based on YAML. When copying configuration values, make sure that you preserve indentation.

Exit and save the file.

Restart the mongod service:

    sudo service mongod restart

## Initiate Migration Process
Login to Parse dashboard at dashboard.parse.com then open the settings for your app. Under **General**, locate the **Migrate** button and click it:
![Migrate in General](/assets/images/small-000.png)

You will be prompted for a MongoDB connection string. Use the following format:

	mongodb://parse:password@your_domain_name:27017/database_name


If you have `Let's Encrypt` SSL, add `?ssl=true` after the URL.

For example, if you are using the domain `example.com`, with the user `parse`, the password `foo`, and a database called `wallx`, your connection string would look like this:

	mongodb://parse:foo@example.com:27017/wallx?ssl=true

The configuration should look like below:

![Database connection string](/assets/images/small-001.png)

Click **Begin the migration**. You should see progress dialogs for copying a snapshot of your Parse hosted database to your server, and then for syncing new data since the snapshot was taken. The duration of this process will depend on the amount of data to be transferred, and may be substantial.
	
![](/assets/images/small-002.png)
![](/assets/images/small-003.png)

**Don't click Finalize yet.** We need to verify the migration first.

## Verify Data Migration
Return to your `mongo` shell, and examine your local database. Begin by accessing **database_name** and examining the collections it contains:

	use database_name
	show collections

Sample output

	Wallpaper
	Likes
	_Index
	_SCHEMA
	_Session
	_User
	_dummy
	system.indexes

You can examine the contents of a specific collection with the `.find() `method:

	db.Wallpaper.find()

After verifying the data in your new database, come back to Parse dashboard in your browser and the **Migration** tab:

![](/assets/images/small-004.png)

Click the **Finalize** button:

![](/assets/images/small-005.png)

Your app should now be migrated.