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


Steps to create the smaple-operator.

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







