# Setup an Autograding VM for your course in Autolab

If you did not setup an autograding VM during the installation then you can do it post installation by following the steps below:



1. Stop the server if its running.

   ```bash
   cd autolab-oneclick/server
   docker-compose stop
   ```

2. Navigate to *autolab-oneclick/server/Tango/vmms* and create a folder for your course.

   ```bash
   mkdir MY_COURSE
   ```

3. Inside this folder, you will need to place the following files.

   ```
   autolab-oneclick/server/Tango/vmms
   | - MY_COURSE
   	| - bashrc
   	| - Dockerfile
   	| - requirements.txt
   ```

   - If you are setting up a conda environment for your course then *bashrc* file is required to activiate the condo environment. The contents of the bashrc file will be:

     ```bash
     . /opt/conda/etc/profile.d/conda.sh
     conda activate MY_COURSE_ENV
     ```

   - Place the packages you want installed in your course environment in *requirements.txt* file.

   - Use and edit this sample Dockerfile. 

     ```dockerfile
     # Autolab - autograding docker image for your course
     
     FROM continuumio/miniconda3
     MAINTAINER Your Name 								# SET YOUR NAME
     
     RUN apt-get update --fix-missing
     RUN apt-get -y install sudo
     RUN apt-get install -y gcc
     RUN apt-get install -y make
     RUN apt-get install -y build-essential
     RUN apt-get -y install libgl1-mesa-glx
     RUN apt-get -y install procps
     RUN apt-get -y install vim
     RUN apt-get -y install zip
     
     WORKDIR /app
     ADD requirements.txt /app
     
     RUN conda create -n MY_COURSE_ENV python=3.8         # NAME YOUR COURSE ENVIRONMENT & SET PYTHON VERSION
     RUN echo "source activate MY_COURSE_ENV" > ~/.bashrc # SET MY_COURSE_ENV
     ENV PATH /opt/conda/envs/MY_COURSE_ENV/bin:$PATH     # SET MY_COURSE_ENV
     RUN pip install --upgrade pip --trusted-host pypi.python.org -r requirements.txt
     
     # Install autodriver																# LEAVE THE REMAINING AS IT IS
     WORKDIR /home
     RUN useradd autolab
     RUN useradd autograde
     RUN mkdir autolab autograde output
     ADD bashrc /home/autolab
     RUN mv /home/autolab/bashrc /home/autolab/.bashrc
     RUN chown autolab:autolab autolab
     RUN chown autolab:autolab output
     RUN chown autograde:autograde autograde
     RUN apt-get install -y git
     RUN git clone https://github.com/uncc-autolab/Tango.git
     WORKDIR Tango/autodriver
     RUN make clean && make
     RUN cp autodriver /usr/bin/autodriver
     RUN chmod +s /usr/bin/autodriver
     
     # Clean up
     WORKDIR /home
     RUN apt-get remove -y git
     RUN apt-get -y autoremove
     RUN rm -rf Tango/
     
     # Check installation
     RUN ls -l /home
     RUN which autodriver
     ```

4. To build this autograding VM on server start, make an entry in *autolab-oneclick/server/Tango/start.sh*

   ```bash
   echo "Building autograding VM for MY_COURSE"
   docker build -t autograding_MY_COURSE vmms/MY_COURSE/;
   ```

5. Add a mapping for your autograding VM in  *autolab-oneclick/server/docker-compose.yml*. This will be useful when you need to install or remove pacakages from your VM.

   ```dockerfile
   tango:
     build: ./Tango
     command: sh start.sh
     ports:
       - '8600:8600'
     privileged: true
     volumes: 
     - ./Tango/config.py:/opt/TangoService/Tango/config.py
     - ./Tango/start.sh:/opt/TangoService/Tango/start.sh
     - ./Tango/vmms/MY_COURSE:/opt/TangoService/Tango/vmms/MY_COURSE  # <-- Add this line for MY_COURSE
   ```

   

6. Start the server to build your autograding VM.

   ```bash
   cd autolab-oneclick/server
   docker-compose up
   ```

   

7. The autograding VM is built inside the tango VM. You can verify this installation by entering the tango VM

   ```bash
   docker ps
   # CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                      NAMES
   # ee9348dbdd09        server_web          "/sbin/my_init"          43 hours ago        Up 43 hours         0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   server_web_1
   # bdd0058b8084        mysql:5.7           "docker-entrypoint.s…"   7 days ago          Up 43 hours         33060/tcp, 0.0.0.0:32769->3306/tcp         server_db_1
   # 31d5437a4d68        server_tango        "sh start.sh"            7 days ago          Up 43 hours         0.0.0.0:8600->8600/tcp                     server_tango_1
   
   # Enter tango VM
   docker exec -it 31d5437a4d68 /bin/bash
   
   # Inside VM. Check built images.
   root@31d5437a4d68:/opt/TangoService/Tango> docker images
   # REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
   # autograding_MY_COURSE    latest              ce106993858b        6 days ago          1.39GB 
   # continuumio/miniconda3   latest              b4adc22212f1        6 months ago        429MB
   ```

   

8. In Autolab, while setting up autograder for your assignment, specify the VM name.

   ```
   Autolab > Course > assignment > Admin Options > Edit Assesment > Autograder > VM Image > autograding_MY_COURSE 
   ```

9. **(Optional)** Create the same conda environment for your course on the server. This will come in handy to test locally test your auto graders on the server machine before your finally deploy them in the course. 

   ```
   conda create -n MY_COURSE_ENV python=3.8
   pip install -r autolab-oneclick/server/vmms/MY_COURSE/requirements.txt
   ```

   

