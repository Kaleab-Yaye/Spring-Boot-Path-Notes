# the what and the why?

so when you want to use a data base like postgruss you have to choice to insall the data base as a whole in to your system or you cacn use a docker.
now thes two are super differn and there si a reasn why will choose to use a docker

1. it make sure every on in the project is using the same verion of data base meangin it will sovle "it worked on my device" tyep of bugs
2. it is easy to impplment you just creat docker-compose.yaml file and then you just run the `docker compose up -d` and just like that you have a data base
3. it doens take time to start overr you can cclean install it again in momentes.

now we will se how to actaully create it 

first in the root directory of the streamign api you creat docker-compose.yaml file

and it shold look like this
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=devuser
      - POSTGRES_PASSWORD=devsecret
      - POSTGRES_DB=streaming_api_db
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

explaining what each section does and is for
*  the verison is the versino of the docker that we are goig to use
*  the name of our service will be Postgres, we could name it any thing.
* image: this is the criticall part this is the actuall iamge of postgress taht is goign to be installe  in our container. now this contaner shoudl ahve some kind of os to run on so the  posgres:15 image runs on
  debian by defualt but we are telli it to make os that runs our container to be run on alpinlinux one that is know to be ligh waitgh ( forgiving on systems) and also secuity focused instead of the debian verion
* enviroment; this is the profile that our data base ( posgres) will have when it first starts up
* the ports are maping; so our API will talk to the os with the port 5432 and allso our os will talk to the container on its 5432 port
* volues; with out this our data wont be persisted so we creeat a volume wihht the name pgdata and then insidede the conteinr we realted this volume with the actuall file directory that is var/......../data. and
* the data that need to be persisited will stat there

# instiating
install docker desktop first
now after all tha what you need to do is jsut type the follwing commadn to make sure the doecker staters, at firts it will downlaod files that are needed including the os contained container
```
docker compose up -d
```
the -d flag indicated that it shoudl run on the back graoudn freeing you terminal 
