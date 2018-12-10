# The helm charts help to set up Prometheus+Grafana quickly in a Kubernetes cluster.
This is a quick way to get monitoring with Prometheus+Grafana in Kubernetes up and running, not for production use.
Charts are originally from:
  Prometheus: prometheus-7.4.5
  Grafana: grafana-1.19.0
Modified locally for quick deployment and fixing some issues.

## Steps

1. Install Helm
```
curl -LO https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-linux-amd64.tar.gz
tar zxvf helm-v2.9.1-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/
rm -rf linux-amd64/
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller --upgrade
```
Helm is only required to install the charts. Once the installation is done, it(and its tiller) can be uninstalled.

2. Configure and install Prometheus
* Configure NFS at the bottom of prometheus/values.yaml
Point them to a valid NFS service IP and path
* Run helm to install Prometheus
```
cd helm_charts
helm install prometheus --name prometheus
```

3. Configure and install Grafana
```
helm install grafana --name grafana
```
Grafana will run on a NodePort, the port number (An integer of 30000+) can be found by running: kubectl get svc | grep grafana
The password for 'admin' user can be printed with: kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

4. Set up Grafana Dashboard
* Access with browswer to http://{master_ip}:{GrafanaNodePort}, login with admin user and the password found above.
* In grafana, define a datasource with the prometheus- node port
name: prometheus-server
type: prometheus
URL: http://prometheus-server.default.svc.cluster.local
* Create a dashboard with graphs.
We can import from https://grafana.com/dashboards
Browse and find the dashbaord you like, get its ID, like 6417 for "Kubernetes Cluster (Prometheus)". In Grafana web GUI, select Import dashboard, fill the ID and click 'Load", Fill the form to specify name, datasource etc and click "Import" to create the dashboard

5. Set up alerting in Grafana
* Choose and sign up a alerting service, like PagerDuty, which has 12-day free trial. 
* Create a service in PagerDuty, add email, phone number, Android/iOS app etc. and get its' integration key.
* In Grafana, go to Alerting tab to create a new 'Notificaiton channel', Select type "PagerDuty" and enter the integration key;
* In your Grafana dashboard, add a new Graph(alert needs a graph, doesn't work with counter or gauge yet), following example uses node_cpu. 
** In metrics tab, fill A with 'node_cpu_seconds_total'
** In Alert tab, specify the Name of the alert, keep condition as default, set a value to 'IS ABOVE' which you expect it will exceed to trigger the alert; in 'Notifications' tab assign your notification channel to 'Send to' and add your custom text to the message. Save and close the graph. Save the dashboard
** You should be able to see the alert line on your graph. Once the condition is true, you will get an alert.
