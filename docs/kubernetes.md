# Install Percona server for MongoDB on Kubernetes

1. Clone the percona-server-mongodb-operator repository:

    ``` {.bash data-prompt="$" }
    $ git clone -b v{{ release }} https://github.com/percona/percona-server-mongodb-operator
    $ cd percona-server-mongodb-operator
    ```

    !!! note

        It is crucial to specify the right branch with `-b`
        option while cloning the code on this step. Please be careful.

2. The Custom Resource Definition for Percona Server for MongoDB should be
    created from the `deploy/crd.yaml` file. The Custom Resource Definition
    extends the standard set of resources which Kubernetes “knows” about with the
    new items, in our case these items are the core of the operator.
    [Apply it](https://kubernetes.io/docs/reference/using-api/server-side-apply/)
    as follows:

    ``` {.bash data-prompt="$" }
    $ kubectl apply --server-side -f deploy/crd.yaml
    ```

    This step should be done only once; the step does not need to be repeated
    with any other Operator deployments.

3. Create a namespace and set the context for the namespace. The resource names
    must be unique within the namespace and provide a way to divide cluster
    resources between users spread across multiple projects.

    So, create the namespace and save it in the namespace context for subsequent
    commands as follows (replace the `<namespace name>` placeholder with some
    descriptive name):

    ``` {.bash data-prompt="$" }
    $ kubectl create namespace <namespace name>
    $ kubectl config set-context $(kubectl config current-context) --namespace=<namespace name>
    ```

    At success, you will see the message that `namespace/<namespace name>` was
    created, and the context was modified.

4. The role-based access control (RBAC) for Percona Server for MongoDB is
    configured with the `deploy/rbac.yaml` file. Role-based access is based on
    defined roles and the available actions which correspond to each role. The
    role and actions are defined for Kubernetes resources in the yaml file.
    Further details about users and roles can be found in [Kubernetes documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#default-roles-and-role-bindings).

    ``` {.bash data-prompt="$" }
    $ kubectl apply -f deploy/rbac.yaml
    ```

    !!! note

        Setting RBAC requires your user to have cluster-admin role
        privileges. For example, those using Google Kubernetes Engine can
        grant user needed privileges with the following command:

    ```default
    $ kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value core/account)
    ```

5. Start the operator within Kubernetes:

    ``` {.bash data-prompt="$" }
    $ kubectl apply -f deploy/operator.yaml
    ```

6. Add the MongoDB Users secrets to Kubernetes. These secrets
    should be placed as plain text in the stringData section of the
    `deploy/secrets.yaml` file as login name and
    passwords for the user accounts (see [Kubernetes
    documentation](https://kubernetes.io/docs/concepts/configuration/secret/)
    for details).

    After editing the yaml file, MongoDB Users secrets should be created
    using the following command:

    ``` {.bash data-prompt="$" }
    $ kubectl create -f deploy/secrets.yaml
    ```

    More details about secrets can be found in [Users](users.md#users).

7. Now certificates should be generated. By default, the Operator generates
    certificates automatically, and no actions are required at this step. Still,
    you can generate and apply your own certificates as secrets according
    to the [TLS instructions](TLS.md#tls).

8. After the operator is started, Percona Server for MongoDB cluster can
    be created with the following command:

    ``` {.bash data-prompt="$" }
    $ kubectl apply -f deploy/cr.yaml
    ```

    The creation process may take some time. The process is over when all Pods
    have reached their Running status. You can check it with the following command:

    ``` {.bash data-prompt="$" }
    $ kubectl get pods
    ```

    The result should look as follows:

    --8<-- "./docs/assets/code/kubectl-get-pods-response.txt"

9. Check connectivity to a newly created cluster.

    First of all, run a container with a MongoDB client and connect its console
    output to your terminal. The following command will do this, naming the new
    Pod `percona-client`:

    ``` {.bash data-prompt="$" }
    $ kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:{{ mongodb44recommended }} --restart=Never -- bash -il
    ```

    Executing it may require some time to deploy the correspondent Pod. Now run
    `mongo` tool in the percona-client command shell using the login (which is
    `userAdmin`) with a proper password obtained from the Secret, and a proper
    namespace name instead of the `<namespace name>` placeholder:

    ``` {.bash data-prompt="percona-client:/$" }
    percona-client:/$ mongo "mongodb://userAdmin:userAdmin123456@my-cluster-name-mongos.<namespace name>.svc.cluster.local/admin?ssl=false"
    ```
