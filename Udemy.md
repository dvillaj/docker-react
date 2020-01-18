# Creating a Production Grade Workflow

## Project Generation

	$ npx create-react-app frontend
	Success! Created frontend at /home/vagrant/frontend
	Inside that directory, you can run several commands:

	  npm start
	    Starts the development server.

	  npm run build
	    Bundles the app into static files for production.

	  npm test
	    Starts the test runner.

	  npm run eject
	    Removes this tool and copies build dependencies, configuration files
	    and scripts into the app directory. If you do this, you can’t go back!

	We suggest that you begin by typing:

	  cd frontend
	  npm start

	Happy hacking!

## Necessary Commands

	npm start
	npm test
	npm run build


## Starting the Container

	$ docker build -f Dockerfile.dev .
	Sending build context to Docker daemon  123.9kB
	Step 1/6 : FROM node:alpine
	 ---> 48c15decdff5
	Step 2/6 : WORKDIR '/app'
	 ---> Using cache
	 ---> 3bc74e8cd832
	Step 3/6 : COPY package.json .
	 ---> Using cache
	 ---> 41a59ee9d02a
	Step 4/6 : RUN npm install
	 ---> Using cache
	 ---> b5ff44a57f82
	Step 5/6 : COPY . .
	 ---> 10cc9c0afda2
	Step 6/6 : CMD ["npm", "run", "start"]
	 ---> Running in a0ca0f0ee06d
	Removing intermediate container a0ca0f0ee06d
	 ---> 61a3c9484c05
	Successfully built 61a3c9484c05


## Docker volumes

|Param|Desc|
|-|-|
|`-v $(pwd):/app`|Mount a shared folder between the actual path and /app|
|`-v /app/node_modules`|Not mount this directory|


	$ docker run -p 3000:3000 -v $(pwd):/app 61a3c9484c05

	> frontend@0.1.0 start /app
	> react-scripts start

	sh: react-scripts: not found
	npm ERR! code ELIFECYCLE
	npm ERR! syscall spawn
	npm ERR! file sh
	npm ERR! errno ENOENT
	npm ERR! frontend@0.1.0 start: `react-scripts start`
	npm ERR! spawn ENOENT
	npm ERR! 
	npm ERR! Failed at the frontend@0.1.0 start script.
	npm ERR! This is probably not a problem with npm. There is likely additional logging output above.
	npm WARN Local package.json exists, but node_modules missing, did you mean to install?

	npm ERR! A complete log of this run can be found in:
	npm ERR!     /root/.npm/_logs/2020-01-18T05_01_50_319Z-debug.log


Now works!

	$ docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app 61a3c9484c05

	> frontend@0.1.0 start /app
	> react-scripts start

	Starting the development server...

	Compiled successfully!

	The app is running at:

	  http://localhost:3000/

	Note that the development build is not optimized.
	To create a production build, use npm run build.

## Executing Tests

	docker run -it b1041ae1c01f npm run test

Other Option:

	docker-compose up
	docker exec -it frontend_web_1 npm run test

## Docker Compose for Running Tests

Create a specific container for testing: `tests`
Re-runs tests if any file changes

The `--build` parameter if why we changed the docker-compose.yml file ...

	$ docker-compose up --build
	WARNING: Found orphan containers (frontend_test_1) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
	Building web
	Step 1/6 : FROM node:alpine
	---> 48c15decdff5
	Step 2/6 : WORKDIR '/app'
	---> Using cache
	---> 3bc74e8cd832
	Step 3/6 : COPY package.json .
	---> Using cache
	---> 06a6a7acf896
	Step 4/6 : RUN npm install
	---> Using cache
	---> 6866233c1373
	Step 5/6 : COPY . .
	---> Using cache
	---> ea243b8d22ee
	Step 6/6 : CMD ["npm", "run", "start"]
	---> Using cache
	---> dc40e1de64a4
	Successfully built dc40e1de64a4
	Successfully tagged frontend_web:latest
	Building tests
	Step 1/6 : FROM node:alpine
	---> 48c15decdff5
	Step 2/6 : WORKDIR '/app'
	---> Using cache
	---> 3bc74e8cd832
	Step 3/6 : COPY package.json .
	---> Using cache
	---> 06a6a7acf896
	Step 4/6 : RUN npm install
	---> Using cache
	---> 6866233c1373
	Step 5/6 : COPY . .
	---> Using cache
	---> ea243b8d22ee
	Step 6/6 : CMD ["npm", "run", "start"]
	---> Using cache
	---> dc40e1de64a4
	Successfully built dc40e1de64a4
	Successfully tagged frontend_tests:latest
	Starting frontend_web_1   ... done
	Starting frontend_tests_1 ... done
	Attaching to frontend_tests_1, frontend_web_1
	tests_1  | 
	tests_1  | > frontend@0.1.0 test /app
	tests_1  | > react-scripts test --env=jsdom
	tests_1  | 
	web_1    | 
	web_1    | > frontend@0.1.0 start /app
	web_1    | > react-scripts start
	web_1    | 
	web_1    | Starting the development server...
	web_1    | 

We can not use the keyboard to enter commands to execute tests, so we need to use `attach` command in docker.
`attach` links stdin and stdout with the container

	$ docker ps
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
	6320552d3b31        frontend_web        "docker-entrypoint.s…"   11 seconds ago      Up 10 seconds       0.0.0.0:3000->3000/tcp   frontend_web_1
	0f3d769f8db5        frontend_tests      "docker-entrypoint.s…"   4 minutes ago       Up 10 seconds                                frontend_tests_1


Not works!

	$ docker attach frontend_tests_1


In this case it is not working because npm is running two processes for running tests

$ docker exec -it frontend_tests_1 sh
/app # ps
PID   USER     TIME  COMMAND
    1 root      0:00 npm
   19 root      0:00 node /app/node_modules/.bin/react-scripts test --env=jsdom
   26 root      0:02 node /app/node_modules/react-scripts/scripts/test.js --env=jsdom
  123 root      0:00 sh
  130 root      0:00 ps

With this option we don't have the ability to interact with the tests

## Multistep Docker Builds

Two steps. In the first step we are going to populate the `build` directory with `npm`, in the second step we use nginx to serve this content

	$ docker build  .
	Sending build context to Docker daemon  854.5kB
	Step 1/8 : FROM node:alpine as builder
	---> 48c15decdff5
	Step 2/8 : WORKDIR '/app'
	---> Using cache
	---> 395a87c1ed1f
	Step 3/8 : COPY package.json .
	---> Using cache
	---> 8dfcecdd4635
	Step 4/8 : RUN npm install
	---> Using cache
	---> 4e07e805f292
	Step 5/8 : COPY . .
	---> Using cache
	---> 12f61b40ce20
	Step 6/8 : RUN npm run build
	---> Running in 595c7c10e8c6

	> frontend@0.1.0 build /app
	> react-scripts build

	Creating an optimized production build...
	Compiled successfully.

	File sizes after gzip:

	45.16 KB (-1 B)  build/static/js/main.7bc37bc7.js
	264 B            build/static/css/main.40b4a62f.css

	The project was built assuming it is hosted at the server root.
	To override this, specify the homepage in your package.json.
	For example, add this to build it for GitHub Pages:

	"homepage": "http://myname.github.io/myapp",

	The build folder is ready to be deployed.
	You may serve it with a static server:

	npm install -g serve
	serve -s build

	Removing intermediate container 595c7c10e8c6
	---> d36887a3d20c
	Step 7/8 : FROM nginx
	latest: Pulling from library/nginx
	000eee12ec04: Already exists 
	eb22865337de: Pull complete 
	bee5d581ef8b: Pull complete 
	Digest: sha256:50cf965a6e08ec5784009d0fccb380fc479826b6e0e65684d9879170a9df8566
	Status: Downloaded newer image for nginx:latest
	---> 231d40e811cd
	Step 8/8 : COPY --from=builder /app/build /usr/share/nginx/html
	---> ae35c89497fe
	Successfully built ae35c89497fe
	vagrant@vagrant:~/docker/frontend$ docker run ae35c89497fe

## Running Nginx

	vagrant@vagrant:~/docker/frontend$ docker run -p 8080:80 ae35c89497fe
	10.0.2.2 - - [21/Dec/2019:12:13:56 +0000] "GET / HTTP/1.1" 200 378 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:71.0) Gecko/20100101 Firefox/71.0" "-"
	10.0.2.2 - - [21/Dec/2019:12:13:56 +0000] "GET /static/css/main.40b4a62f.css HTTP/1.1" 200 358 "http://localhost:8080/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:71.0) Gecko/20100101 Firefox/71.0" "-"
	10.0.2.2 - - [21/Dec/2019:12:13:56 +0000] "GET /static/js/main.7bc37bc7.js HTTP/1.1" 200 144953 "http://localhost:8080/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:71.0) Gecko/20100101 Firefox/71.0" "-"
	10.0.2.2 - - [21/Dec/2019:12:13:56 +0000] "GET /static/media/logo.5d5d9eef.svg HTTP/1.1" 200 2671 "http://localhost:8080/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:71.0) Gecko/20100101 Firefox/71.0" "-"


# Continuous Integration and Deployment with AWS

## Setup GitHub

	cd frontend/
	git init
	git add .
	it commit -m "Initial commit"
	git remote add origin git@github.com:dvillaj/docker-react.git

## Automatic Build Creation

	git add .
	git commit -m "added travis file"
	git push origin master
	history