# Some Useful Docker Commands

  - Check if you have previously built docker images with command
    - ```docker images```

  - Check if you have running docker containers:
    - ```docker ps```

  - Stop the running docker containers:
    - ```docker stop $(docker ps -q)```

  - Delete cached containers:
    - ```docker rm $(docker ps -aq)```

  - Delete all existing docker images
    - ```docker rmi $(docker images -q)```
    - **WARNING**: This **will** delete all docker images.

  - Stop and delete docker containers,volumes
    - ```docker-compose down ```

  - Stop docker containers in the compose file
    - ```docker-compose stop```

  - For log
    - ```docker logs -f <container_name>```

  - For attach (get in container and run command)
    - ```docker attach <container_name>```

  - Execute command in running container
    - ```docker exec -it <container_id> /bin/bash```

  - Run a docker image
    - ```docker run --name <container_name> -it <image_name>```

  - Start containers in compose file

    - ```docker-compose up```

  - Start containers in detacted mode

    - ```docker-compose up -d```

