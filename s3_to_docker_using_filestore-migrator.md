### 1) Create the Rocket.Chat container using the docker-compose.yml file:
[docker-compose.yml](https://github.com/hlatki01/rocket.chat/blob/main/docker-compose.yml)

Note: in this case the Rocket.Chat server container name is `rocketchat_rocketchat_1` and the mongodb container is `rocketchat_mongo_1`, you can get yours by using the command: `docker ps`

### 1.1) Get your container up and wait for the container creation
```
docker-compose up -d
```

### 1.2) Get the Rocket.Chat container down and keep the mongodb container running
```
docker stop rocketchat_rocketchat_1
```

### 2) If your files are multipart, join them by using cat:
```
sudo bash -c 'cat files.tar.gz.part{0..26} >> export.tar.gz'
```

if not, just untar it:
```
tar xf export.tar.gz
```

### 3) Copy the database to the mongo container:
```
docker cp ~/export/database/. rocketchat_mongo_1:/rc_dump
```

### 3.1) Access the mongodb container:
```
sudo docker exec -it rocketchat_mongo_1 bash
```

### 3.2) Access the mongodb and delete the fresh created database:
```
use rocketchat
db.dropDatabase()
```

### 3.3) Restore your database:
```
mongorestore --db rocketchat ./rc_dump/5d117b074ecabc0001f00510
```

### 3.4) Change the FileUpload_Storage_Type to FileSystem, set the FileUpload_FileSystemPath and Site_Url to the new workspace address

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

```js
db.rocketchat_settings.update(
    { _id: 'Site_Url' },
    { $set: { value: 'https://YOURNEWROCKETCHATSERVER.com' } }
);
```

### 4) Change upload folder permissions
```
sudo chmod ugo+wrx uploads
```

### 5) Install Go Library
```
sudo apt update && sudo apt install golang
```

### 6) Clone the migration tool (filestore-migrator)
```
git clone https://github.com/RocketChat/filestore-migrator migrator
```

### 6.1) Access the migrator folder
```
cd migrator
```

### 6.2) Build the tool:
```
go build ./cmd/filestore-migrator
```

### 6.3) Migrate the files, first will be Uploaded Files (Uploads):
```
sudo ./filestore-migrator -action=upload -databaseUrl=mongodb://localhost:27017/rocketchat -sourceType=s3 -detectDestination=true -store=Uploads -tempLocation ~/backup/files
```

### 6.4) Now, the Avatars:
```
sudo ./filestore-migrator -action=upload -databaseUrl=mongodb://localhost:27017/rocketchat -sourceType=s3 -detectDestination=true -store=Avatars -tempLocation ~/backup/files
```

### 6.5) Copy the files to the target destination:
```
for i in ~/export/files/avatars/*; do cp "$i" ~/YOURROCKETCHATFOLDER/uploads/; done
for i in ~/export/files/uploads/*; do cp "$i" ~/YOURROCKETCHATFOLDER/uploads/; done
```

Done!

if `Request Entity Too Large` error is being displayed while uploading large files, follow this guide: https://docs.rocket.chat/guides/administration/settings/file-upload/file-upload-faqs


