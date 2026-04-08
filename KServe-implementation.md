# Kserve Demonstration for Iris model

### Install Cert Manager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```

### Install KServe CRDs

```
kubectl create namespace kserve

helm install kserve-crd oci://ghcr.io/kserve/charts/kserve-crd \
  --version v0.16.0 \
  -n kserve \
  --wait
```

### Install KServe controller

```
helm install kserve oci://ghcr.io/kserve/charts/kserve \
  --version v0.16.0 \
  -n kserve \
  --set kserve.controller.deploymentMode=RawDeployment \
  --wait
```

### Deploy the Intent Classifier model

```
kubectl create namespace intent

cat <<EOF | kubectl apply -n intent -f -
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: intent-classifier
spec:
  predictor:
    model:
      modelFormat:
        name: sklearn
      storageUri: "https://github.com/hajsha/Intent-classifier-model/blob/c4200fc36a4e4eccfb0e78043f5e071397606b91/model/intent_model.pkl"
      resources:
        requests:
          cpu: "100m"
          memory: "512Mi"
        limits:
          cpu: "1"
          memory: "1Gi"
EOF

kubectl get inferenceservice intent-classifier -n intent
```

### Check the logs

kubectl get pods -n kserve
kubectl logs kserve-controller-manager-6d49cc494f-rqxd6 -n kserve
kubectl get HorizontalPodAutoscalers.autoscaling -n intent
kubectl get svc -n intent






### Port-forward to access the model

```
kubectl -n ml port-forward svc/<svc-name> 8080:80
```

### Inference the Model

```
curl -s -X POST http://localhost:8080/v1/models/intent-classifier:predict \
  -H "Content-Type: application/json" \
  -d '{"instances":["I want to cancel my subscription"]}' | jq
```


