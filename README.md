# Cheatsheet

### Git

wipe all git history 
- `git reset $(git commit-tree HEAD^{tree} -m "add uat branch.")`

pop most recent commit
- `git reset --soft HEAD^`

stash all staged files
- `git stash push -m "CHANGES" -- $(git diff --staged --name-only)`

-------------------------

### OpenSSL

get expiration date of SSL cert (no SNI)
- `openssl s_client -showcerts -connect ${SERVER}:443 </dev/null | openssl x509 -noout -dates`

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

watch deployment status
- `kubectl rollout status deployment/${deploy} -n ${namespace}`

restart all pods in a deployment/daemonset
- `kubectl rollout restart daemonset/${daemonset} -n ${namespace}`

exec into specific container within a pod
- `kubectl exec -it ${pod} -c ${container} /bin/bash` - ubuntu
- `kubectl exec -it ${pod} -c ${container} /bin/ash` - alpine

rerun cronjob
- `kubectl create job --from=cronjob/${cron_job} ${new_cron_name}`

check RBAC permissions
- `kubectl auth can-i create pod --namespace ${NAMESPACE} --as ${SERVICE_ACCOUNT}`

-------------------------

### Helm

rollback one helm version
- `helm rollback ${release}`

rollback specific version
- `helm rollback ${release} ${revision}`
