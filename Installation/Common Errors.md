# Fixing errors during installation



## ActiveRecord::NoDatabaseError:

Incase you get the following error at step 6/6 in the installation: 

```bash
ActiveRecord::NoDatabaseError: Unknown database 'app_autolab_development'
```

This  is just a demo database creation error which can be resolved in the following manner

- Stop and delete the running autolab containers.

- ```bash
  docker stop $(docker ps -q)
  docker rm $(docker ps -aq)
  ```

- re run the installation

  ```bash
  ./install.sh -s   # or ./install.sh -l if doing local installation
  ```

- Since all the required docker images have already been created, the installation should be fast and the database creation should succeed.



## GetImageBlob: no such file or directory

```bash
open /var/lib/docker/tmp/GetImageBlob119673120: no such file or directory
./install.sh: line 98: ./install.sh: No such file or directory
Failed command: 
```

The solution is to restart docker.

```bash
sudo service docker stop
sudo service docker start
```

 Next re-run the installation.

If the above solution doesn’t work out then make sure the following path exists on your machine.

```bash
/var/lib/docker/tmp/
```

Create the path if it isn't there.

 

## IPv4 Forwarding is disabled:

```bash
When installing-> from Docker Image Ubuntu Xenial: IPv4 Forwarding is disabled:
```

 [Enable ipv4 forwarding](https://github.com/moby/moby/issues/490)

```bash
sysctl -w net.ipv4.ip_forward=1
```

Next, re-run the installation.

