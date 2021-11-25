This installation guide was tested in the following environment:
Rocket.Chat 4.0.5
OS: Ubuntu LTS 20.0.1
Mongodb 4.12.2

### 1. Send the export file to your server, in this case cURL was used:

```
curl -o export.tar.gz https://your-domain.com/export.tar.gz
```

### 2. Get down the rocketchat container only:

```
docker-compose rc_rocketchat_1 down
```

### 3. Extract the export archive

```
tar xf export.tar.gz
```

### 4. Copy files to a mapped volume

```
sudo cp files/avatars/* ~/rc/uploads
sudo cp files/uploads/* ~/rc/uploads
```

### 5. Change upload folder permissions

```
sudo chmod ugo+wrx uploads
```

### 6. Copy the dumped files from the host machine to the container

```
docker cp database/. rc_mongo_1:/rc_dump
```

### 7. Access the mongo container

```
sudo docker exec -it rc_mongo_1 bash
```

### 8. Restore the database - change "5f3a5244fd8309adas30001b165db" for your database string

```
mongorestore --db rocketchat ./rc_dump/5f3a5244fd8309adas30001b165db --drop
```

### 9. Access the mongo console and select the right database

```
mongo
use rocketchat
```

### 10. Change the FileUpload_Storage_Type to FileSystem and set the FileUpload_FileSystemPath

```js
db.rocketchat_settings.update(
    { _id: 'FileUpload_Storage_Type' },
    { $set: { value: 'FileSystem' } }
);
```

```js
db.rocketchat_settings.update(
    { _id: 'FileUpload_FileSystemPath' },
    { $set: { value: '/app/uploads' } }
);
```

### 11. Change the old url to the new URL

```
const oldUrl = 'https://your-rc-server.rocket.chat/';
const newUrl = 'https://your-onprem-rc-server.com/';

```

### 12. Replace the old url with the new one

```js
for (let type of ['uploads', 'avatars']) {
    db[`rocketchat_${type}`].find().forEach((cursor) => {
        delete cursor.AmazonS3;
        cursor.store = 'FileSystem:Uploads';
        cursor.url = cursor.url.replace(RegExp(`^${oldUrl}`), newUrl);
        cursor.url = cursor.url.replace(RegExp(`^${newUrl}ufs/AmazonS3:`), `${newUrl}ufs/FileSystem:`);
        cursor.path = cursor.path.replace(
            RegExp('^/ufs/AmazonS3:'),
            '/ufs/FileSystem:'
        );
        db[`rocketchat_${type}`].save(cursor);
    });
}
```

### 13. Exit from mongo console typing twice 

```
exit
```

### 14. Get the rocketchat container up again:

```
docker-compose up -d
```
