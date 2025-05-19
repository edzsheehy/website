---
reviewers:
- eparis
- pmorie
title: How to configure Redis using a ConfigMap
content_type: tutorial
weight: 30
---

<!-- overview -->

This page provides an example of how to configure [Redis](https://github.com/redis/redis) using a [ConfigMap](docs/concepts/configuration/configmap/). This procedure builds on the [Configure a Pod to Use a ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/) task.

## {{% heading "objectives" %}}

By the end of this tutorial, you will be able to perform the following:

* Create a ConfigMap with Redis configuration values.
* Create a Redis Pod that mounts and uses the created ConfigMap.
* Verify that the configuration was correctly applied.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- lessoncontent -->

## Tutorial

Follow these steps to configure a Redis cache using data stored in a ConfigMap.

### Create a ConfigMap

The first step is to create a ConfigMap with an empty configuration block. Run the following multi-line command in your shell to do so:

```shell
cat <<EOF >./example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: ""
EOF
```

### Apply your ConfigMap to your cluster

Run the following `kubectl` commands to apply the ConfigMap you just created to your Kubernetes cluster along with a Redis Pod manifest:

```shell
kubectl apply -f example-redis-config.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

### Examine your Redis Pod manifest

Examine the contents of the Redis Pod manifest and note the following:

* A volume named `config` is created by `spec.volumes[1]`.
* The `key` and `path` under `spec.volumes[1].configMap.items[0]` exposes the `redis-config` key from the 
  `example-redis-config` ConfigMap as a file named `redis.conf` on the `config` volume.
* The `config` volume is then mounted at `/redis-master` by `spec.containers[0].volumeMounts[1]`.

This has the net effect of exposing the data in `data.redis-config` from the `example-redis-config`
ConfigMap above as `/redis-master/redis.conf` inside the Pod. The following YAML file shows what this Pod should look like:

{{% code_sample file="pods/config/redis-pod.yaml" %}}

### Examine the created objects

Run the following `kubectl` command to retrieve the created objects for review:

```shell
kubectl get pod/redis configmap/example-redis-config 
```

You should see the following output:

```
NAME        READY   STATUS    RESTARTS   AGE
pod/redis   1/1     Running   0          8s

NAME                             DATA   AGE
configmap/example-redis-config   1      14s
```

Remember that we left the `redis-config` key in the `example-redis-config` ConfigMap blank. Run the following command to test if this is true:

```shell
kubectl describe configmap/example-redis-config
```

If you created your ConfigMap correctly, you should see an empty `redis-config` key:

```shell
Name:         example-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
```

### Verify the configuration

Use the following `kubectl exec` command to enter the Pod and run the `redis-cli` tool to check the current configuration:

```shell
kubectl exec -it redis -- redis-cli
```

#### Verify that maxmemory value and policy

Run the following command to check the value of `maxmemory`:

```shell
127.0.0.1:6379> CONFIG GET maxmemory
```

The resulting output should show the default value of `0`:

```shell
1) "maxmemory"
2) "0"
```

Similarly, run the following command to check the value of `maxmemory-policy`:

```shell
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

This should show the default value of `noeviction`:

```shell
1) "maxmemory-policy"
2) "noeviction"
```

### Configure the ConfigMap

The next step is to add some configuration values to the `example-redis-config` ConfigMap. Open your `example-redis-config.yaml` file and add the `maxmemory` and `maxmemory-policy` key-value pairs as shown here:

{{% code_sample file="pods/config/example-redis-config.yaml" %}}

Next, run the following `kubectl` command to apply the updated ConfigMap to your cluster:

```shell
kubectl apply -f example-redis-config.yaml
```

Finally, run the following command to confirm that the ConfigMap was updated:

```shell
kubectl describe configmap/example-redis-config
```

You should see the configuration values we just added:

```shell
Name:         example-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
----
maxmemory 2mb
maxmemory-policy allkeys-lru
```

### Restart the Redis Pod

Your Redis Pod must be restarted for it to retrieve the updated values from any ConfigMaps associated with it. You can do this by running the following `kubectl` commands that delete your Redis Pod and reinitialize it immediately afterward:

```shell
kubectl delete pod redis
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

### Verify the updated configuration

Just as you did [previously](#verify-the-configuration), use the following `kubectl exec` command to enter the Pod and run the `redis-cli` tool to check that your configuration has updated correctly:

```shell
kubectl exec -it redis -- redis-cli
```

Once again, run the following command to check the value of `maxmemory`:

```shell
127.0.0.1:6379> CONFIG GET maxmemory
```

It now should show the value of `2097152`:

```shell
1) "maxmemory"
2) "2097152"
```

Again, run the following command to check the value of `maxmemory-policy`:

```shell
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

It now reflects the desired value of `allkeys-lru`:

```shell
1) "maxmemory-policy"
2) "allkeys-lru"
```

### Finish up

To finish this tutorial, run the following command to clean up your work and delete the created resources:

```shell
kubectl delete pod/redis configmap/example-redis-config
```

## {{% heading "whatsnext" %}}

Check out these resources for more information on the topics covered in this tutorial:

* Learn more about [ConfigMaps](/docs/tasks/configure-pod-container/configure-pod-configmap/).
* Follow an example of [Updating configuration via a ConfigMap](/docs/tutorials/configuration/updating-configuration-via-a-configmap/).
