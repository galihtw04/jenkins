# Skenario 2
> Jika pada sebelumnya kita deploy masih manual menggunakan sh, sekarang kita akan memanfaatkan plugin yang ada pada jenkins

- Setup Plugin

Install plugin Docker Pipline
> note sudah saya install
![image](https://github.com/galihtw04/jenkins/assets/96242740/b7dd1cfb-3233-41e5-a4df-4ab2ae36c0ca)

install plugin external file

![image](https://github.com/galihtw04/jenkins/assets/96242740/d58ae538-e1b1-439c-8542-343d66405c32)
> scroll kebawah

![image](https://github.com/galihtw04/jenkins/assets/96242740/4996b58d-9f62-46f8-bbe6-22e73527da40)

![image](https://github.com/galihtw04/jenkins/assets/96242740/2558430b-6329-4ff3-aa9f-bf37fe5b9ab6)
> note karena sudah diiinstall jadi notifnya already.

selanjutnya kita siapkan secret untuk pull image dari repo local yang akan di simpan pada manifest kita nanti

```
kubectl get -n apps-caculator secret admin-repo -o yaml
```
![image](https://github.com/galihtw04/jenkins/assets/96242740/e45220d1-31c5-444c-ba90-48a3d02c5220)
> Nanti akan seperti pada manifest di repo [ini](https://raw.githubusercontent.com/galihtw04/test3-build/master/ns_secret.yaml), user/passowrd admin/P@ssword sesuai pada tutorial sebelumnya.

create ci/cd pipline

step Dashboard>new item>(name)>select pipline>ok
![image](https://github.com/galihtw04/jenkins/assets/96242740/a5a80100-846e-42fa-be8b-a151eeb30c1c)

scroll paling bawah dan isikan syntax dibawah ini,

```
pipeline {
  environment {
    dockerimagename = "registry-nexus.cloud/apps-lab3"
	containerName = "test"
  }

   agent {
       label 'master02'
    }

  stages {

    stage('Checkout Source') {
      steps {
        git 'https://github.com/galihtw04/uji-coba1.git'
      }
    }

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }

    stage('Pushing Image') {
      environment {
               registryCredential = 'registry-docker'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry-nexus.cloud', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }
	
	stage('Create Docker Container') {
	  steps {
		script {
          withCredentials([usernamePassword(credentialsId: 'registry-docker', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
            sh 'sudo docker login registry-nexus.cloud -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
		    sh 'docker run -d -p 3000:3000 --name testing ${dockerimagename}'
			sh 'docker rm -f testing'
            }
			}
        }
    }

	
    stage('Deploying App to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(configs: "ns_secret.yaml", kubeconfigId: "kubeconfig-k8s")
		  kubernetesDeploy(configs: "caculator.yaml", kubeconfigId: "kubeconfig-k8s")
        }
      }
    }
  }
}
```

save dan build code pipline kita
![image](https://github.com/galihtw04/jenkins/assets/96242740/e18a1084-8ec3-4e28-82f8-7d2c03306d28) 
