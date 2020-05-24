dockerize angular app

steps
1. setup ubuntu linux - done

2. setup nodejs, angular cli - done
sudo apt-get install -y nodejs npm
node -v, npm -v
sudo apt-get install -g @angular/cli
ng --version

3. create an angular project - done
ng new angular-docker-app
cd angular-docker-app
ng serve

3. setup docker in local environment - done
sudo apt-get install docker.io
docker -v

4. Add docker to the project - done
add Dockerfile

5. create application docker image locally - done
sudo docker build -t angular-docker-app:dev .

6. run docker container locally - done
sudo docker run --publish 4201:4200 --detach --name ada angular-docker-app:dev
http://localhost:4201
sudo docker images
sudo docker container ls
sudo docker stop ada
sudo docker rm ada
OR sudo docker rm --force ada to stop and remove together
sudo docker run -v ${PWD}:/app -v /app/node_modules -p 4201:4200 --rm example:dev - creates a new container instance, from the image we just created, and runs it. -v ${PWD}:/app mounts the code into the container at “/app”. Since we want to use the container version of the “node_modules” folder, we configured another volume: -v /app/node_modules. You should now be able to remove the local “node_modules” flavor. -p 4201:4200 exposes port 4200 to other Docker containers on the same network (for inter-container communication) and port 4201 to the host. --rm removes the container and volumes after the container exits.


7. setup docker compose locally - done
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
give docker-compose executable - sudo chmod +x docker-compose
verify - docker-compose --version

8. run docker using docker compose - done
Build the image and fire up the container: sudo docker-compose up -d --build
http://localhost:4202/
sudo docker-compose stop

9. create production docker build - done
create Dockerfile-prod
First, we take advantage of the multistage build pattern to create a temporary image used for building the artifact – the production-ready Angular static files – that is then copied over to the production image. The temporary build image is discarded along with the original files, folders, and dependencies associated with the image. This produces a lean, production-ready image.
Next, the unit and e2e tests are run in the build process, so the build will fail if the tests do not succeed.

10. run production docker using docker compose - done
create prod docker compose yml
sudo docker-compose -f docker-compose-prod.yml up -d --build
sudo docker container ls -- see both dev and prod containers are running
sudo docker-compose -f docker-compose-prod.yml up -d --build --remove-orphans
the above removes existing orphan containers, like the dev one for example here, and starts a new container
verify - http://localhost:80/
stop and remove containers - sudo docker-compose down --remove-orphans

11. setup dockerhub - done
signup for dockerhub account
arupsarkardocker
create a repo

12. push prod docker image to dockerhub - done
updated docker-compose-prod.yml to push images to dockerhub account repo
sudo docker-compose -f docker-compose-prod.yml build
sudo docker login docker.io
sudo docker push arupsarkardocker/angular-docker-app
sudo docker logout - to remove text store credential

13. verify prod docker image - done
sudo docker pull arupsarkardocker/angular-docker-app
sudo docker run --publish 80:80 --detach --name ada-prod arupsarkardocker/angular-docker-app:latest - When you put EXPOSE 80 (or any port you want) in your Dockerfile that’s going to tell Docker that your container’s service can be connected to on port 80. our prod image was exposing on port 80 so that nginx can serve the dist content
verify - http://localhost:80/ - app should be running

14. make a small change to angular project html - done

15. build the project - done
sudo docker-compose -f docker-compose-prod.yml build

16. create new docker image - done
docker compose build itself create new docker image

17. update docker image with this new image in dockerhub - done
sudo docker login docker.io
sudo docker push arupsarkardocker/angular-docker-app
sudo docker logout - to remove text store credential

18. verify the new prod image from dockerhub - done
sudo docker pull arupsarkardocker/angular-docker-app
sudo docker run --publish 80:80 --detach --name ada-prod arupsarkardocker/angular-docker-app:latest


19. add project to Git - done
git init
git remote add origin <url to git repo>
add ssh key to talk to github
ssh-keygen -t rsa -b 4096 -C "swarupsarkar187@gmail.com"
save to default location - /home/swarup/.ssh/id_rsa - private key, public key as id_rsa.pub
passpharse - given
add new SSH key to the ssh-agent to manage your keys
Start the ssh-agent in the background - eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa - add the ssh key to the agent
add the ssh key (public key stored in id_rsa.pub) to the github account
git remote add origin git@github.com:gitswarup/angular-docker-app.git
git push -u origin master
first time connection can't be established as it's known part of known hosts. hence, provide the fingerprint to confirmaton. this will authenticate with github add add the repo to the known hosts and push the code


20. link github with dockerhub - done
linking github with dockerhub auto build runs on dockerhub infrastructure. The same can be achieved vice-versa. Using github packaging. Via github packaging, git build will create push the docker image to dockerhub. But that needs more config setup. But it might be more cheaper as github infra will be much cheaper than dockerhub infra. For now, both are same and free.
triggered build
build was successful
pull latest
restart container

21. push new changes to github - done
changed html label
push changes to github
auto build pickup failed - maybe dockerhub infra issue?
push change again
worked this time

22. verify docker image update in dockerhub
sudo docker pull arupsarkardocker/angular-docker-app
sudo docker restart ada-prod - restart didn't take the updated image
sudo docker stop ada-prod
sudo docker container prune
sudo docker run --publish 80:80 --detach --name ada-prod arupsarkardocker/angular-docker-app:latest
