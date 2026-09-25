# "Buenos Aires": Kubernetes Pod Crashing

## Description

There are two pods: "logger" and "logshipper" living in the default namespace. Unfortunately, logshipper has an issue (crashlooping) and is forbidden to see what logger is trying to say. Could you help fix Logshipper?
<br><br>
Do not change the K8S definition of the logshipper pod.
<br><br>
NOTE: Wait about 2 minutes after VM start for k3s to settle, then about 30 seconds after the pod is stably Ready before using "Check My Solution".

## Test

Wait about 2 minutes after VM start for k3s to settle. Then <kbd>kubectl get pods -l app=logshipper -o json | jq -r '.items[].status.containerStatuses[0].ready'</kbd> returns <kbd>true</kbd> and stays that way.
<br><br>
The logshipper pod specification has not been changed.
<br><br>
The "Check My Solution" button runs the script <i>/home/admin/agent/check.sh</i>, which you can see and execute.

## Clues


<b>1.</b> View the logs of the logshipper pod to see why it is failing: <kbd>kubectl get pod --show-labels</kbd> and <kbd>kubectl describe pod -l app=logshipper</kbd>.<br><br> 

<b>2.</b> You can view the logshipper pod configuration with the previous "describe" command to see the service account attached to the pod. Once you find that, look for the cluster role binding to see which cluster role the service account is attached to with <kbd>kubectl get ClusterRole</kbd><br><br>

<b>3.</b> Using the cluster role found <i>logshipper-cluster-role</i>, edit it to include the <i>get</i> and <i>watch</i> verbs in the cluster roles (this will default to Vim editor, to emulate the Esc key you can use <kbd>Ctrl [</kbd> ) <kbd>kubectl edit ClusterRole logshipper-cluster-role</kbd><br><br>

<b>4.</b>You may need to restart the logshipper pod. In a Deployment we can just <i>kubectl delete pod logshipper-</i>. We can also: <kbd>kubectl rollout restart deployment logshipper</kbd> or we can <kbd>kubectl scale deployment</kbd> down (--replicas=0) and up (--replicas=1) again. After a fix, wait until the VM has been up ≥2 minutes and the pod has been Ready ~30s before checking.


**check.sh**

```bash
#!/bin/bash
# DO NOT MODIFY THIS FILE ("Check My Solution" will fail)

MIN_UPTIME_S=120
MIN_RUNNING_S=30

uptime_s=$(awk '{print int($1)}' /proc/uptime 2>/dev/null) || uptime_s=0
if [ "$uptime_s" -lt "$MIN_UPTIME_S" ]; then
  echo -n "NO"
  exit 0
fi

json=$(sudo k3s kubectl get pods -l app=logshipper -o json 2>/dev/null) || {
  echo -n "NO"
  exit 0
}

ready=$(printf '%s' "$json" | jq -r '.items[0].status.containerStatuses[0].ready // empty')
ready=$(printf '%s' "$ready" | tr -d '\r')

if [ "$ready" != "true" ]; then
  echo -n "NO"
  exit 0
fi

started=$(printf '%s' "$json" | jq -r '.items[0].status.containerStatuses[0].state.running.startedAt // empty')
started=$(printf '%s' "$started" | tr -d '\r')
if [ -z "$started" ]; then
  echo -n "NO"
  exit 0
fi

started_epoch=$(date -d "$started" +%s 2>/dev/null) || {
  echo -n "NO"
  exit 0
}
now_epoch=$(date +%s)
if [ $((now_epoch - started_epoch)) -lt "$MIN_RUNNING_S" ]; then
  echo -n "NO"
  exit 0
fi

sa=$(printf '%s' "$json" | jq -r '.items[0].spec.serviceAccountName // empty')
sa=$(printf '%s' "$sa" | tr -d '\r')
if [ "$sa" != "logshipper-sa" ]; then
  echo -n "NO"
  exit 0
fi

logger_ready=$(sudo k3s kubectl get pods -l app=logger -o json 2>/dev/null \
  | jq -r '.items[0].status.containerStatuses[0].ready // empty' | tr -d '\r')
if [ "$logger_ready" != "true" ]; then
  echo -n "NO"
  exit 0
fi

logs=$(sudo k3s kubectl logs -l app=logshipper --tail=30 2>/dev/null) || {
  echo -n "NO"
  exit 0
}
if ! printf '%s' "$logs" | grep -q 'Thanks for fixing me'; then
  echo -n "NO"
  exit 0
fi

echo -n "OK"
exit 0
```
