FROM：kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.34.1/deploy/static/provider/cloud/deploy.yaml

坑1：
ingress-nginx的网络问题，hostNetwork: true
坑2：
kubernetes的版本太高1.18，但官方给的yaml里面的admissionregistration.k8s.io/v1beta1，而新版本要写成admissionregistration.k8s.io/v1
