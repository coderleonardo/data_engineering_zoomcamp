To use containers with Docker we must first create our Dockerfile, which is a step-by-step commands that the image use to instantiate our container.

For example
```
FROM python:3.9 # Set the base image to be used

RUN pip install pandas # Run the command to install pandas

WORKDIR /app # Set the working directory to app
COPY pipeline.py pipeline.py # Copy the pipeline.py to the ./app folder in the container

ENTRYPOINT [ "python", "pipeline.py" ] # Execute the following commands when the container is initialize
```

To build the image using Docker we must run the command ```sudo docker build -t <docker-image-name>:<tag> .```

Note that the "dot" says to build using all the files from the current directory.

After created the image, to initialize the container in a interactive mode ("it") we use the command ```sudo docker run -it <docker-image-name>:<tag>```