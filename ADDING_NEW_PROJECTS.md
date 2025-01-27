# Adding new projects

1. Go to `cncf/devstats-docker-images`:

- Add project entry to `devstats-helm/projects.yaml` file. (also update `all:`) Find projects orgs, repos, select start date, eventually add test coverage for complex regular expression in `cncf/devstatscode`:`regexp_test.go`.
- To identify repo and/or org name changes, date ranges for entrire projest use `cncf/devstats`:`util_sh/(repo|org)_name_changes_bigquery.sh org|org/repo`. There is also a more resource-consuming script, example use: `./util_sh/org_name_changes_complex.sh org 'year.201*'`.
- You may need to update `cncf/devstats`:`util_sql/(org_repo)_name_changes_bigquery.sql` to include newest months.
- For other Helm deployments (like LF or GraphQL) update `k8s/projects.yaml` or `gql/projects.yaml` or `devstats-helm/projects.yaml` file instead of `example/projects.yaml`.
- Update `./images/build_images.sh` (add project's directory).
- Update `./k8s/all_*.txt` or `./example/all_*.txt` or `./gql/all_*.txt` or `devstats-helm/all_*` or `./devstats-helm/projects.yaml` (lists of projects to process).
- Update `images/Dockerfile.full.prod` and `images/Dockerfile.full.test` files.
- Update `images/Dockerfile.minimal.prod` and `images/Dockerfile.minimal.test` files.


2. Go to `cncf/devstats`:

- Do not commit changes until all is ready, or commit with `[no deploy]` in the commit message.
- Update `projects.yaml` file, also update `all:`.
- Copy setup scripts and then adjust them: `cp -R oldproject/ projectname/`, `vim projectname/*`. Most them can be shared for all projects in `./shared/`, usually only `psql.sh` is project specific.
- Update `devel/all_*.txt all/psql.sh grafana/dashboards/all/dashboards.json scripts/all/repo_groups.sql devel/get_icon_type.sh devel/get_icon_source.sh devel/add_single_metric.sh` files.
- Add Google Analytics (GA) for the new domain and keep the `UA-...` code for deployment.
- Update automatic deploy script: `./devel/deploy_all.sh`.
- Update static index pages `apache/www/index_*`.
- Update `partials/projects.html partials/projects_health.html metrics/all/sync_vars.yaml` (number of projects and partials).
- Copy `metrics/oldproject` to `metrics/projectname`. Update `./metrics/projectname/vars.yaml` file.
- `cp -Rv scripts/oldproject/ scripts/projectname`, `vim scripts/projectname/*`. Usually it is only `repo_groups.sql` and in simple cases it can fallback to `scripts/shared/repo_groups.sql`, you can skip copy then.
- `cp -Rv grafana/oldproject/ grafana/projectname/` and then update files. Usually `%s/oldproject/newproject/g|w|next` and `%s/Old Project/New Project/g|w|next`.
- Try to source from Grafana with most similar project start data: `cp -Rv grafana/dashboards/oldproject/ grafana/dashboards/projectname/` and then update files.  Use `devel/mass_replace.sh` script, it contains some examples in the comments.
- Something like this: `` MODE=ss0 FROM='"oldproject"' TO='"newproject"' FILES=`find ./grafana/dashboards/newproject -type f -iname '*.json'` ./devel/mass_replace.sh ``.
- Update `grafana/dashboards/proj/dashboards.json` for all already existing projects, add new project using `devel/mass_replace.sh` or `devel/replace.sh`.
- For example: `./devel/dashboards_replace_from_to.sh dashboards.json` with `FROM` file containing old links and `TO` file containing new links.
- When adding new dashboard to all projects, you can add to single project (for example "cncf") and then populate to all others via something like:
- `` for f in `cat ../devstats-docker-images/k8s/all_test_projects.txt`; do cp grafana/dashboards/oldproject/new-contributors-table.json grafana/dashboards/$f/; done ``, then: `FROM_PROJ=oldproject ./util_sh/replace_proj_name_tag.sh new-contributors-table.json`.
- When adding new dashboard to projects that use dashboards folders (like Kubernetes) update `cncf/devstats:grafana/proj/custom_sqlite.sql` file.
- To actually deploy on bare metal follow `cncf/devstats:ADDING_NEW_PROJECT.md`.
- If not deploying, then generate grafana artwork: `./devel/update_artwork.sh`, then `./grafana/create_images.sh` or `./devel/icons_all.sh`.


3. Go to `cncf/devstats-docker-images`:

- Consider upgrading Grafana: `vim ./images/Dockerfile.grafana`.
- Run `DOCKER_USER=... SKIP_PATRONI=1 ./images/build_images.sh` to build a new image.
- Eventually run `DOCKER_USER=... ./images/remove_images.sh` to remove image(s) locally (new image is pushed to the Docker Hub).


4. Go to `cncf/devstats-helm`:

- Update `prod/README.md` specify new ranges for prod-only and test-only projects (at the bottom of the file).
- Update `devstats-helm/values.yaml` (add project).
- Now: N - index of the new project added to `github.com/cncf/devstats-helm/devstats-helm/values.yaml`. M=N+1. Inside `github.com/cncf/devstats-helm`:
- Consider `forceAddAll=1|tsdb|''` and `skipAddAll=1|''` flags when adding multiple projects, or any project which is disabled or not included in 'All ...'
- Use `forceAddAll=tsdb` to regenerate 'All CNCF' time series data.

While on the `devstats-test` namespace, `git pull` and then for example if N=55 (index of the new project):

- `git pull`.
- Install new project (excluding static pages and ingress): `helm install devstats-test-projname ./devstats-helm --set skipSecrets=1,indexPVsFrom=55,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,indexProvisionsFrom=55,indexCronsFrom=55,indexGrafanasFrom=55,indexServicesFrom=55,indexAffiliationsFrom=55,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,projectsOverride='+cncf\,+opencontainers\,+istio\,+knative\,+zephyr\,+linux\,+rkt\,+sam\,+azf\,+riff\,+fn\,+openwhisk\,+openfaas\,+cii\,+prestodb',skipECFRGReset=1,nCPUs=32,forceAddAll=tsdb`.
- Recreate static pages handler: `helm delete devstats-test-statics`, `helm install devstats-test-statics ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipProvisions=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipAPI=1,skipNamespaces=1,indexStaticsFrom=0,indexStaticsTo=1`.
- Recreate ingress with a new hostname: `helm delete devstats-test-ingress`, `helm install devstats-test-ingress ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipProvisions=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipStatic=1,skipAPI=1,skipNamespaces=1,indexDomainsFrom=0,indexDomainsTo=1,ingressClass=nginx-test,sslEnv=test`.
- Redeploy All CNCF Grafana (new project in health dashboards): `kubectl edit deployment devstats-grafana-all` - change `image:` add or remove `:latest` which will force rolling update without downtime.
- Run vars regenerate on all projects (excluding new one, this is needed to update home dahsboard's list of all projects): `helm install --generate-name ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionCommand='devstats-helm/vars.sh',useFlags='',indexProvisionsTo=55`.
- Update All CNCF repo groups definitions: `helm install --generate-name ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,indexProvisionsFrom=38,indexProvisionsTo=39,provisionCommand='./devstats-helm/repo_groups.sh',useRepos=1,skipECFRGReset=1`.
- Eventually update All CNCF tags definitions (will break dashboards for a while): `helm install --generate-name ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,indexProvisionsFrom=38,indexProvisionsTo=39,provisionCommand='./devstats-helm/tags.sh'`.
- Eventually reinit All CNCF: `helm install --generate-name ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,indexProvisionsFrom=38,indexProvisionsTo=39,provisionCommand='./devstats-helm/reinit.sh'`.

While on the `devstats-prod` namespace, `git pull` and then for example if N=55 (index of the new project):

- `git pull`.
- Install new project (excluding static pages and ingress): `helm install devstats-prod-projname ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,indexPVsFrom=55,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,indexProvisionsFrom=55,indexCronsFrom=55,indexGrafanasFrom=55,indexServicesFrom=55,indexAffiliationsFrom=55,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,testServer='',prodServer='1',skipECFRGReset=1,nCPUs=32,forceAddAll=tsdb`.
- Recreate static pages handler: `helm delete devstats-prod-statics`, `helm install devstats-prod-statics ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipProvisions=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipAPI=1,skipNamespaces=1,indexStaticsFrom=1`.
- Recreate ingress with a new hostname: `helm delete devstats-prod-ingress`, `helm install devstats-prod-ingress ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipProvisions=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipStatic=1,skipAPI=1,skipNamespaces=1,indexDomainsFrom=1,skipAliases=1,ingressClass=nginx-prod,sslEnv=prod`.
- Redeploy All CNCF Grafana (new project in health dashboards): `kubectl edit deployment devstats-grafana-all` - change `image:` add or remove `:latest` which will force rolling update without downtime.
- Run vars regenerate on all projects (excluding new one, this is needed to update home dahsboard's list of all projects): `helm install --generate-name ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,testServer='',prodServer='1',provisionImage='lukaszgryglicki/devstats-prod',provisionCommand='devstats-helm/vars.sh',useFlags='',indexProvisionsTo=55`.
- Update All CNCF repo groups definitions: `helm install --generate-name ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionImage='lukaszgryglicki/devstats-prod',indexProvisionsFrom=38,indexProvisionsTo=39,provisionCommand='./devstats-helm/repo_groups.sh',useRepos=1,skipECFRGReset=1`.
- Eventually update All CNCF tags definitions (will break dashboards for a while): `helm install --generate-name ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionImage='lukaszgryglicki/devstats-prod',indexProvisionsFrom=38,indexProvisionsTo=39,provisionCommand='./devstats-helm/tags.sh'`.
- Eventually reinit All CNCF: `helm install --generate-name ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionImage='lukaszgryglicki/devstats-prod',indexProvisionsFrom=38,indexProvisionsTo=39,provisionCommand='./devstats-helm/reinit.sh'`.

Both test & prod namespaces:

- To have all dashboards recreated you can also kill all Grafana pods via `../devstats-k8s-lf/util/delete_objects.sh po devstats-grafana`, deployments will recreate them with the newest projects lists.
- Delete intermediate helm installs - those with auto generated name like `devstats-helm-1565240123`: `helm delete devstats-helm-1565240123`.

Regenerate projects health on "summary" projects (follow `cncf/devstats-docker-images`:`devstats-helm/health.sh` instructions):

Test:

- Generate annotations on the test server: `helm install --generate-name ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionCommand='devstats-helm/annotations.sh'`
- Run health dashboards regenerate on All CNCF project: `helm install --generate-name ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionCommand='devstats-helm/health.sh',indexProvisionsFrom=38,indexProvisionsTo=39`.
- Run health dashboards regenerate on All GraphQL project: `helm install --generate-name ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionCommand='devstats-helm/health.sh',indexProvisionsFrom=48,indexProvisionsTo=49`.
- Run health dashboards regenerate on All CDF project: `helm install --generate-name ./devstats-helm --set skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,provisionCommand='devstats-helm/health.sh',indexProvisionsFrom=43,indexProvisionsTo=44`.

Prod:

- Generate annotations on the prod server: `helm install --generate-name ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,testServer='',prodServer='1',provisionImage='lukaszgryglicki/devstats-prod',provisionCommand='devstats-helm/annotations.sh'`.
- Run health dashboards regenerate on All CNCF project: `helm install --generate-name ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,testServer='',prodServer='1',provisionImage='lukaszgryglicki/devstats-prod',provisionCommand='devstats-helm/health.sh',indexProvisionsFrom=38,indexProvisionsTo=39`.
- Run health dashboards regenerate on GraphQL project: `helm install --generate-name ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,testServer='',prodServer='1',provisionImage='lukaszgryglicki/devstats-prod',provisionCommand='devstats-helm/health.sh',indexProvisionsFrom=48,indexProvisionsTo=49`.
- Run health dashboards regenerate on All CDF project: `helm install --generate-name ./devstats-helm --set namespace='devstats-prod',skipSecrets=1,skipPVs=1,skipBackupsPV=1,skipVacuum=1,skipBackups=1,skipBootstrap=1,skipCrons=1,skipAffiliations=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,skipStatic=1,skipAPI=1,skipNamespaces=1,testServer='',prodServer='1',provisionImage='lukaszgryglicki/devstats-prod',provisionCommand='devstats-helm/health.sh',indexProvisionsFrom=43,indexProvisionsTo=44`.


5. Go to `cncf/devstats-helm-lf`:

- Update `devstats-helm/values.yaml` (add project).
- Now: N - index of the new project added to `github.com/cncf/devstats-helm-lf/devstats-helm/values.yaml`. M=N+1. Inside `github.com/cncf/devstats-helm`:
- If N=63, then: `AWS_PROFILE=... KUBECONFIG=... helm2 install ./devstats-helm --set skipSecrets=1,indexPVsFrom=63,skipBootstrap=1,indexProvisionsFrom=63,indexCronsFrom=63,skipGrafanas=1,skipServices=1,skipNamespace=1 --name devstats-projname`.


6. Go to `cncf/devstats-helm-example` (optional):

- Update `README.md` - add new project.
- Update `github.com/cncf/devstats-helm-example/devstats-helm-example/values.yaml` (add project).
- Now: N - index of the new project added to `github.com/cncf/devstats-helm-example/devstats-helm-example/values.yaml`. M=N+1. Inside `github.com/cncf/devstats-helm-example`:
- Run `helm install ./devstats-helm-example --set skipSecrets=1,skipBootstrap=1,skipCrons=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,indexProvisionsFrom=N,indexProvisionsTo=M,indexPVsFrom=N,indexPVsTo=M` to create provisioning pods.
- Run `helm install ./devstats-helm-example --set skipSecrets=1,skipPVs=1,skipBootstrap=1,skipProvisions=1,skipGrafanas=1,skipServices=1,skipPostgres=1,skipIngress=1,indexCronsFrom=N,indexCronsTo=M` to create cronjobs (they will wait for provisioning to finish).
- Run `helm install ./devstats-helm-example --set skipSecrets=1,skipPVs=1,skipBootstrap=1,skipProvisions=1,skipCrons=1,skipPostgres=1,skipIngress=1,indexGrafanasFrom=N,indexGrafanasTo=M,indexServicesFrom=N,indexServicesTo=M` to create grafana deployments and services. Grafanas will be usable when full provisioning is completed.
- You can do 3 last steps in one step instead: `helm install ./devstats-helm-example --set skipSecrets=1,skipBootstrap=1,skipPostgres=1,skipIngress=1,indexProvisionsFrom=N,indexProvisionsTo=M,indexCronsFrom=N,indexCronsTo=M,indexGrafanasFrom=N,indexGrafanasTo=M,indexServicesFrom=N,indexServicesTo=M,indexPVsFrom=N,indexPVsTo=M`.
- Recreate ingress with a new hostname: `kubectl delete ingress devstats-ingress`, `helm install ./devstats-helm-example --set skipSecrets=1,skipPVs=1,skipBootstrap=1,skipProvisions=1,skipCrons=1,skipGrafanas=1,skipServices=1,skipPostgres=1`.
- Eventually do something very similar for `cncf/devstats-helm-graphql` or `cncf/devstats-helm-lf`.


7. Go to `cncf/contributors` (optional):

- Update `contrib_projects.yaml`.
- Eventually update `scripts/contrib/repo_groups.sql`.


8. Go to `cncf/velocity` (optional):

- Update `reports/cncf_projects_config.csv`.
- Update `BigQuery/velocity_lf.sql BigQuery/velocity_cncf.sql`.
- Update `map/hints.csv map/defmaps.csv map/urls.csv map/ranges_sane.csv`.


9. You should visit all dashboards and adjust date ranges and for some dashboards automatically selected values.

10. Update affiliations.

- Update cncf/gitdm affiliations with [official project maintainers](https://docs.google.com/spreadsheets/d/1Pr8cyp8RLrNGx9WBAgQvBzUUmqyOv69R7QAFKhacJEM/edit#gid=262035321).
