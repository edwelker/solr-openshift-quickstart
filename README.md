## Quickstart to run Solr on Apache Tomcat 7 (JBoss EWS 2.0) on OpenShift

See the following for explicit steps:

create OpenShift application, Tomcat 7 (JBoss EWS 2.0)
rhc create-app <name> jbossews-2.0
if you created your application via the web interface, git clone the repository created for you.
cd into your repository directory
git rm -rf pom.xml src, which tells OpenShift that you are going to deploy a war directly.
Download the latest solr binary and untar
curl -O http://mirror.cc.columbia.edu/pub/software/apache/lucene/solr/4.6.0/solr-4.6.0.tgz && tar zxvf solr-4.6.0.tgz
Copy the solr war to /webapps in your repository.
cp solr-4.6.0/example/webapps/solr.war webapps
Add the following to the end of the "common.loader" property in your .openshift/config/catalina.properties file.  This will tell OpenShift to look for these directories in your OpenShift data directory (nothing will be there yet, but we'll add that later).

    ,${catalina.home}/../app-root/data/solr.home/ext/*.jar,${catalina.home}/../app-root/data/solr.home

So the end result should be

    common.loader=${catalina.base}/lib,${catalina.base}/lib/*.jar,${catalina.home}/lib,${catalina.home}/lib/*.jar,${catalina.home}/../app-root/data/solr.home/ext/*.jar,${catalina.home}/../app-root/data/solr.home
Edit your .openshift/config/web.xml file to point to the solr.home (add at the bottom)
<env-entry>
<env-entry-name>solr/home</env-entry-name>
<env-entry-value>../app-root/data/solr.home</env-entry-value>
<env-entry-type>java.lang.String</env-entry-type>
</env-entry>
Add the changes to your repository
git add webapps/solr.war
git add .openshift/config/catalina.properties
git add .openshift/config/web.xml
Commit your changes
git commit -m "initial Solr on OpenShift config"
Push your changes
git push origin master
ssh into your OpenShift repository
ssh 529f5cb5cc5f7fa13000010f@thedomain.com (or whatever your OS instance address is)
Download Solr to your app-root/data directory, and untar
cd app-root/data (or cd $OPENSHIFT_DATA_DIR)
curl -O http://mirror.cc.columbia.edu/pub/software/apache/lucene/solr/4.6.0/solr-4.6.0.tgz && tar zxvf solr-4.6.0.tgz
make a "solr.home" directory
mkdir solr.home
Copy the necessary jars, solr index and configuration files to solr.home (index/config), or solr.home/ext (jars)
cp -rf solr-4.6.0/example/lib/ext/ solr-4.6.0/example/solr/* solr-4.6.0/example/resources/log4j.properties solr.home
cp solr-4.6.0/contrib/dataimporthandler/lib/* solr-4.6.0/contrib/extraction/lib/* solr.home/ext
Restart all OpenShift applications and dependencies (could just use ctl_app, since no dependencies yet)
ctl_all restart
go to http://OS-server/solr/admin

How to create an OpenShift instance based on the Solr 4.6.0 Quickstart

Checkout the Solr Quickstart repo from Stash
git clone <this repo>
Create your OpenShift instance
rhc create-app solrX jbossews-2.0 -g jumbo
Change into the solr quickstart repo directory
cd solr-quickstart
Add the OpenShift instance as a remote
git remote add solr_os ssh://52a2294ccc5f7f11250000b5@yourdomain.com/~/git/solr4.git/  (or whatever your OS instance address is)
Force the push of the quickstart branch to your OpenShift instance's master branch
git push solr_os --force quickstart:master


Based on the OpenShift `jbossews` cartridge.  Docs for that:
https://github.com/openshift/origin-server/tree/master/cartridges/openshift-origin-cartridge-jbossews/README.md
