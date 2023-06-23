# Redis

Here,we are going to dicuss two type of implementation of Redis : 

1. Stand-alone
2. Cluster

# Redis Standalone Mode Guide

This guide provides step-by-step instructions for using Redis in standalone mode.

## Prerequisites

Before you begin, ensure you have the following:

- Kubernetes cluster up and running
- kubectl command-line tool configured to access your Kubernetes cluster

## Deployment Steps

Follow these steps to use Redis in standalone mode:

1. Start the Redis server by opening a terminal and running the following command:

This will start the Redis server on the default port (6379). If you want to use a different port, specify it using the `--port` option:

      
2. Once the Redis server is running, you can interact with it using the Redis CLI (command line interface). Open another terminal and run the following command:


The Redis CLI will connect to the Redis server.

- Use this command to connect with Redis server : redis-cli -h <bindServer> -p <port>  (Use -h <bindServer> if accessing from external Server)

3. You can now start using Redis commands in redis-cli. Here are a few examples:

- Set a key-value pair:

  ```
  SET mykey myvalue
  ```

- Get the value of a key:

  ```
  GET mykey
  ```

- Delele a key-value pair:

  ```
  DEL mykey
  ```


Feel free to explore the Redis documentation for a comprehensive list of available commands and their usage.

4. To exit the Redis CLI, type `exit` or press `Ctrl+C`.

## Configuration

Redis provides various configuration options that you can customize based on your needs. By default, Redis uses a configuration file named `redis.conf`. You can modify this file to adjust settings like port number, persistence, security, and more.

To modify the Redis configuration:

- If using docker image,mount the redis.conf file : docker run -v /path/to/file/redis.conf:/usr/local/etc/redis/redis.conf -d --name myredis redis redis-server /usr/local/etc/redis/redis.conf

1. Locate the `redis.conf` file in your Redis installation directory.

2. Open the file in a text editor.

3. Make the necessary changes to the configuration parameters. Refer to the Redis documentation for detailed explanations of each configuration option.

4. Save the `redis.conf` file.

5. Restart the Redis server for the changes to take effect.



# Redis Cluster Mode Guide

This guide provides step-by-step instructions for using Redis in cluster mode.

## Configuration

1. Download Docker image
            
    Use the following command to pull docker image:`docker pull redis`


    For cluster, we are using six instances of Redis running, out of which three will be Master nodes and the other three will be Slave nodes. (We don’t need to specify them explicitly).  
        
2. Prepare Redis.conf file 

    So, we will be creating six folders, each having a ‘redis.conf’ file, that will be used to create docker instances. Use this command to do so:

        mkdir 7000 7001 7002 7003 7004 7005


    Note: These folders are names to identify which port are we going to provide to the Redis node.



    Now, in each folder created above, we will add redis.conf file, which will look like this:

        port 7000 
        cluster-enabled yes
        cluster-config-file nodes.conf 
        cluster-node-timeout 5000 
        appendonly yes 
        bind <server-IP>


    A few points to Note: 

    Remember to keep this port same as the folder name. For e.g. In folder 7001, the port will be 7001, in folder 7002, the port will be 7002 and so on.
    This `server-IP` must be replaced by the IP of the server we are using. This is to make our Redis instance accessible from the outside, where that server is also accessible.


    We are now ready with our minimal setup for creating our cluster. Let’s start the docker containers.

3. Start Docker containers    

    We will be starting the docker containers using the docker image downloaded in step 1. Use the following commands:

        docker run -v <folder-path>/7000/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name myredis-0 redis redis-server /usr/local/etc/redis/redis.conf
        docker run -v <folder-path>/7001/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name myredis-1 redis redis-server /usr/local/etc/redis/redis.conf
        docker run -v <folder-path>/7002/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name myredis-2 redis redis-server /usr/local/etc/redis/redis.conf
        docker run -v <folder-path>/7003/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name myredis-3 redis redis-server /usr/local/etc/redis/redis.conf
        docker run -v <folder-path>/7004/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name myredis-4 redis redis-server /usr/local/etc/redis/redis.conf
        docker run -v <folder-path>/7005/redis.conf:/usr/local/etc/redis/redis.conf -d --net=host --name myredis-5 redis redis-server /usr/local/etc/redis/redis.conf

    Please note:

        <folder-path> must be an absolute path.
        There must be proper permissions to read these ‘redis.conf’ files. Permissions can be set using command chmod 755 <file name>. We need sudo access to set permissions here.
        We have six docker containers running till here. The list of which can be obtained using the command: docker ps | grep redis

4. Create a cluster using Redis nodes

    Now, we will create a Redis cluster using the Redis nodes we have just started. For this, we need redis-cli. Well, this redis-cli is itself present inside the redis nodes we have created.



    So, we will open shell from any of the docker containers just created above, using this command:
        docker exec -it container-id sh

    We will create the cluster using the command:

        redis-cli --cluster create server-IP:7000 server-IP:7001 server-IP:7002 server-IP:7003 server-IP:7004 server-IP:7005 --cluster-replicas 1


    Please note:

    server-IP will be the same as that mentioned in the redis.conf file above.
    --cluster-replicas 1 will create three nodes as master nodes and the other three as their slave nodes.


    Within a few seconds, if everything is fine, a message will be displayed saying:

        [OK] All nodes agree about slots configuration.
        >>> Check for open slots...
        >>> Check slots coverage...
        [OK] All 16384 slots covered.


    Our Redis cluster is now up and running on ports 7000, 7002, 7002, 7003, 7004, and 7005.

## Usage 

Follow these steps to use Redis in cluster mode:

1. Once the Redis server is running, you can interact with it using the Redis CLI (command line interface).Connect to any of redis instance and interact with it using redis-cli. Open another terminal and run the following command:

The Redis CLI will connect to the Redis server.

- Use this command to connect with Redis server : redis-cli -c -h `bindServer` -p `port`  (Use -h `bindServer` if accessing from external Server)

2. You can now start using Redis commands in redis-cli. Here are a few examples:

   Remember to use the ASKING command before each CRUD operation to ensure the request is directed to the correct node based on key hashing.

   Example : 
    
        redis-cli -c -p 7000
        127.0.0.1:7000> Asking
        OK
        127.0.0.1:7000> SET key value
        -> Redirected to slot [12539] located at 127.0.0.1:7002
    

- Set a key-value pair:

  ```
  SET mykey myvalue
  ```

- Get the value of a key:

  ```
  GET mykey
  ```

- Delele a key-value pair:

  ```
  DEL mykey
  ```


Feel free to explore the Redis documentation for a comprehensive list of available commands and their usage.

3. To exit the Redis CLI, type `exit` or press `Ctrl+C`.


## Conclusion

If you face any difficulty in doing this, you can also refer to the official documentation of Redis.

Redis is now set up and running in standalone and cluster mode on your system. You can utilize its powerful features and commands to build applications that leverage in-memory data storage and caching.

For more advanced use cases and deployment scenarios, consider exploring Redis in other modes like clustered or replicated setups.

For additional information and detailed documentation, please refer to the official Redis website at https://redis.io/.

Happy Redis-ing!