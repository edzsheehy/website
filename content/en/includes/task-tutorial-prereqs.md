To complete this tutorial, you need the following:

* A Kubernetes cluster.
  * We recommend that you run your cluster with at least **two** [nodes](docs/concepts/architecture/nodes/) that are **not acting as control plane hosts**.
  * If you do not have a cluster yet, you can create one using [minikube](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/) or a Kubernetes playground (such as [Killercoda](https://killercoda.com/playgrounds/scenario/kubernetes), [KodeKloud](https://kodekloud.com/public-playgrounds), or [Play with Kubernetes](https://labs.play-with-k8s.com/)).
* The `kubectl` command-line tool, configured to communicate with your cluster.
  * The example shown on this page works with `kubectl` 1.14 and above.
* An understanding of the [Configure a Pod to Use a ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/) tutorial. 
