# webserver-operator
// TODO(user): Add simple overview of use/purpose

## Description
// TODO(user): An in-depth paragraph about your project and overview of use

## Getting Started

### Prerequisites
- go version v1.20.0+
- docker version 17.03+.
- kubectl version v1.11.3+.
- Access to a Kubernetes v1.11.3+ cluster.

### To Deploy on the cluster
**Build and push your image to the location specified by `IMG`:**

```sh
make docker-build docker-push IMG=<some-registry>/webserver-operator:tag
```

**NOTE:** This image ought to be published in the personal registry you specified. 
And it is required to have access to pull the image from the working environment. 
Make sure you have the proper permission to the registry if the above commands don’t work.

**Install the CRDs into the cluster:**

```sh
make install
```

**Deploy the Manager to the cluster with the image specified by `IMG`:**

```sh
make deploy IMG=<some-registry>/webserver-operator:tag
```

> **NOTE**: If you encounter RBAC errors, you may need to grant yourself cluster-admin 
privileges or be logged in as admin.

**Create instances of your solution**
You can apply the samples (examples) from the config/sample:

```sh
kubectl apply -k config/samples/
```

>**NOTE**: Ensure that the samples has default values to test it out.

### To Uninstall
**Delete the instances (CRs) from the cluster:**

```sh
kubectl delete -k config/samples/
```

**Delete the APIs(CRDs) from the cluster:**

```sh
make uninstall
```

**UnDeploy the controller from the cluster:**

```sh
make undeploy
```

## Contributing
// TODO(user): Add detailed information on how you would like others to contribute to this project

**NOTE:** Run `make --help` for more information on all potential `make` targets

More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html)

## License

Copyright 2023.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

# 注意事项
## role
创建rabc的时候，role.yaml中定义了ClusterRole。
但是apply之后，关于deployments，services的权限可能会没有创建进去。这样会导致创建WebServer CR失败。
需要手动补全。

# 原理
webserver_types.go 里面 WebServerSpec 中定义的字段，就是 controller 获取的instance里面的spec字段。
现在 WebServerSpec 如下：
```go
type WebServerSpec struct {
	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	// The number of replicas that the webserver should have
	Replicas int `json:"replicas,omitempty"`

	// The container image of the webserver
	Image string `json:"image,omitempty"`

	// The port that the webserver should have
	Port int `json:"port,omitempty"`

	// The nodeport of the webserver
	NodePort int `json:"nodeport,omitempty"`

	// Foo is an example field of WebServer. Edit webserver_types.go to remove/update
	Foo string `json:"foo,omitempty"`
}
```
所以实际运行中 instance 的值为：
```sh
instance:&{TypeMeta:{Kind:WebServer APIVersion:my.domain/v1} ObjectMeta:{Name:webserver-sample GenerateName: Namespace:default SelfLink: UID:4bef8369-8dfe-48d5-a0b7-0707aa16ccf9 ResourceVersion:2326733 Generation:1 CreationTimestamp:2024-04-11 06:49:50 +0000 UTC DeletionTimestamp:<nil> DeletionGracePeriodSeconds:<nil> Labels:map[app.kubernetes.io/created-by:webserver-operator app.kubernetes.io/instance:webserver-sample app.kubernetes.io/managed-by:kustomize app.kubernetes.io/name:webserver app.kubernetes.io/part-of:webserver-operator] Annotations:map[kubectl.kubernetes.io/last-applied-configuration:{"apiVersion":"my.domain/v1","kind":"WebServer","metadata":{"annotations":{},"labels":{"app.kubernetes.io/created-by":"webserver-operator","app.kubernetes.io/instance":"webserver-sample","app.kubernetes.io/managed-by":"kustomize","app.kubernetes.io/name":"webserver","app.kubernetes.io/part-of":"webserver-operator"},"name":"webserver-sample","namespace":"default"},"spec":{"image":"nginx:1.23.1","nodeport":30010,"port":80,"replicas":3}}
] OwnerReferences:[] Finalizers:[] ManagedFields:[{Manager:kubectl-client-side-apply Operation:Update APIVersion:my.domain/v1 Time:2024-04-11 06:49:50 +0000 UTC FieldsType:FieldsV1 FieldsV1:{"f:metadata":{"f:annotations":{".":{},"f:kubectl.kubernetes.io/last-applied-configuration":{}},"f:labels":{".":{},"f:app.kubernetes.io/created-by":{},"f:app.kubernetes.io/instance":{},"f:app.kubernetes.io/managed-by":{},"f:app.kubernetes.io/name":{},"f:app.kubernetes.io/part-of":{}}},"f:spec":{".":{},"f:image":{},"f:nodeport":{},"f:port":{},"f:replicas":{}}} Subresource:}]} Spec:{Replicas:3 Image:nginx:1.23.1 Port:80 NodePort:30010 Foo:} Status:{}}
```

可以看到 Spec 字段中的值是一一对应的。

也只有定义了字段之后，才可以在 `sample` 的 spec 中使用这些字段。

## 后续的修改
现在controller中进行拟合的时候，目标值是固定的，后续可以写到一个 config 里面，这样方便统一管理。
