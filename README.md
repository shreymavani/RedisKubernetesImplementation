# Redis On Kubernetes

Here,we are going to dicuss two type of implementation of Redis : 

1. Stand-alone
2. Cluster

## Prerequisites

Before you begin, ensure you have the following:

- Kubernetes cluster up and running
- kubectl command-line tool configured to access your Kubernetes cluster
- Download those yaml files and put it into a directory and open a terminal in that directory

# Redis Standalone Mode Guide

This guide provides step-by-step instructions for using Redis in standalone mode.

## Deployment Steps

Follow these steps to use Redis in standalone mode:

1. Start the Redis server by opening a terminal and running the following command:

- This will start the Redis server on the default port (6379). If you want to use a different port, specify it using the `--port` option:

        kubectl apply -f redis-deployment.yaml

        kubectl apply -f redis-service.yaml

## Usage
1. Once the Redis server is running, you can interact with it using the Redis CLI (command line interface). Open another terminal and run the following command:

- The Redis CLI will connect to the Redis server.

        kubectl exec -it redis-deployment-name sh -- redis-cli

2. You can now start using Redis commands in redis-cli. Here are a few examples:

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
   
## Clean up

1. Clean up 

       kubectl delete -f redis-deployment.yaml

       kubectl delete -f redis-service.yaml

# Redis Cluster Mode Guide

This guide provides step-by-step instructions for using Redis in cluster mode.

## Deployment Steps

Follow these steps to use Redis in standalone mode:

1. Start the Redis server by opening a terminal and running the following command:

        kubectl apply -f redis-deployment.yaml

        kubectl apply -f redis-service.yaml
  
3. Deploy Redis Cluster

- To do this, we run the following command and type yes to accept the configuration. The first three nodes become masters, and the last three become slaves.

        kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')

 *OR*

- If above command won't work or throw some error,follow below given commands.

      kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP} '

- Above command provides you the ipAddress of all running instances on kubernetes.After that execute the below given command :

      kubectl exec -it redis-cluster-0 sh -- redis-cli --cluster create server-IP1:6379 server-IP2:6379 server-IP3:6379 server-IP4:6379 server-IP5:6379 server-IP6:6379  --cluster-replicas 1

Note: server-IP1, server-IP2,...,server-IP6 is the output of this `kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP} '` command.
  
    Within a few seconds, if everything is fine, a message will be displayed saying:

        [OK] All nodes agree about slots configuration.
        >>> Check for open slots...
        >>> Check slots coverage...
        [OK] All 16384 slots covered.


    Our Redis cluster is now up and running on 6 different instances.

## Usage 

Follow these steps to use Redis in cluster mode:

1. Once the Redis server is running, you can interact with it using the Redis CLI (command line interface).Connect to any of redis instance and interact with it using redis-cli. Open another terminal and run the following command:

The Redis CLI will connect to the Redis server.

- Use this command to connect with Redis server :

        kubectl exec -it redis-cluster-0 sh -- redis-cli -c -h ipOfAnyOneInstance -p 6379  (Use -h flag if accessing from external Server)

2. You can now start using Redis commands in redis-cli. Here are a few examples:

   Remember to use the ASKING command before each CRUD operation to ensure the request is directed to the correct node based on key hashing.

   Example : 

        127.0.0.1:6379> Asking
        OK
        127.0.0.1:6379> SET key value
        -> Redirected to slot [12539] located at ipOfAnyOfOneInstance:6379
    

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


# Clean Up

1. Clean up 

       kubectl delete -f redis-statefulset.yaml

       kubectl delete -f redis-service.yaml

      kubectl delete pvc data-redis-cluster-0
      kubectl delete pvc data-redis-cluster-1
      kubectl delete pvc data-redis-cluster-2
      kubectl delete pvc data-redis-cluster-3
      kubectl delete pvc data-redis-cluster-4
      kubectl delete pvc data-redis-cluster-5

## Conclusion

If you face any difficulty in doing this, you can also refer to the official documentation of Redis.

Redis is now set up and running in standalone and cluster mode on your system. You can utilize its powerful features and commands to build applications that leverage in-memory data storage and caching.

For more advanced use cases and deployment scenarios, consider exploring Redis in other modes like clustered or replicated setups.

For additional information and detailed documentation, please refer to the official Redis website at https://redis.io/.

Happy Redis-ing!