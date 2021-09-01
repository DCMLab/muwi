# Dockerfile for MuWi modified by Dror Chawin
# And thanks to Dragan Espenschied (Rhizome)

FROM wikibase/wikibase:1.35-bundle AS base

WORKDIR /var/www/html/extensions
ADD ./download-extension.sh .


ADD https://github.com/wikidata/WikibaseImport/archive/master.tar.gz /WikibaseImport.tar.gz

RUN bash download-extension.sh OAuth;\
    bash download-extension.sh Elastica;\
    bash download-extension.sh CirrusSearch;\
    bash download-extension.sh WikibaseCirrusSearch;\
    bash download-extension.sh UniversalLanguageSelector;\
    bash download-extension.sh cldr;\
    bash download-extension.sh EntitySchema;\
    tar xzf OAuth.tar.gz;\
    tar xzf Elastica.tar.gz;\
    tar xzf CirrusSearch.tar.gz;\
    tar xzf WikibaseCirrusSearch.tar.gz;\
    tar xzf UniversalLanguageSelector.tar.gz;\
    tar xzf cldr.tar.gz;\
    tar xzf WikibaseImport.tar.gz;\
    tar xzf EntitySchema.tar.gz;\
    rm ./*.tar.gz;\
    rm download-extension.sh;

# -------------
# Add extensions to be installed via composer.

FROM composer@sha256:d374b2e1f715621e9d9929575d6b35b11cf4a6dc237d4a08f2e6d1611f534675 AS composer

COPY --from=base /var/www/html /var/www/html
WORKDIR /var/www/html
RUN rm /var/www/html/composer.lock

RUN composer install --no-dev


## these may depend on some extensions Rhizome has and we don't
#RUN COMPOSER=composer.local.json composer require --no-update mediawiki/chameleon-skin:~3.0;\
#    composer update mediawiki/chameleon-skin --no-dev -o;

#RUN COMPOSER=composer.local.json composer require --no-update professional-wiki/wikibase-local-media:*;\
#    composer update professional-wiki/wikibase-local-media --no-dev -o


# -----------------------------------------------
# Assembling final image.

FROM wikibase/wikibase:1.35-bundle

COPY --from=composer /var/www/html /var/www/html

WORKDIR /var/www/html # needed?

# --------------------------------------
# copying MuWi specific files
# this could be better?
COPY LocalSettings.php.wikibase-bundle.template /LocalSettings.php.wikibase-bundle.template
COPY extra-install.sh /
COPY EPFL_Logo_Digital_RGB_PROD_square_small.png /var/www/html/resources/assets/wiki.png
COPY extra-entrypoint-run-first.sh /
RUN cat /LocalSettings.php.wikibase-bundle.template >> /LocalSettings.php.template && rm /LocalSettings.php.wikibase-bundle.template
COPY oauth.ini /templates/oauth.ini
