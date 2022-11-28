# Vuejs with Docker

This template should help get you started developing with Vue 3 in Vite and Docker.

## 1 - Create Vuejs Project

```sh
npm init vue@latest
```

Once we have finished installing the project, we will do the following:
1. `cd project-name` (in my case `cd vuejs-docker`)
2. `npm install`
3. `npm run dev`

When we run npm run dev we will receive an ip to see the project, if you can see the project all it's okay.

## 2 - Prepare Local Development
We need to prepare a Dockerfile and Docker-compose to run a container with vuejs3 and vite

1. Edit Package.json:
    ```sh
        "scripts": {
            "dev": "vite",
            "build": "run-p type-check build-only",
            "preview": "vite preview",
            "build-only": "vite build",
            "type-check": "vue-tsc --noEmit",
            "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts --fix --ignore-path .gitignore"
        },

2. Edit vite.config.ts adding `server { host: true,  port: 8080 }` you should see your vite.config.ts as the vite below this:
     ```sh
            export default defineConfig({
            server: {
                host: true,
                port: 8080
            },
            plugins: [vue()],
            resolve: {
                alias: {
                "@": fileURLToPath(new URL("./src", import.meta.url)),
                },
            },
            });

3. We need to create a Dockerfile (I named it Dev.Dockerfile) in the root of the project:
     ```sh
            FROM node:18.12.1-alpine

            WORKDIR /app
            COPY . ./
            RUN npm install

            EXPOSE 8080 8080

            CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
    ```
    Run the following command (you can change the name of the image (vuejs_docker:dev) for the name that you want)
    ```sh
    docker build -f Dev.Dockerfile -t vuejs_docker:dev . --network="host"
    ```

4. Create and edit docker-compose.yaml in the root of your project:
    container_name -> put the name that do you want
    image -> the name of the image that you have created at the point 3
     ```sh
       version: '3.7'
        services:
        app-vue-front:
            container_name: app-vue-front
            image: vuejs_docker:dev
            build:
            context: .
            volumes:
            - '.:/app'
            # - '/app/node_modules'
            ports:
            - '8080:8080'
    ```
    Run the following command to run the container
    ```sh
    docker-compose up -d
    ```

5. Check if the container it's working
    Navigate to `localhost:8080` and you will see the vuejs project.
    IMPORTANT -> If you are using a virtual machine you must use the ip (of the VM) like `0.0.0.0:8080` change 0's for your ip numbers

## 3 - Prepare Production Development
Once we have prepare local enviroment we need to create another Dockerfile with a Nginx image and edit docker-compose with the new image
1. Create a nginx.confg file in the root
    ```sh
        worker_processes 4;

        events { worker_connections 1024; }

        http {
            server {
                listen 80;
                root  /usr/share/nginx/html;
                include /etc/nginx/mime.types;

                location /appui {
                    try_files $uri /index.html;
                }
            }
        }

2. We need to create a Dockerfile (I named it Setup.Dockerfile) in the root of the project:
    EXPOSE 80 -> this port must be the same of the http - server - listen in `nginx.conf` file
     ```sh
        # develop stage
        FROM node:18.12.1-alpine as develop-stage
        WORKDIR /app

        # Copy the package.json and install dependencies
        COPY package*.json ./
        RUN npm install

        # Copy rest of the files
        COPY . .

        # Build the project
        RUN npm run build


        FROM nginx:alpine as production-build
        COPY ./nginx.conf /etc/nginx/nginx.conf

        ## Remove default nginx index page
        RUN rm -rf /usr/share/nginx/html/*

        # Copy from the stahg 1
        COPY --from=develop-stage /app/dist /usr/share/nginx/html

        EXPOSE 80
        # ENTRYPOINT ["nginx", "-g", "daemon off;"]
    ```
    Run the following command (you can change the name of the image (vuejs_docker:setup) for the name that you want)
    ```sh
    sudo docker build -f Setup.Dockerfile -t vuejs_docker:setup . --network="host"
    ```

3. Edit docker-compsoe.yaml with the new image
    container_name -> put the name that do you want
    image -> the name of the image that you have created at the point 2 - 3 and point 3 - 3
     ```sh
        version: '3.7'
        services:
        # Local
        app-vue-front:
            container_name: app-vue-front
            image: vuejs_docker:dev
            build:
            context: .
            volumes:
            - '.:/app'
            # - '/app/node_modules'
            ports:
            - '8080:8080'

        # Prod
        app-vue-setup:
            container_name: app-vue-setup
            image: vuejs_docker:setup
            build:
            context: .
            volumes:
            - '.:/app'
            # - '/app/node_modules'
            ports:
            - '80:80'
    ```
    Run the following command to run the containers
    ```sh
    docker-compose up -d
    ```
4. Check if the containers are working
    Navigate to `localhost:8080` and you will see the vuejs project (hot reload).
    Navigate to `localhost:80` and you will see the vuejs project in nginx server (no hot reload).
    IMPORTANT -> If you are using a virtual machine you must use the ip (of the VM) like `0.0.0.0:8080` change 0's for your ip numbers (same with `0.0.0.0:80`)
