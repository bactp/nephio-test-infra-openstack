 kubectl delete crd backups.velero.io deletebackuprequests.velero.io downloadrequests.velero.io podvolumebackups.velero.io podvolumerestores.velero.io resticrepositories.velero.io restores.velero.io schedules.velero.io serverstatusrequests.velero.io volumesnapshotlocations.velero.io backupstoragelocations.velero.io

 kubectl delete deployment velero -n velero
kubectl delete ns velero
kubectl get crds | grep velero



sudo mv velero-v1.7.0-linux-amd64/velero /usr/local/bin/

kubectl logs deployment/velero -n velero -f

kubectl get deployment.apps 

velero backup create nginx-backup --selector app=nginx

velero backup describe nginx-backup

velero backup logs nginx-backup


velero install     --provider aws     --plugins velero/velero-plugin-for-aws:v1.2.1     --bucket velero     --secret-file ./credentials-velero     --use-volume-snapshots=false     --backup-location-config region=minio1,s3ForcePathStyle="true",s3Url=http://192.168.28.187:32077


nano credentials-velero
[default]
aws_access_key_id = nephio1234
aws_secret_access_key = secret1234
