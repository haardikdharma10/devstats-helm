# Changing projects state (moving to incubation, graduating etc)

1. Go to `cncf/devstats-docker-images`:

- Update status in `*/projects.yaml`.
- Add `graduated_date` or similar (`incubating_date`, `archived_date`).


2. Go to `cncf/devstats`:

- Follow instructions from `cncf/devstats`:`GRADUATING.md`.


3. Go to `cncf/devstats-docker-images`:

- Consider upgrading Grafana: `vim ./images/Dockerfile.grafana`.
- Run `DOCKER_USER=... SKIP_PATRONI=1 ./images/build_images.sh` to build a new images.
- Eventually run `DOCKER_USER=... ./images/remove_images.sh` to remove image(s) locally (new image is pushed to the Docker Hub).


4. Go to `cncf/devstats-helm`:

While on the `devstats-test` namespace: `git pull`, then:

- Recreate static pages handler: `../devstats-k8s-lf/util/delete_objects.sh po devstats-static-test`.
- If graduation/incubation in the past - generate annotations for this project: `helm install --generate-name ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionCommand='devstats-helm/annotations.sh',indexProvisionsFrom=X,indexProvisionsTo=Y`
- Run vars regenerate on all projects: `helm install --generate-name ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionCommand='devstats-helm/vars.sh'`.
- Recreate Grafanas: `../devstats-k8s-lf/util/delete_objects.sh po devstats-grafana`.
- Reinit TSDB data may be needed: `helm install --generate-name ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionCommand='./devstats-helm/reinit.sh',projectsOverride='+cncf\,+opencontainers\,+istio\,+knative\,+zephyr\,+linux\,+rkt\,+sam\,+azf\,+riff\,+fn\,+openwhisk\,+openfaas\,+cii\,+prestodb',indexProvisionsFrom=X,indexProvisionsTo=Y`.
- Delete intermediate helm installs - those with auto generated name like `devstats-helm-1565240123`: `helm delete devstats-helm-1565240123`.

While on the `devstats-prod` namespace: `git pull`, then:

- Recreate static pages handler: `../devstats-k8s-lf/util/delete_objects.sh po devstats-static`.
- If graduation/incubation in the past - generate annotations for this project: `helm install --generate-name ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,testServer='',prodServer='1',provisionImage='lukaszgryglicki/devstats-prod',provisionCommand='devstats-helm/annotations.sh',indexProvisionsFrom=X,indexProvisionsTo=Y`.
- Run vars regenerate on all projects: `helm install --generate-name ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,testServer='',prodServer='1',provisionImage='lukaszgryglicki/devstats-prod',provisionCommand='devstats-helm/vars.sh'`.
- Recreate Grafanas: `../devstats-k8s-lf/util/delete_objects.sh po devstats-grafana`.
- Reinit TSDB data may be needed: `helm install --generate-name ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionImage='lukaszgryglicki/devstats-prod',provisionCommand='./devstats-helm/reinit.sh',indexProvisionsFrom=X,indexProvisionsTo=Y`.
- Delete intermediate helm installs - those with auto generated name like `devstats-helm-1565240123`: `helm delete devstats-helm-1565240123`.

