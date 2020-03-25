# sample-operator
Example to create a sample operator using [operator-sdk](https://github.com/operator-framework/operator-sdk)

In this example we will create a sample-operator for deploying [sample application](https://github.com/shailendra14k/Examples/tree/master/Sample) based on spring-boot and camel. 

Prerequisites.

1. Build the sample App image:
~~~
https://github.com/shailendra14k/Examples/blob/master/Sample/README.md
~~~

2. Install operator-sdk 
~~~
https://github.com/operator-framework/operator-sdk/blob/master/doc/user/install-operator-sdk.md
~~~

3. Install golang
~~~
https://golang.org/dl/
~~~


## Steps to create the sample-operator.

1. Create sample-operator project
~~~
mkdir -p $GOPATH/src/github.com/shailendra14k/  // remane to your account.
cd $GOPATH/src/github.com/shailendra14k/
export GO111MODULE=on
operator-sdk new sample-operator
cd sample-operator
go mod tidy // Install dependencies
~~~

2. Verify the directory structure cretaed. 
~~~
├── build
│   ├── bin
│   │   ├── entrypoint
│   │   └── user_setup
│   └── Dockerfile
├── cmd
│   └── manager
│       └── main.go
├── deploy
│   ├── operator.yaml
│   ├── role_binding.yaml
│   ├── role.yaml
│   └── service_account.yaml
├── go.mod
├── go.sum
├── pkg
│   ├── apis
│   │   └── apis.go
│   └── controller
│       └── controller.go
├── tools.go
└── version
    └── version.go
~~~
More info about SDK [project layout](https://github.com/operator-framework/operator-sdk/blob/master/doc/project_layout.md) doc.

2. Create Custom Resource Definition (CRD)
~~~
operator-sdk add api --api-version=shailendra14k.com/v1alpha1 --kind=Sample
~~~

3. Update the Smaple CR Spec and Status at `pkg/apis/shailendra14k/v1alpha1/sample_types.go`
~~~Go
type SampleSpec struct {
  Size        int32             `json:"size"`
  BodyValue   string            `json:"bodyvalue"`

}

// SampleStatus defines the observed state of Sample
type SampleStatus struct {
    Nodes []string `json:"nodes"`
}

~~~

Run the below command to update the generaed code after modifyin *_types.go
~~~
operator-sdk generate k8s

//Generate the updated CRDs
operator-sdk generate crds
~~~

4. Create a new Controller to watch and reconcile Sample resource
~~~
operator-sdk add controller --api-version=shailendra14k.com/v1alpha1 --kind=Sample
~~~

5. Replace the default `pkg/controller/sample/sample_controller.go` with the sample_controller.go 


6. Build and push the Operator image to a reqistry.
~~~
operator-sdk build quay.io/shailendra14k/sample-operator:v0.1 --image-builder podman

//Verify the operator image cretaed locally
[root@shsingh sample-operator]# podman images
REPOSITORY                                                TAG      IMAGE ID       CREATED          SIZE
quay.io/shailendra14k/sample-operator                     v0.1     50fceb91c078   27 seconds ago   150 MB

// Push the image

podman push quay.io/shailendra14k/sample-operator:v0.1
~~~

7. Update the default `deploy/operator.yaml` with the correct image deatils
~~~GO
serviceAccountName: sample-operator
containers:
  - name: sample-operator
    # Replace this with the built image name
    image: quay.io/shailendra14k/sample-operator:v0.1
    command:
    - sample-operator
    imagePullPolicy: Always
~~~

## Steps to deploy the sample-operator on Openshift cluster

1. Create a new project
~~~
oc new-project sample-operator
~~~

2. Create the sample CRD
~~~
oc create -f deploy/crds/shailendra14k.com_samples_crd.yaml
~~~

3. Deploy the Operator along with set-up the RBAC
~~~
oc create -f deploy/service_account.yaml
oc create -f deploy/role.yaml
oc create -f deploy/role_binding.yaml
oc create -f deploy/operator.yaml
~~~

4. Verify the deploymnet
~~~
$ oc get all
NAME                                   READY     STATUS    RESTARTS   AGE
pod/sample-operator-867cf7c68f-jpjxk   1/1       Running   0          95s

NAME                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/sample-operator-metrics   ClusterIP   172.30.125.171   <none>        8383/TCP,8686/TCP   76s

NAME                              READY     UP-TO-DATE   AVAILABLE   AGE
deployment.apps/sample-operator   1/1       1            1           97s

NAME                                         DESIRED   CURRENT   READY     AGE
replicaset.apps/sample-operator-867cf7c68f   1         1         1         97s
~~~

5. Create the Sample Custom Resource(CR)
~~~GO
$ cat deploy/crds/shailendra14k.com_v1alpha1_sample_cr.yaml 
apiVersion: shailendra14k.com/v1alpha1
kind: Sample
metadata:
  name: example-sample
spec:
  # Add fields here
  size: 2
  bodyvalue: "Response received from POD : {{env:HOSTNAME}}"
~~~
~~~
$ oc create -f deploy/crds/shailendra14k.com_v1alpha1_sample_cr.yaml 
sample.shailendra14k.com/example-sample created
~~~








