
pipeline {
    agent any
    stages {
        stage("Clone Code") {
            steps {
                echo "Cloning the Code"
                //For cloning the repo, we have to run the below command and we have check the branch also from browser git
                git url: "https://github.com/akshaykr3011/django-notes-app.git", branch: "main" 
            }
        }
        stage("Build") {
            steps {
                echo "Building the Image"
                //After cloning the repo, we have to build it, this is shell cmd, so groovy syntax for shell cmd is "sh"
                //You will get this error - "permission denied while trying to connect to the Docker daemon socket", 
                //because jenkins user is not present in docker group, so you have to go the terminal and
                //and run this command: cd /var/lib/jenkins/workspace/notes-app-cicd, now to give the permisson, run this 
                //command: sudu usermod -aG docker jenkins, and do reboot: sudo reboot, again try to build, error resolved
                sh "docker build -t notes-app ."
            }
        }
        stage("Push the Code into Docker Hub") {
            steps {
                echo "Pushing the image into Docker Hub"
                //We have to push the image to dockerHub, for that we need have access of DockerHub, go to browser and login
                //into dockerHub. to push the image you have login into dockeHub in EC2 server terminal by cmd: docker login
                //give the credentials there. If we will give the credentails here then it will be known to everyone
                //so that we will create environmet variables for credential, for that Go to Jenkins Dashboard, then Manage Jenkins,
                //then Security, then inside security Go to "Credentials", then click on "System" to add your credentials
                //Click on "Global credentials" then "Add Credentials", Fill the form, We will create Id for username and pass
                //and give their dockerHub username and pass, remember the ID name, the run below cmd
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]){
                    sh "docker tag notes-app ${env.dockerHubUser}/notes-app:latest"
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    //build now to check, now you see login succeded on console output
                    //CICD using Groovy syntax, Declarative Pipeline
                    //We will send the image to dockerHub with Declarative pipeline. 1st will tag the image
                    //for that we have to tag the image with username, we will add this into line number 33.
                    //After that push the image to dockerHub by running below command
                    sh "docker push ${env.dockerHubUser}/notes-app:latest"
                    
                    //You will see that image on dockerHub website
                }
            }
        }
        stage("Deploy") {
            steps {
                echo "Deploying the Code into Container"
                //We will deploy the Container. build makes Images, Images makes Container.
                //We will run the docker in deattached mode(-d), want to publish(-p), on port number 8000 of our system 
                //port number 8000 of container
                //sh "docker run -d -p 8000:8000 akshaykr3011/notes-app:latest"
                //sh "docker-compose"
                sh "docker-compose down && docker-compose up -d"
                //we have check is 8000 port is open or not on EC2, by default it is not open, so we have to set inside
                //security group, add rule 8000, Anywhere.
            }
        }//But it is not robust, if we will do again build, then it will give error "port is already allocated"
        //to resolve this we have docker compose because we need system which down the running build and up the upcoming build 
        //So, we have to install the docker compose in EC2 server by cmd: "sudo apt-get install docker-compose"
        //we will not run docker run command in line no. 52, Instead will will run "docker-compose" in line 53 and 
        //comment line 52 & Put image: "your image name" inside "docker-compose.yml" file
        //When we will run the cmd: docker-compose up -d, it will buid the container and run in backgroud on the port 8000
        //It will give error - port is already in use. So run the cmd: docker ps, and kill the privious one by cmd: docker kill containerID
        //again run: docker-compose up -d, it will up the container., it will start the docker(app)
        //run: docker-compose down, it will down the container, it will stop the docker(app)
        //Now if you run:docker ps, you will see there is no container
        //So the solution is : sh "docker-compose down && docker-compose up -d" in line number 54 and try "Build Now", 
        //It wll build succesfully
        //Now will do some change in docker-compose.yml on github, we will replace "build: ." as "image:akshaykr3011/notes-app"
        //we will do some changes in github repo and commit & merge the code. Then we have to click on "Build Now" on Jenkins to build the app, this is called Continuous delivery
        //We want that this script file would also come from github, for that we copy this content and will create filename "Jenkinsfile" and paste the content there

        //Now we will not use Pipeline script, we will use "Pipeline script from SCM".
        //We will fill the form details like git clone link and branch name.
        //It will do Declarative checkout: SCM.->Clone Code->Build->Push the code into Docker Hub->Deploy 
        //To automate the above things, there is a concept called "web hooks". for that git hits the jenkins whenever any changes happens it will automatically send
        //the request to jenkins.
        //for this we have to configure the github. Go to your repo->Go to settings->Click on webhooks->Add webhook->Authenticate it->
        //Payload url: put the jenkins url: like-http://54.86.39.149:8080/github-webhook/ ->send me everything ->Click on Add webhook.
        //Now wait for green tick, or refresh the github page
        //Now do any change in code and commit it, and check, It is building automatically or not on Jenkins.
    }
}
