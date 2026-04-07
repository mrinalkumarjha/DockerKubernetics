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
7.  docker build -t sample-web-app:1.0.0 .   : To create image locally based on custom docker file
8.  docker run --name sample-web-app-container -p 9000:80 sample-web-app:1.0.0   : Create container based on local image created 


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
          
        
          

      
     

     

