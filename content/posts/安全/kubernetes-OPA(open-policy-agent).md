在本文中，我们将演示如何使用OPA执行最细粒度的安全策略。请注意，本文是一个系列的一部分，我们将基于“OPA作为代码介绍”和“集成OPA到Kubernetes”中获得的知识进行。如果你还没有这样做，请浏览本系列中已发表的文章。

https://www.magalix.com/blog/introducing-policy-as-code-the-open-policy-agent-opa
https://www.magalix.com/blog/integrating-open-policy-agent-opa-with-kubernetes-a-deep-dive-tutorial


你可能已经熟悉Pod安全策略，可以在其中对Pod应用非常特定的安全控制。例如，使用Linux内核功能，使用主机命名空间、网络、端口或文件系统，以及其他许多功能。使用OPA，你还可以对pods施加类似的控制，在本实验室中，我们将创建一个OPA策略，不允许在pods中创建有特权的容器。特权容器对主机的访问级别比非特权容器高。



# 为什么使用OPA而不是原生的Pod安全策略？

使用Pod安全策略来执行我们的安全策略并没有什么问题。然而，根据定义，PSP只能应用于pods。它们不能处理其他Kubernetes资源，如Ingresses、Deployments、Services等。OPA的强大之处在于它可以应用于任何Kubernetes资源。OPA作为一个许可控制器部署到Kubernetes，它拦截发送到API服务器的API调用，并验证和/或修改它们。相应地，你可以有一个统一的OPA策略，适用于系统的不同组件，而不仅仅是pods。例如，有一种策略，强制用户在其服务中使用公司的域，并确保用户只从公司的镜像存储库中提取镜像。请注意，我们使用的OPA是使用kube-mgmt部署的，而不是OPA Gatekeeper。



# Rego的策略代码

在本文中，我们假设你已经熟悉了OPA和Rego语言。我们还假设你有一个正在运行的Kubernetes集群，该集群部署了OPA和kube-mgmt容器。有关安装说明，请参阅我们的前一篇文章。我们的no-priv-pod.rego文件如下所示：
```
package kubernetes.admission
deny[msg] {
  c := input_containers[_]
  c.securityContext.privileged
  msg := sprintf("Privileged container is not allowed: %v, securityContext: %v", [c.name, c.securityContext])
}
input_containers[c] {
  c := input.request.object.spec.containers[_]
}
input_containers[c] {
  c := input.request.object.spec.initContainers[_]
}
```
让我们简要地浏览一下这个文件：

第1行：包含package。注意，你必须使用kubernetes.admission让政策工作。

第2行：Deny是默认对象，它将包含我们需要执行的策略。如果所包含的代码计算结果为true，则将违反策略。

第3行：我们定义了一个变量，它将容纳pod中的所有容器，并从稍后定义的input_containers[c]接收值。

第4行：如果pod包含“privileged”属性，则该语句为true。

第5行：当用户尝试运行特权容器时显示给他们的消息。它包括容器名称和违规的安全上下文。

第7-9行：input_containers[c]函数从请求对象中提取容器。注意，使用了_字符来遍历数组中的所有容器。在Rego中，你不需要定义循环—下划线字符将自动为你完成此操作。

第10-12行：我们再次为init容器定义函数。请注意，在Rego中，可以多次定义同一个函数。这样做是为了克服Rego函数中不能返回多个输出的限制。当调用函数名时，将执行两个函数，并使用AND操作符组合输出。因此，在我们的例子中，在一个或多个位置中存在一个有特权的容器将违反策略。



# 部署策略

OPA会在opa命名空间的ConfigMaps中找到它的策略。要将我们的代码应用到ConfigMap中，我们运行以下命令：

kubectl create configmap no-priv-pods --from-file=no-priv-pod.rego
kube-mgmt边车（sidecar）容器在opa命名空间中持续监视API服务器，以便你只需创建ConfigMap就可以部署策略。



运行策略

让我们通过尝试部署一个特权容器来确保我们的策略是有效的：
```
kubectl -n default apply -f - <<EOT
apiVersion: v1
kind: Pod
metadata:
 name: nginx-privileged
 labels:
  app: nginx-privileged
spec:
 containers:
 - name: nginx
  image: nginx
  securityContext:
   privileged: true #false
EOT
```
Error from server (Privileged container is not allowed: nginx, securityContext: {"privileged": true}): error when creating "STDIN": admission webhook "validating-webhook.openpolicyagent.org" denied the request: Privileged container is not allowed: nginx, securityContext: {"privileged": true}
请注意，我们有意将pod部署到默认命名空间，因为我们的admission webhook将忽略在opa命名空间或kube-system中创建的任何资源。



总结

OPA是一种通用的、平台无感的策略实施工具，可以通过多种方式与Kubernetes集成。

你可以使用OPA策略来模拟Pod安全策略，以防止在集群上调度特权容器。

因为OPA可以与其他Kubernetes资源一起工作，而不仅仅是Pods，所以建议使用它来创建跨越所有相关资源的集群级策略文档。
