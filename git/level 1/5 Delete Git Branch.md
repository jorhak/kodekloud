```
The Nautilus developers are engaged in active development on one of the project repositories located at /usr/src/kodekloudrepos/media. During testing, several test branches were created, and now they require cleanup. Here are the requirements provided to the DevOps team:



On the Storage server in Stratos DC, delete a branch named xfusioncorp_media from the /usr/src/kodekloudrepos/media Git repository.
```

1. Ingresar al servidor
```
ssh natasha@ststor01
```

2. Eliminar branch
```
cd /usr/src/kodekloudrepos/media
git config --global --add safe.directory /usr/src/kodekloudrepos/media
git branch
sudo git checkout master
sudo git branch -d xfusioncorp_media
```
