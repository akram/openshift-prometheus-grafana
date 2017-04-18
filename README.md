```
oc new-project monitoring
oc new-app docker.io/hawkular/hawkular-grafana-datasource
oc expose svc hawkular-grafana-datasource
oadm pod-network make-projects-global monitoring

oc create -f https://raw.githubusercontent.com/hawkular/hawkular-openshift-agent/master/deploy/openshift/hawkular-openshift-agent-configmap.yaml -n monitoring
oc process -f https://raw.githubusercontent.com/hawkular/hawkular-openshift-agent/master/deploy/openshift/hawkular-openshift-agent.yaml | oc create -n monitoring -f -
oc adm policy add-cluster-role-to-user hawkular-openshift-agent system:serviceaccount:monitoring:hawkular-openshift-agent



oc new-app prom/prometheus
oc annotate svc prometheus prometheus.io/scrape='true'

oc create -f prometheus-config/prometheus-env.yaml
oc create -f prometheus-config/prometheus-configmap.yaml
oc volume --add dc/prometheus --name config-volume -t configmap --configmap-name  prometheus-configmap -m /etc/prometheus --overwrite
oc volume --add dc/prometheus --name rules-volume  -t configmap --configmap-name  prometheus-rules     -m /etc/prometheus-rules --overwrite




oc new-app prom/alertmanager
oc annotate svc alertmanager prometheus.io/scrape='true'
oc annotate svc alertmanager prometheus.io/path='/alertmanager/metrics'
oc create configmap alertmanager-templates --from-file=alertmanager-config/alertmanager-templates
oc create -f alertmanager-config/alertmanager-configmap.yaml
oc volume --add dc/alertmanager --name config-volume     -t configmap --configmap-name  alertmanager-configmap -m /etc/alertmanager           --overwrite
oc volume --add dc/alertmanager --name templates-volume  -t configmap --configmap-name  alertmanager-templates -m /etc/alertmanager-templates --overwrite


```
