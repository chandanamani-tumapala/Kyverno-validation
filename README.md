# Kyverno Policies for Kubernetes Resource Validation

This repository contains three Kyverno ClusterPolicy resources that enforce naming and labeling conventions for Kubernetes resources. The policies are designed to improve consistency and compliance in your Kubernetes environment by ensuring that Pods and ConfigMaps adhere to specific standards.

## Policies Overview

### 1. **Require Labels on Pods**
   - **Policy Name**: `require-labels`
   - **Kind**: `ClusterPolicy`
   - **API Version**: `kyverno.io/v1`
   - **Enforcement Action**: Enforce
   - **Resources Affected**: `Pod`
   - **Description**: This policy ensures that all `Pod` resources must have a label named `app`. If the label is missing, the resource will be rejected.
   - **Message**: "All Pods must have the label 'app' set."
   - **Pattern**:
     ```yaml
     metadata:
       labels:
         app: "?*"
     ```
   - **Example of Non-compliance**: A Pod without the `app` label will be rejected.
   - **Example of Compliance**:
     ```yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: test-pod-example
       labels:
         app: "test-pod-e"
     spec:
       containers:
         - name: nginx
           image: nginx
           resources:
             requests:
               memory: "64Mi"
               cpu: "250m"
             limits:
               memory: "128Mi"
               cpu: "500m"
     ```

   - **Example of Non-compliance**:
     ```yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: test-pod
       labels:
         app: ""
     spec:
       containers:
         - name: nginx
           image: nginx
           resources:
             requests:
               memory: "64Mi"
               cpu: "250m"
             limits:
               memory: "128Mi"
               cpu: "500m"
     ```

---

### 2. **Validate ConfigMap Filename Contains 'qwartz'**
   - **Policy Name**: `validate-filename-contains-qwartz`
   - **Kind**: `ClusterPolicy`
   - **API Version**: `kyverno.io/v1`
   - **Enforcement Action**: Enforce
   - **Resources Affected**: `ConfigMap`
   - **Description**: This policy ensures that the name of a `ConfigMap` contains the substring `qwartz`. If the name does not contain `qwartz`, the resource will be rejected.
   - **Message**: "The resource '{{request.object.metadata.name}}' violates the naming policy. The name must include the substring 'qwartz'. Please rename the resource to comply with this requirement."
   - **Pattern**:
     ```yaml
     metadata:
       name: "*qwartz*"
     ```
   - **Example of Non-compliance**: A ConfigMap with a name like `invalid-config` will be rejected.
   - **Example of Compliance**:
     ```yaml
     apiVersion: v1
     kind: ConfigMap
     metadata:
       name: valid-qwartz-config
       labels:
         app: demo
     data:
       key: value
     ```

   - **Example of Non-compliance**:
     ```yaml
     apiVersion: v1
     kind: ConfigMap
     metadata:
       name: invalid-config
       labels:
         app: demo
     data:
       key: value
     ```

---

### 3. **Validate ConfigMap Filename with Specific Prefixes and Suffixes**
   - **Policy Name**: `validate-file-name-labels`
   - **Kind**: `ClusterPolicy`
   - **API Version**: `kyverno.io/v1`
   - **Enforcement Action**: Enforce
   - **Resources Affected**: `ConfigMap`
   - **Description**: This policy enforces a specific naming convention for `ConfigMap` resources. The name must start with one of the following prefixes: `dv`, `bp`, `os`, or must contain the pattern `-s?s-`.
   - **Message**: "The resource '{{request.object.metadata.name}}' does not follow the naming convention: must start with 'dv', 'bp', 'os' or contain '-s?s-'. Please rename the resource to comply with the policy."
   - **Conditions**:
     - The name must start with `dv*`, `bp*`, or `os*`.
     - Alternatively, the name must contain `-s?s-`.
   - **Example of Non-compliance**: A ConfigMap named `invalid-config` will be rejected because it does not meet the naming conditions.
   - **Example of Compliance**:
     ```yaml
     apiVersion: v1
     kind: ConfigMap
     metadata:
       name: dv-qwartz-s1-config
     data:
       app-key4: "value4"   # Valid key
       app-key3: "value3"   # Valid key
     ```

   - **Example of Non-compliance**:
     ```yaml
     apiVersion: v1
     kind: ConfigMap
     metadata:
       name: invalid-config
     data:
       app-key1: "value1"
     ```

---

## Resources Created

Some resources in the Kubernetes cluster follow the policies outlined above, while others do not. Below are examples of both compliant and non-compliant resources:

### Compliant Resources

1. **Pod Example (Compliant with Policy 1)**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test-pod-example
     labels:
       app: "test-pod-e"
   spec:
     containers:
       - name: nginx
         image: nginx
         resources:
           requests:
             memory: "64Mi"
             cpu: "250m"
           limits:
             memory: "128Mi"
             cpu: "500m"
   ```
2. **Valid-qwartz-config**:
   ```yaml
   apiVersion: v1
kind: ConfigMap
metadata:
  name: valid-qwartz-config
  labels:
    app: demo
data:
  key: value
```
3. **Invalid-complex**:
  ```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: invalid-config
data:
  app-key1: "value1"
```
To apply these Kyverno policies to your Kubernetes cluster:

Save the policy YAML files into your local system 

kubectl -f require-labels.yaml
kubectl apply -f require-label-policy.yaml
kubectl apply -f test-pod.yaml
 kubectl apply -f validate-filename-contains-qwartz.yaml
 kubectl apply -f valid-config.yaml
 kubectl apply -f invalid-config.yaml
 kubectl apply -f validate-complex.yaml
 kubectl apply -f invalid-complex.yaml
you can check whether the resources which are meeting the policies are created or the one which are not meeting the policies are not been created and the error is also displed.
