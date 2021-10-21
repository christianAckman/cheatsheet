# Cheatsheet

### Git

wipe all git history 
- `git reset $(git commit-tree HEAD^{tree} -m "add uat branch.")`

pop most recent commit
- `git reset --soft HEAD^`

stash all staged files
- `git stash push -m "CHANGES" -- $(git diff --staged --name-only)`

checkout someone's fork
```
git remote add theirusername git@github.com:theirusername/reponame.git
git fetch theirusername
git checkout -b mynamefortheirbranch theirusername/theirbranch
```

-------------------------

### OpenSSL

get expiration date of SSL cert (no SNI)
- `openssl s_client -showcerts -connect ${SERVER}:443 </dev/null | openssl x509 -noout -dates`

get expiration date of SSL cert (with SNI)
- `openssl s_client -showcerts -connect ${SERVER}:443 -servername ${SERVER} </dev/null | openssl x509 -noout -dates`

check if cert & key match
- `openssl x509 -in tls.crt -pubkey -noout -outform pem | shasum -a 256`
- `openssl pkey -in new-prod-tls.key -pubout -outform pem | shasum -a 256`

view PEM contents
- `openssl x509 -noout -text -in ${PEM_FILE}`

-------------------------

### GCP

list snapshots
- `gcloud compute snapshots list`

create a snapshot
- `gcloud compute disks snapshot ${DISK} --snapshot-names ${SNAPSHOT_NAME} --zone ${ZONE}`

create a disk from a snapshot
- `gcloud compute disks create --source-snapshot ${SNAPSHOT_NAME}`

get instances in a network
- `gcloud compute instances list --filter 'networkInterfaces.network=https://www.googleapis.com/compute/v1/projects/${PROJECT}/global/networks/${NETWORK}'`

enable firewall logging
- `gcloud compute firewall-rules update ${FIREWALL_RULE} --enable-logging`

get roles attached to service account 
- `gcloud projects get-iam-policy ${GCP_PROJECT} --flatten="bindings[].members" --format='table(bindings.role)' --filter="bindings.members:${SERVICE_ACCOUNT}"`

- enable deleted KMS key
```
gcloud kms keyrings list --location ${REGION}
gcloud kms keys versions list --key ${KEY} --location ${REGION} --keyring ${KEY_RING}
gcloud kms keys versions restore 1 --key ${KEY} --location ${REGION} --keyring ${KEY_RING}
gcloud kms keys versions enable 1 --key ${KEY} --location ${REGION} --keyring ${KEY_RING}
```

get available GKE versions
- `gcloud container get-server-config --region=${REGION} --format=json`

configure CLI
- `gcloud config set account ${SERVICE_ACCOUNT}`
- `gcloud config set project ${GCP_PROJECT}`
- `gcloud config set compute/zone ${REGION}`

switch gcloud contexts
- `gcloud config configurations list`
- `gcloud config configurations activate ${PROFILE}`

set service account
- `gcloud auth activate-service-account --key-file=${GCP_CREDENTIALS_JSON}`

setup GCR
- `gcloud auth configure-docker`

-------------------------

### AWS

get terminated spot instances:
- `aws ec2 describe-instances \  
    --filters Name=instance-lifecycle,Values=spot Name=instance-state-name,Values=terminated,stopped \
    --query "Reservations[*].Instances[*].InstanceId" | jq .`

---

### K8s

setup kubectl context (EKS)
- `aws eks update-kubeconfig --name ${cluster} --region ${region} --profile ${aws_profile}`

update a deployment's image, keeping history
- `kubectl set image deployment/${deployment} master ${image} -n ${namespace} --record`

watch deployment/daemonset status
- `kubectl rollout status deployment/${deploy} -n ${namespace}`
- `kubectl rollout status daemonset/${daemonset} -n ${namespace}`

restart all pods in a deployment/daemonset
- `kubectl rollout restart daemonset/${daemonset} -n ${namespace}`

kill all pending pods
- `kubectl | grep -v Running | awk '{print $1}' | xargs kubectl delete po --grace-period=0`

scale a deployment
- `kubectl scale deployment/myapp --replicas=2`

restart all pods in a namespace
- `kubectl delete pod --all -n ${NAMESPACE}`

exec into specific container within a pod
- `kubectl exec -it ${pod} -c ${container} /bin/bash` - ubuntu
- `kubectl exec -it ${pod} -c ${container} /bin/ash` - alpine

spin up a pod
- `kubectl run ${name} --image ${image} --port ${port}`

create svc for pod/deploy
- `kubectl expose pod ${pod} --port ${port}`

forward localhost:8080 to svc/deploy
- `kubectl port-forward svc/my-service ${local_port}:${pod_port}`
- `kubectl port-forward deploy/my-deployment ${local_port}:${pod_port}`

rerun cronjob
- `kubectl create job --from=cronjob/${cron_job} ${new_cron_name}`

check RBAC permissions
- `kubectl auth can-i create pod --namespace ${NAMESPACE} --as ${SERVICE_ACCOUNT}`

run a pod and target a node
- `kubectl run nginx --image nginx:latest --overrides='{"apiVersion": "v1", "spec": {"nodeSelector": { "cloud.google.com/gke-nodepool": "pool-1" }}}'`

set node as unschedulable
- `kubectl cordon ${NODE}`
- `kubectl taint nodes ${NODE} key1=value1:NoSchedule`

set node as schedulable
- `kubectl cordon ${NODE}`
- `kubectl taint nodes ${NODE} key1=value1:NoSchedule-`

evict all pods from a node
- `kubectl drain --force --ignore-daemonsets=true`

set storage class as expandable
- `kubectl patch storageclass gp2 -p '{"allowVolumeExpansion": true}'`

-------------------------

### Helm

rollback one helm version
- `helm rollback ${release}`

rollback specific version
- `helm rollback ${release} ${revision}`
