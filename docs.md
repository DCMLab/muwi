# dcmlsrv1

`/triplestore/` --> yaml files translating the docker-compose file
every container gets its own pod

xxx.svc --> xxx-svc
translate environment variables into `name:` and `value:` (all values in "double quotes")

NOT translated from docker-compose file: `restart`, `expose` (in wdqs, wdqs-proxy and wdqs-updater)

## Important commands:

    kubectl describe "pod"
    kubectl create -f yaml
    kubectl delete -f yaml
    kubectl delete pod "pod"
    kubectl get pods [-w][-A]
    kubectl get nodes
    kubectl get svc

Using namespaces:

    kubectl get namespaces
    kubectl create ns "namespace"
    kubectl create -f yaml -n "namespace"

DON'T TOUCH namespace `kube-system`

## First adaptions

Added to wdqs-frontend.yml:

    - name: BRAND_TITLE
      value: "MuseWiki Query Service"

Added to wikibase.yml:

      - name: MW_SITE_NAME
        value: "MuseWiki"
      - name: MW_SITE_LANG
        value: "en"

Changes to LocalSettings.php.wikibase-bundle.template:

    # formatter URLs
    $wgWBRepoSettings['formatterUrlProperty'] = 'P2';

    # prevent anonymous edits and creation of user accounts
    $wgGroupPermissions['*']['edit'] = false;
    $wgGroupPermissions['*']['createaccount'] = false;

## Building the docker image and pushing it to the registry

Navigate to the folder with the Dockerfile and run

    docker build -t muwi:[version_tag] .

Login to the registry (on campus or VPN)

    docker login ic-registry.epfl.ch

Determine the image's name and tag in the registry

    docker tag muwi:[local_tag] ic-registry.epfl.ch/dcml/muwi:[registry_tag]
