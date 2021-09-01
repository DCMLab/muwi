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

Added to wdqs.yml AND to wdqs-updater.yml:

    - name: WIKIBASE_SCHEME
      value: "https"


Changes to LocalSettings.php.wikibase-bundle.template:

    # formatter URLs
    $wgWBRepoSettings['formatterUrlProperty'] = 'P2';

    # prevent anonymous edits and creation of user accounts
    $wgGroupPermissions['*']['edit'] = false;
    $wgGroupPermissions['*']['createaccount'] = false;

Add "New Item and "New Property to sidebar": Edit https://muwi.epfl.ch/w/index.php?title=MediaWiki:Sidebar to

    * navigation
    ** mainpage|mainpage-description
    ** Special:AllPages|List of articles
    ** Special:ListProperties|List of properties
    ** recentchanges-url|recentchanges
    ** randompage-url|randompage
    ** helppage|help-mediawiki
    * actions
    ** Special:NewItem|New item
    ** Special:NewProperty|New property
    * SEARCH
    * TOOLBOX
    * LANGUAGES
    
    
## Trying to make sending mail work

Added to /var/www/html/LocalSettings.php (without success):

$wgEnableEmail = true;

$wgSMTP = [
	'host'     => "smtp.mail.ch",
	'IDHost'   => "mail.ch",
	'port'     => 25,
	'auth'     => true,
	'username' => "muwi",
	'password' => "pw"       
];

## Changing the prefixes in wdqs_updater

Replace all `http://www.wikidata.org` by `https://muwi.epfl.ch` in the files `embed.wdqs.min.7422d3a4ab293343eb08.js` and `wdqs.min.80f60aedc9f4a347615d.js` in the folder `/usr/share/nginx/html/js`

## Configuring Quickstatements

* Go to https://muwi.epfl.ch/wiki/Special:OAuthConsumerRegistration/propose
  * Application name: QuickStatements
  * OAuth "callback" URL: https://mwquick.epfl.ch/api.php
  * Allow consumer to specify a callback in requests and use "callback" URL above as a required prefix: Yes
  * Rights: High-volume editing, Edit existing pages, Edit protected pages, Create, edit, and move pages
* Save consumerKey and consumerSecret and add it to /quickstatements/data/oauth.ini in the pod.
* Go to https://muwi.epfl.ch/wiki/Special:OAuthManageConsumers/proposed and approve the request

## Changing character limits for strings

*live*: In the Wikibase container, change /var/www/html/extensions/Wikibase/repo/config/Wikibase.default.php change the corresponding values in 'string-limits' (l.44)

## Logging into MYSQL

    kubectl exec -it [mysql-pod] -- mysql -u wikiuser -psqlpass -D my_wiki

## MYSQL dump

Log into mysql container and then:

    mysqldump -u wikiuser -psqlpass my_wiki > my_wiki.sql


### Deleting sites

    TRUNCATE TABLE sites;
    TRUNCATE TABLE site_identifiers;


## Building the docker image and pushing it to the registry

Navigate to the folder with the Dockerfile and run

    docker build -t muwi:[version_tag] .

Login to the registry (on campus or VPN)

    docker login ic-registry.epfl.ch

Determine the image's name and tag in the registry

    docker tag muwi:[local_tag] ic-registry.epfl.ch/dcml/muwi:[registry_tag]
    docker push ic-registry.epfl.ch/dcml/muwi:[registry_tag]
