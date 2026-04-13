# DockerKubernetics

# Image
It is a blue print of any application. ex class in c#.

# Container
It is running instance of image, we can create as many container from one image. Ex: we create object from class in c#.

# Commands

1. Docker images : list all images
2. Docker pull image-name : it will pull image to local system. Ex : docker pull hello-world  will pull hello-world image from server to local.
3. Docker ps -a : it will list all container. -a will list all container. if we dont add -a it will list only active container which is not exited.
4. Docker run image-name : it will create new instance of container based on image name provided.
5. docker rmi 452a : This will delete image with matching id 452a
6. Docker rm 324s : This will remove container with matching id 324s
7.  docker build -t sample-web-app:1.0.0 .   : To create image locally based on custom docker file, it create Image + temporary containers (internal use) not running container like docker run do.
8.  docker run --name sample-web-app-container -p 9000:80 sample-web-app:1.0.0   : Create container based on local image created 

# Docker File instructions
  ### FROM: The FROM command in a Dockerfile is the starting point of your image. It defines the base image on top of which your application is built. 
          EX: FROM ubuntu:22.04
          This means: It Use the Ubuntu OS image, Version/tag = 22.04

  ### COPY: The COPY command in a Dockerfile is used to copy files or directories from your local system into the Docker image. syntax : COPY <source> <destination>
          EX: COPY . /app
          it Copies everything from current directory Into /app inside the container
          COPY package.json /app/
          Only copies one file.

  ### WORKDIR: The WORKDIR command in a Dockerfile sets the working directory inside the container. All subsequent commands (RUN, COPY, CMD, etc.) will execute relative to this directory.
          EX: WORKDIR /app
          It Sets /app as the current directory, If it doesn’t exist → Docker creates it automatically

  ### ARG: ARG (Build-time variable), Used only during image build.
          EX: ARG VERSION=1.0
              RUN echo $VERSION

              It is Available only while building the image. NOT available in running container.
              Can be overridden at build time: docker build --build-arg VERSION=2.0 .

  ### ENV: ENV (Runtime environment variable). Used for both build time AND runtime.
          EX: ENV APP_ENV=production
              It is Available inside container at runtime. Can be accessed by application. It Persists in final image.

  ### EXPOSE : The EXPOSE instruction in a Dockerfile is used to declare which ports your container listens on at runtime. 
                EXPOSE does NOT actually publish the port.
                It only Documents the port, Helps tools understand container networking
                
          EX: EXPOSE 80 : means Container intends to listen on port 80


  ### RUN : The RUN instruction in a Dockerfile is used to execute commands during the image build process. It is primarily used to install packages, set up dependencies, and prepare the environment.
            Each RUN creates a new layer:
            RUN executes only at build time
            NOT executed when container starts
            For runtime commands → use CMD or ENTRYPOINT
            
          EX: FROM mcr.microsoft.com/dotnet/sdk:8.0
              WORKDIR /src
              COPY . .
              
              RUN dotnet restore
              RUN dotnet build

  ### ENTRYPOINT: The ENTRYPOINT instruction in a Dockerfile defines the main command that will always run when a container starts. Think of it as the fixed executable of your container.
          EX: ENTRYPOINT ["echo", "Hello World"]
          When container runs: output will be Hello World

          ENTRYPOINT ["dotnet", "MyApp.dll"]
          This means Container will always run your application, No need to specify command every time.


# How to create image for simple dotnet api project ?

  ### sample docker file based on .net 8
          FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
          WORKDIR /src
          # following instruction csproj to src folder as it is the current working directory, then restore and publish the project
          COPY ./HelloDockerApi.csproj ./
          RUN dotnet restore "HelloDockerApi.csproj"
          # After restore we are copying all files to the src folder and publish the project to the /app folder in the container
          COPY . .
          # publish the project to the /app folder in the container
          RUN dotnet publish "HelloDockerApi.csproj" -c Release -o /app/
          
          # After the build stage is complete, we are using the final image to run the application. We are copying the published files from the build stage to the final image and setting the entry point to run the                     application.
          # Starts a completely new image
          FROM mcr.microsoft.com/dotnet/sdk:8.0 AS final
          WORKDIR /app
          # Copy all file from app folder of build image in to current work directory which is app. but inside final image.
          COPY --from=build /app .
          ENTRYPOINT ["dotnet", "HelloDockerApi.dll"]

          Now run following command to create image from above docker file.
          docker build -t mtestapp:1.0.0 .
          Now run following command to create running container from above image.
          docker run -p 8000:5000 -e DOTNET_URLS=http://+:5000 --name mtestapp-container mtestapp:1.0.0

# Internal Execution Flow Of docker file instruction.

          For each instruction (like WORKDIR, RUN, COPY), Docker does:
          
          Takes previous image layer
          Creates a temporary container from it
          Executes the instruction inside that container
          Commits the result as a new image layer
          Deletes the temporary container

          So specifically for WORKDIR /src

          Internally:
          
          Docker creates a temporary container from previous layer
          Inside that container:
          Creates /src directory (if not exists)
          Sets it as working directory metadata
          Commits this as a new image layer
          Removes the temporary container

          Visualizing This
          Step 1: FROM image
                  ↓
             [Image Layer 1]
          
          Step 2: WORKDIR /src
                  ↓
             (Temp Container created)
                  ↓
             mkdir /src
             set WORKDIR
                  ↓
             (Commit layer)
                  ↓
             [Image Layer 2]
                  ↓
             (Temp container removed)


# How to host simple html page in docker container ?

  ### Step1: create a hello world html page named index.html.

     
      <!DOCTYPE html>
      <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Hello World</title>
        </head>
        <body>
            <h1>Hello World from docker by mrinal.</h1>
        </body>
      </html>

  ### Step2: Create a DockerSample Folder and inside DockerSample create another folder like SampleWebApp. and copy index.html inside SampleWebApp.

  ### Step3: Create dockerfile in DockerSample folder with following statement

      
        FROM nginx
        COPY ./SampleWebApp/ /usr/share/nginx/html

        explanation of above syntax : 
        
         FROM nginx : this is web server needed for serving html
          COPY ./SampleWebApp/ /usr/share/nginx/html :  here we are copying all file from SampleWebApp to usr/share/nginx/html folder of nginx server. usr/share/nginx/html this folder path is mentioned in official docker             hub image document of nginx.  

  ### Step4: Run following command to create image based on above created docker file.

          docker build -t sample-web-app:1.0.0 .

          . means dockerfile is available in same directory from where above command is executing . otherwise we need to give path of docker file.

  ### Step5 : Run following command to create container from above image.

          docker run --name sample-web-app-container -p 9000:80 sample-web-app:1.0.0

          9000:80  : here 9000 is port of docker host env and 80 is port of container. we are doing port mapping . 
          Once above command ran successfully we can access out html using localhost:9000 .

        
          
        
          

      
     

     

