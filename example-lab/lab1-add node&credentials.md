# add node in jenkins
Di sini kita akan mencoba untuk menambahkan node untuk target kita gunakan untuk CI/CD, agar bisa dikonekan kita perlu mendownload package java

- install default-jre
```
sudo apt install default-jre -y
```

- create user jenkins
```
adduser jenkins --disabled-password
passwd jenkins
```

- add sudo in user jenkins
```
echo "jenkins ALL=(ALL) NOPASSWD:ALL" /etc/sudoer.d/jenkins
```

- add user jenkins in group docker
```
sudo usermod -aG docker jenkins
```

- add .kube
> disini dilakukan hanya pada node master saja
```
sudo cp -r /root/.kube /home/jenkins
sudo chown jenkins:jenkins /home/jenkins/.kube -R
```

lanjut ke web jenkins

# add credentials (node)
> credentials digunakan untuk login pada node yang menyimpan user dan password dan node.
manage jenkins > credentials > system > global credentials (unrestricted) > add credentials 
![image](https://github.com/galihtw04/jenkins/assets/96242740/0fe6bf7d-2e52-4f4b-9c11-0500d069c383)

# menambahkan node
akses jenkins di firefox/chrome, manage jenkins > nodes > new node > create > (Remote root directory) > (labels) > lunch method > Host > creadential > save
![image](https://github.com/galihtw04/jenkins/assets/96242740/f7cbf107-e2f2-47b9-aebf-4025830040b5)

![image](https://github.com/galihtw04/jenkins/assets/96242740/acbaafd5-7d2f-48d3-9ed3-8a2b72575938)

![image](https://github.com/galihtw04/jenkins/assets/96242740/9068bf7e-ac78-4c8d-a857-d56857e86395)
