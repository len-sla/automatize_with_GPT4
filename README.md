## Project Name Using GPT4 to automise boring tasks
Given task to GPT4: with usage of  docker : in working directory check which of mp4 files are rotated and  if necessary  with the help of script using docker rotate them . Save result in subdirectory adding to to source mp4 name 'rotated" explain script and code step ny step

---



Sure, I can guide you through the process of completing this task using Docker. Here are the steps:
First, we need to create a Docker image that has all the necessary tools and dependencies for our script to run. For this, we'll create a Dockerfile with the following content:



```
FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y ffmpeg

WORKDIR /app

COPY rotate.sh .

RUN chmod +x rotate.sh

CMD ["/bin/bash"]


 
 ```


In this Dockerfile, we're using the latest Ubuntu image as the base image and installing the ffmpeg package, which we'll use to rotate the videos. We're also setting the working directory to /app, copying the rotate.sh script to the container, making it executable, and setting the default command to run a bash shell.
Next, we'll create the rotate.sh script. Here's an example script that you can use:




bash
 ```
#!/bin/bash

mkdir -p rotated

for file in *.mp4
do
    rotation=$(ffprobe -v error -select_streams v:0 -show_entries stream_tags=rotate -of default=nw=1:nk=1 "$file")
    if [ "$rotation" == "90" ] || [ "$rotation" == "270" ]; then
        ffmpeg -i "$file" -vf "transpose=2" "rotated/${file}_rotated.mp4"
    else
        cp "$file" "rotated/${file}"
    fi
done
 ```



This script will loop through all the MP4 files in the current directory and check if they need to be rotated. If a file has a rotation value of either 90 or 270 degrees, it'll rotate the video using the ffmpeg tool and save it in a subdirectory named rotated. If the video doesn't need to be rotated, it'll simply copy it to the rotated directory.
We can now build the Docker image using the following command in the same directory as the Dockerfile:




 ```
docker build -t mp4-rotator .
 ```
This will create a Docker image named mp4-rotator.
We can now use the Docker image to run our script on a directory containing MP4 files. Let's assume our MP4 files are located in a directory named videos. We can run the script using the following command:




bash
 ```
docker run --rm -v $(pwd)/videos:/app mp4-rotator ./rotate.sh
 ```



Here, we're running a container using the mp4-rotator image, mounting the videos directory as the working directory in the container, and running the rotate.sh script.

This will create a subdirectory named rotated inside the videos directory and save all the rotated videos in that directory with the suffix _rotated.mp4. The original videos will still be in the videos directory.


---

This was not working as is.
Reason: directory in docker container ie /app was mapped to local with mp4 and bask script could not be found with error like 



 ``` 
 docker: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "./rotate.sh": stat ./rotate.sh: no such file or directory: unknown."
 ```


 It was enough coping bash script rotate.sh in Dockerfile  to another directory ie /tools
 puting it to PATH worked like a charm :)).
 
 

Then I checked and if necessary rotated mp4 so they could played on some old fashioned mp4 player on TV.
new Dockerfile was like below



---




 ```
 FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y ffmpeg


WORKDIR /app

# Add /app to the PATH environment variable
ENV PATH="/tools:${PATH}"

#COPY rotate.sh .
COPY rotate.sh /tools/rotate.sh


RUN chmod +x /tools/rotate.sh

CMD ["/bin/bash"]
 ```
 
 and running docker with modified 
 ```
docker run --rm -v $(pwd)/path/to/mp4/video:/app mp4-rotator /tools/rotate.sh
 ```
### Contact
Created by: _lencz.sla@gmail.com_

