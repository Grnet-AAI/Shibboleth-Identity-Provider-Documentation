
# Shibboleth Identity Provider 4

- [Shibboleth Identity Provider 4](#shibboleth-identity-provider-4)
- [Γενικά](#γενικά)
  - [Προετοιμασία συστήματος](#προετοιμασία-συστήματος)
- [Εγκατάσταση Shibboleth Identity Provider - Δημιουργία του Shibboleth idp.war](#εγκατάσταση-shibboleth-identity-provider---δημιουργία-του-shibboleth-idpwar)
  - [Metadata providers](#metadata-providers)
  - [Παραμετροποίηση αρχείων καταγραφής](#παραμετροποίηση-αρχείων-καταγραφής)
  - [Παραμετροποίηση επικοινωνίας με LDAP/AD](#παραμετροποίηση-επικοινωνίας-με-ldapad)
  - [Παραμετροποίηση attribute-resolver.xml](#παραμετροποίηση-attribute-resolverxml)
  - [Παραμετροποίηση απελευθέρωσης Attributes](#παραμετροποίηση-απελευθέρωσης-attributes)
- [Παραμετροποίηση Βάσης](#παραμετροποίηση-βάσης)
  - [Εγκατάσταση βάσης δεδομένων](#εγκατάσταση-βάσης-δεδομένων)
- [Παραμετροποίηση JPA Server Side Storage](#παραμετροποίηση-jpa-server-side-storage)
  - [Παραμετροποίηση PersistentId](#παραμετροποίηση-persistentid)
- [User consent και session information](#user-consent-και-session-information)
- [Παραμετροποίηση Single Log Out](#παραμετροποίηση-single-log-out)
- [Παραμετροποίηση Λογότυπου φορέα](#παραμετροποίηση-λογότυπου-φορέα)


# Γενικά
Για την ένταξη ενός Φορέα στην Ομοσπονδία ΔΗΛΟΣ του ΕΔΥΤΕ απαιτείται η υλοποίηση ενός Παρόχου Ταυτότητας συμβατού με τα πρότυπα SAML 2.0 του οργανισμού OASIS. Προτείνεται ο [Shibboleth Identity Provider](https://shibboleth.net/products/identity-provider.html).
Απαιτούμενη για τη λειτουργία του Shibboleth Identity Provider είναι η ύπαρξη του ακόλουθου λογισμικού

* Java Runtime Environment (jre)
* Java Servlet Container ( π.χ. Tomcat, Jetty)

Ο Shibboleth Identity Provider εκμεταλλευόμενος τις cross-platform δυνατότητες της Java μπορεί να εγκατασταθεί τόσο σε λειτουργικό σύστημα Microsoft Windows όσο και σε λειτουργικό σύστημα Linux. Πρόταση της Ομοσπονδίας ΔΗΛΟΣ του ΕΔΥΤΕ είναι ή χρήση λειτουργικού συστήματος Linux, και οι ακόλουθες οδηγίες είναι βασισμένες σε λειτουργικό σύστημα Debian 10.

Προτείνεται ισχυρά η εγκατάσταση πακέτων συστήματος από το λειτουργικό σύστημα, καθώς το μη "πακεταρισμένο" λογισμικό έχει σημαντικά μεγαλύτερο κόστος συντήρησης.


## Προετοιμασία συστήματος

**EntityID**

Το entityID στο SAML είναι μια συμβολοσειρά που χαρακτηρίζει μοναδικά ένα entity (είτε Identity Provider είτε Service Provider). Σύμφωνα με το [SAML 2.0 Core specification](https://www.oasis-open.org/committees/download.php/35711/sstc-saml-core-errata-2.0-wd-06-diff.pdf)
θα πρέπει να είναι ένα URI του οποίου το μήκος να μην ξεπερναά τους 1024 χαρακτήρες. Το entityID ενός Identity Provider δεν θα πρέπει να αλλάξει ποτέ και να είναι προφανές πως συνδέεται με τον Φορέα που διαχειρίζεται τον Identity Provider.

Προτείνεται να διαλέξετε ένα entityID με τα ακόλουθα στοιχεία

* Να είναι ένα σωστό URI
* Να είναι ένα absolute URL
* Να περιέχει το πρωτεύον DNS domain του φορέα σας
* Να περιέχει τη συμβολοσειρά 'idp'

**Ενημέρωση και αναβαθμίσεις πακέτων**

Για την διευκόλυνση της διαδικασίας, προτείνεται η ενημέρωση του λειτουργικού συστήματος όπως και η εγκατάσταση των πακέτων curl, vim. 



```bash
apt-get update && apt-get upgrade
apt-get install curl vim 
``` 


**Εγκατάσταση Java 11**



```
apt-get install openjdk-11-jdk
```

Σε περίπτωση που έχετε και άλλες εγκατεστημένες εκδόσεις της java μπορείτε να τις απεγκαταστήσετε ή επιλέξτε την openjdk-11 με 

```
update-alternatives --config java
```

**Παραμετροποίηση openLDAP (εάν απαιτείται)**

Σε αυτό το κομμάτι περιγράφεται η ρύθμιση του openLDAP server σας ώστε να γνωρίζει τα 2
schemas που χρησιμοποιούνται στην Ομοσπονδία ΔΗΛΟΣ του ΕΔYTE και στο
eduGAIN, το eduPerson και το schac.
Σε περίπτωση που δεν έχει προηγηθεί αυτή η ενέργεια στο παρελθόν πατήστε 
<details>
<summary>εδώ</summary>
Κατεβάστε το schac schema από [εδώ](https://wiki.refeds.org/download/attachments/44957731/schac-20150413-1.5.0.schema.txt?version=1&modificationDate=1429052013839&api=v2)

και το eduPerson αντίστοιχα από [εδώ](https://spaces.internet2.edu/download/attachments/2309/2xOpenLdapEduPerson-201602.zip?version=1&modificationDate=1469471690870&api=v2)

Για το schac:

* Σώστε το αρχείο ως schac.schema
* Δημιουργείστε το directory /tmp/schac
```
mkdir /tmp/schac
```
Δημιουργείστε ένα dummy αρχείο με το όνομα slapd.conf με περιεχόμενα
```
include /tmp/schac/schac.schema
```
Μέσα από το directory /tmp/schema εκτελέστε την ακόλουθη εντολή
```
slaptest -f slapd.conf -F .
```
που θα δημιουργήσει το αρχείο
```
/tmp/schac/cn\\=config/cn\\=schema/cn\\=\\{0\\}schac.ldif
```
Επεξεργαστείτε το αρχείο ώστε να:
* Αφαιρεθούν όλες οι γραμμές που *δεν* ξεκινούν με
  * dn:
  * objectClass:
  * cn:
  * olcAttributeTypes:
  * olcObjectClasses:
* Αλλαχθεί το dn: σε dn: cn=schac,cn=schema,cn=config
* Αλλάχθεί το cn: σε cn: schac



Μετονομάστε το αρχείο σε schac.ldif
```
mv /tmp/schac/cn\=config/cn\=schema/cn\=\{0\}schac.ldif /tmp/schac/schac.ldif
```

 Προσθέστε το schema με
```
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f /tmp/schac/schac.ldif
```
Και αντίστοιχα για το eduPerson schema:


Σώστε το αρχείο ως eduperson.schema
Δημιουργείστε το directory /tmp/eduperson
```
mkdir /tmp/eduperson
```
Δημιουργείστε ένα dummy αρχείο με το όνομα slapd.conf με περιεχόμενα
```
include /tmp/eduperson/edueprson.schema
```
- Μέσα από το directory /tmp/eduperson εκτελέστε την ακόλουθη εντολή
```
slaptest -f slapd.conf -F .
```
που θα δημιουργήσει το αρχείο /tmp/eduperson/cn\=config/cn\=schema/cn\=\{0\}eduperson.ldif

Επεξεργαστείτε το αρχείο ώστε να:
* Αφαιρεθούν όλες οι γραμμές που *δεν* ξεκινούν με
  * dn:
  * objectClass:
  * cn:
  * olcAttributeTypes:
  * olcObjectClasses:
* Αλλαχτεί το dn: σε dn: cn=eduperson,cn=eduperson,cn=config
* Αλλαχτεί το cn: σε cn: eduperson

Μετονομάστε το σε eduperson.ldif
```
mv /tmp/eduperson/cn\=config/cn\=schema/cn\=\{0\}eduperson.ldif /tmp/eduperson/eduperson.ldif
```

Προσθέστε το schema στον openLDAP με
```
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f /tmp/eduperson/eduperson.ldif
```
</details>


**Εγκατάσταση και παραμετροποίηση Tomcat 9 Servlet Container**


```
apt-get install tomcat9
```

Είναι απαραίτητη η Παραμετροποίηση κάποιων καθολικών παραμέτρων της Java για την εκκίνηση του Tomcat. O IdP χρειάζεται περισσότερη μνήμη και πρόσβαση σε μερικά αρχεία συστήματος:
Επεξεργαστείτε το αρχείο ώστε:

/etc/default/tomcat9
```
#/etc/default/tomcat9
# start the Tomcat with enough memory:
JAVA_OPTS = "-Djava.awt.headless = true -XX: + UseConcMarkSweepGC -Xms1024m -Xmx2048m"
```
Override configuration για το systemd ώστε να αποδοθούν δικαιώματα εγγραφής στο Tomcat:

```
systemctl edit tomcat9.service
```
Το παραπάνω θα δημιουργήσει ένα αρχείο στο /etc/systemd/system/tomcat9.service.d/ το override.conf, το οποίο πρέπει να Επεξεργαστείτε στην μορφή:

/etc/systemd/system/tomcat9.service.d/override.conf
```
[Service]
ReadWritePaths=/opt/shibboleth-idp/logs/
ReadWritePaths=/opt/shibboleth-idp/metadata/
```

**Παραμετροποίηση προεπιλεγμένης πόρτας**

Για λόγους ασφάλειάς η πόρτα 8080 απενεργοποιείται
Ο AJP connector ενεργοποιείται στο port 8009 στο οποίο θα έρχονται τα requests από τον Web Server.
Επεξεργαστείτε το αρχείο

/etc/tomcat9/server.xml
```xml
#/etc/tomcat9/server.xml
   <!-- Αναζητήστε το connector για την πόρτα 8009 και επεξεργαστείτε το  -->
  <Service name="Catalina">
    <!-- Απενεργοποίηση του 8080 -->
    <!-- non-SSL/TLS HTTP/1.1 Connector on port 8080 -->
    <!-- <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" /> -->
    <!-- ... -->
    <!-- Define an AJP 1.3 Connector on port 8009 -->
    <Connector port="8009"
               address="127.0.0.1"
               protocol="AJP/1.3"
               redirectPort="8443"
               enableLookups="false"
               useIPVHosts="true"
               maxPostSize="100000"
               URIEncoding="UTF-8"
               secretRequired="false" />
    <!-- ... -->
  </Service>
  <!-- ... -->
```
Απενεργοποιήστε το session persistence του Tomcat για να ελευθερώσετε τα log files σας από errors τύπου lack of persistence of the session
objects όταν κάνετε shutdown. Χρησιμοποιώντας τον editor της αρεσκείας σας, επεξεργαστείτε το αρχείο /etc/tomcat9/context.xml και
ενεργοποιήστε (αφαιρέστε τα σχόλια από) το ακόλουθο element:


/etc/tomcat9/context.xml
```xml
###/etc/tomcat9/context.xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
     ...
-->
<!-- The contents of this file will be loaded for each web application -->
<Context>
  
    <!-- Default set of monitored resources. If one of these changes, the    -->
    <!-- web application will be reloaded.                                   -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>
  
    <!-- Uncomment this to disable session persistence across Tomcat restarts -->
  
    <Manager pathname="" />
  
</Context>
```

**Εγκατάσταση Java Standard Tag Library (JSTL)**

```
apt install libtaglibs-standard-impl-java
```

και επεξεργαστείτε το αρχείο /etc/tomcat9/catalina.properties για να προστεθεί το path του jstl

```
#/etc/tomcat9/catalina.properties
common.loader="${catalina.base}/lib","${catalina.base}/lib/*.jar","${catalina.home}/lib","${catalina.home}/lib/*.jar","/usr/share/java/*.jar"
```
**Προετοιμασία για την ενεργοποίηση του Idp Servlet**

Θα πρέπει να δημιουργηθεί ένα deployment descriptor αρχείο που θα περιέχει τις απαιτούμενες λεπτομέρειες για τον Tomcat9 ώστε να διαβάσει και να κάνει deploy το war αρχείο του Shibboleth Identity Provider.

Δημιουργήστε το αρχείο /etc/tomcat9/Catalina/localhost/idp.xml με τα ακόλουθα περιεχόμενα:

```xml
<Context docBase="/opt/shibboleth-idp/war/idp.war"
         privileged="true"
         unpackWAR="true"
         antiResourceLocking="false"
         swallowOutput="true" />
```

**Μεταφόρτωση πιστοποιητικού της ομοσπονδίας ΔΗΛΟΣ**

Η Ομοσπονδία ΔΗΛΟΣ παρέχει metadata aggregates για τα Entities της Ομοσπονδίας και για τους SP που δημοσιεύονται στο eduGAIN, αν ο Identity Provider
σας συμμετέχει στο eduGAIN.
Τα metadata aggregates της Ομοσπονδίας ΔΗΛΟΣ του ΕΔYΤE είναι ψηφιακά υπογεγραμμένα με το ιδιωτικό κλειδί της Ομοσπονδίας. Το δημόσιο κλειδί που αντιστοιχεί στο ιδιωτικό αυτό κλειδί δημοσιεύεται εδώ και μπορείτε να το κατεβάσετε σε PEM format από το αντίστοιχο link.
Για να μπορεί ο Identity Provider να επιβεβαιώσει την γνησιότητα των metadata που διαβάζει από τα aggregates της ομοσπονδίας, θα πρέπει να γνωρίζει το δημόσιο αυτό κλειδί. Η απαιτούμενη παραμετροποίηση έχει ως εξής:

Δημιουργήστε τον κατάλογο:
```
mkdir /etc/ssl/aai/
```
Κατεβάστε το πιστοποιητικό και αποθηκεύστε το τοπικά
```
curl -o grnet-mdsigner.crt https://md.aai.grnet.gr/grnetaai_md_cert.pem
```
Υπολογίστε τα sha1 και sha256 fingerprints του πιστοποιητικού
```
openssl x509 -fingerprint -sha1 -noout -in grnet-mdsigner.crt
openssl x509 -fingerprint -sha256 -noout -in grnet-mdsigner.crt
```
Συγκρίνετε τις τιμές που υπολογίστηκαν με τις τιμές που δημοσιεύονται στο https://md.aai.grnet.gr/grnetaai_md_cert.html

Εάν οι τιμές είναι σωστές, αντιγράψτε το αρχείο στο directory /etc/ssl/aai/.


**Eγκατάσταση και παραμετροποίηση Apache server** 


Εγκαταστήστε τον apache http server
```
apt-get install apache2
```
Ενεργοποιήστε τα απαραίτητα modules *proxy, proxy\_ajp* και *ssl*
```
a2enmod proxy proxy_ajp ssl headers
```
Μπορείτε να επεξεργαστείτε το αρχείο /etc/apache2/sites-available/default-ssl ή να δημιουργήσετε ένα καινούργιο. Παραθέτουμε παρακάτω ένα
ελάχιστο παράδειγμα που προσφέρει την αναγκαία λειτουργικότητα.
```
<VirtualHost IDP-IP-ADRESS:443 [IDP-IPv6-ADRESS]:443>
  ServerName              idp.domainforea.gr
  
  SSLEngine on
  SSLCertificateFile      /etc/ssl/localcerts/idp.domainforea.gr.crt.pem
  SSLCertificateKeyFile   /etc/ssl/private/idp.domainforea.gr.key.pem
  
  AddDefaultCharset UTF-8
  
  <Location /idp>
    Require all granted
    ProxyPass ajp://localhost:8009/idp
    Header always append X-FRAME-OPTIONS "SAMEORIGIN"
  </Location>
 
<VirtualHost IDP-IP-ADRESS:8443 [IDP-IPv6-ADRESS]:8443>
  ServerName              idp.domainforea.gr
  
  SSLEngine on
  SSLCertificateFile      /etc/ssl/localcerts/idp.domainforea.gr.crt.pem
  SSLCertificateKeyFile   /etc/ssl/private/idp.domainforea.gr.key.pem
  
 
  SSLVerifyClient       optional_no_ca
  SSLVerifyDepth        10
  SSLOptions            +StdEnvVars +ExportCertData
  
  <Location /idp>
    Require all granted
    ProxyPass ajp://localhost:8009/idp
  </Location>
  
</VirtualHost>
```
Επανεκκινήστε τον apache http server
```
service apache2 restart
```
**Έλεγχος ώρας συστήματος**

Η σωστή ώρα συστήματος είναι πολύ σημαντική στο SAML καθώς τα SAML μηνύματα έχουν περιορισμένη χρονική ισχύ (συνήθως της τάξης των μερικών λεπτών) και κατά συνέπεια ο λάθος χρονισμός ενός συστήματος ή η λάθος
επιλογή ζώνης ώρας μπορεί να προκαλέσει σημαντικά προβλήματα στην επικοινωνία του Shibboleth Identity Provider με Service Providers που συμμετέχουν στην Ομοσπονδία ΔΗΛΟΣ του ΕΔΥΤΕ, ή το eduGAIN.

Επιβεβαιώστε ότι συνδέεστε σε κάποιους NTP servers με
```
ntpq -p
```
και ότι η ώρα συστήματος σας είναι η σωστή με
```
date -R
```
Σε περίπτωση που χρειάζεται να την ορίσετε εκ νέου
```
dpkg-reconfigure tzdata
```
# Εγκατάσταση Shibboleth Identity Provider - Δημιουργία του Shibboleth idp.war

Κατεβάστε το αρχείο εγκατάστασης καθώς και το αρχείο με το sha256 hash του αρχείου εγκατάστασης από το https://shibboleth.net/downloads/identity-provider/latest/

Για παράδειγμα, για την έκδοση 4.1.2 μεταβείτε στο directory /opt
```
cd /opt
```

```
wget https://shibboleth.net/downloads/identity-provider/latest/shibboleth-identity-provider-4.1.2.tar.gz
```
```
wget https://shibboleth.net/downloads/identity-provider/latest/shibboleth-identity-provider-4.1.2.tar.gz.sha256
```
Υπολογίστε το sha256 hash του αρχείου εγκατάστασης και συγκρίνετε το με την τιμή που βρίσκεται στο shibboleth-identity-provider-4.1.2.tar.gz.sha256
```
sha256sum shibboleth-identity-provider-4.1.2.tar.gz
```
Αποσυμπιέστε το αρχείο εγκατάστασης
```
tar xvfz shibboleth-identity-provider-4.1.2.tar.gz
```
Εκτελέστε το installation script με
```
cd shibboleth-identity-provider-4.1.2
```
```
JAVA_HOME=/usr/ bin/install.sh
```
Κατά την διαδικασία της εγκατάστασης θα ερωτηθείτε για τα ακόλουθα:
```
Installation Directory: Αν δεν υπάρχει κάποιος πολύ σημαντικός λόγος να αλλάξετε το default path της εγκατάστασης, αφήστε το στο /opt/shibboleth-idp

Hostname: Το FQDN του host

SAML EntityID: Το SAML EntityID του Identity Provider .

Attribute Scope: To Scope επηρεάζει τα λεγόμενα scoped attributes, τα οποία είναι τα ακόλουθα eduPersonScopedAffiliation,eduPersonPrincipalName, eduPersonUniqueId
και έχουν σύνταξη της μορφής <*value@scope>*
Το scope του Identity Provider δηλώνεται στα Metadata του και η τιμή του συνήθως συμπίπτει με το primary domain του φορέα.

Backchannel PKCS12 Password: Ο κωδικός με τον οποίον θα προστατευτεί το keystore στο οποίο αποθηκεύεται το TLS certificate που θαχρησιμοποιήσει ο Shibboleth Identity Provider για backchannel επικοινωνία.

Cookie Encryption Key Password: Ο Identity Provider θα δημιουργήσει ένα κλειδί συμμετρικής κρυπτογράφησης AES το οποίο θα χρησιμοποιεί για την κρυπτογράφηση των cookies και κάποιων επιπλέον δεδομένων για εσωτερική χρήση. Το κλειδί αυτό αποθηκεύεται
σε ένα Java Keystore File (JCEKS). Ο κωδικός που θα δώσετε εδώ θα χρησιμοποιηθεί για την προστασία του κλειδιού στο keystore (είναι ο κωδικός με τον οποίον προστατεύεται η πρόσβαση τόσο στο κλειδί το ίδιο όσο και στο keystore).
```
Παρακάτω παρουσιάζεται το output από ένα session εγκατάστασης:
```
root@statusreporting:/opt/shibboleth-identity-provider-4.1.2# JAVA_HOME=/usr/ bin/install.sh
Buildfile: /opt/shibboleth-identity-provider-4.1.2/bin/build.xml
 
install:
Source (Distribution) Directory (press <enter> to accept default): [/opt/shibboleth-identity-provider-4.1.2] ?
 
Installation Directory: [/opt/shibboleth-idp] ?
 
INFO [net.shibboleth.idp.installer.V4Install:158] - New Install.  Version: 4.1.2
Host Name: [83.212.174.166] ?
statusreporting.vm.grnet.gr
INFO [net.shibboleth.idp.installer.V4Install:601] - Creating idp-signing, CN = statusreporting.vm.grnet.gr URI = https://statusreporting.vm.grnet.gr/idp/shibboleth, keySize=3072
INFO [net.shibboleth.idp.installer.V4Install:601] - Creating idp-encryption, CN = statusreporting.vm.grnet.gr URI = https://statusreporting.vm.grnet.gr/idp/shibboleth, keySize=3072
Backchannel PKCS12 Password:
Re-enter password:
INFO [net.shibboleth.idp.installer.V4Install:644] - Creating backchannel keystore, CN = statusreporting.vm.grnet.gr URI = https://statusreporting.vm.grnet.gr/idp/shibboleth, keySize=3072
Cookie Encryption Key Password:
Re-enter password:
INFO [net.shibboleth.idp.installer.V4Install:685] - Creating backchannel keystore, CN = statusreporting.vm.grnet.gr URI = https://statusreporting.vm.grnet.gr/idp/shibboleth, keySize=3072
INFO [net.shibboleth.utilities.java.support.security.BasicKeystoreKeyStrategyTool:166] - No existing versioning property, initializing...
SAML EntityID: [https://statusreporting.vm.grnet.gr/idp/shibboleth] ?
 
Attribute Scope: [vm.grnet.gr] ?
 
INFO [net.shibboleth.idp.installer.V4Install:474] - Creating Metadata to /opt/shibboleth-idp/metadata/idp-metadata.xml
INFO [net.shibboleth.idp.installer.BuildWar:103] - Rebuilding /opt/shibboleth-idp/war/idp.war, Version 4.1.2
INFO [net.shibboleth.idp.installer.BuildWar:113] - Initial populate from /opt/shibboleth-idp/dist/webapp to /opt/shibboleth-idp/webpapp.tmp
INFO [net.shibboleth.idp.installer.BuildWar:92] - Overlay from /opt/shibboleth-idp/edit-webapp to /opt/shibboleth-idp/webpapp.tmp
INFO [net.shibboleth.idp.installer.BuildWar:125] - Creating war file /opt/shibboleth-idp/war/idp.war
 
BUILD SUCCESSFUL
Total time: 1 minute 23 seconds
root@statusreporting:/opt/shibboleth-identity-provider-4.1.2#
```

Για την συνέχεια του οδηγού μεταβείτε στο directory του Identity Provider
```
cd /opt/shibboleth-idp/
```
**Παραμετροποίηση Permissions**

Ο χρήστης ως ο οποίος τρέχουν τα processes του Tomcat9 θα πρέπει να έχει δικαίωμα να διαβάσει τα αρχεία που περιέχονται στο /opt/shibboleth-idp/conf/ και στο /opt/shibboleth-idp/credentials/ και να μπορεί να διαβάζει και να γράφει στα directories /opt/shibboleth-idp/logs/ και /opt/shibboleth-idp/metadata/
Η πρόταση της Δ.Ο. της Ομοσπονδίας ΔΗΛΟΣ του ΕΔΥΤΕ είναι να γίνει group owner των παραπάνω directories το tomcat group και να αποδοθούν τα κατάλληλα permissions στο group αυτό όπως περιγράφεται παρακάτω:
```
chgrp -R $( getent group | grep ^tomcat | cut -d ":" -f1 ) /opt/shibboleth-idp/conf /opt/shibboleth-idp/credentials
```
```
chmod -R g+r /opt/shibboleth-idp/conf /opt/shibboleth-idp/credentials
```
```
chown $( getent passwd | grep ^tomcat | cut -d ":" -f1 ):$( getent group | grep ^tomcat | cut -d ":" -f1 ) /opt/shibboleth-idp/logs /opt/shibboleth-idp/metadata
```
**Επανεκκίνηση του Tomcat9 και πρώτη δοκιμή**

Προτείνεται μαζί με την επανεκκίνηση να παρακολουθήσετε και τα σχετικά logs.
```
service tomcat9 restart | tail -f logs/idp-process.log
```
και στην συνέχεια Εκτελέστε το παρακάτω ώστε να δείτε την κατάσταση του Identity Provider
```
curl https://hostname/idp/status
```

**Προετοιμασία IdP Metadata**

Μπορείτε να επεξεργαστείτε το αρχείο /opt/shibboleth-idp/idp-metadata.xml ώστε να συμφωνεί με τις απαιτήσεις τις ομοσπονδίας και της συνομοσπονδίας eduGAIN. 
Παραθέτουμε ελάχιστο παράδειγμα που προσφέρει την αναγκαία λειτουργικότητα.
```xml
<!-- Το παρακάτω xml είναι παράδειγμα οι δικές σας τιμές θα διαφέρουν από αυτές-->
<?xml version="1.0" encoding="utf-8"?>
<EntityDescriptor xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xmlns:shibmd="urn:mace:shibboleth:metadata:1.0" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:xi="http://www.w3.org/2001/XInclude" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mdui="urn:oasis:names:tc:SAML:metadata:ui" xmlns:mdrpi="urn:oasis:names:tc:SAML:metadata:rpi" entityID="https://idp.admin.grnet.gr/idp/shibboleth">
  <IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <Extensions>
      <shibmd:Scope regexp="false">grnet-hq.admin.grnet.gr</shibmd:Scope>
      <mdui:UIInfo>
        <mdui:DisplayName xml:lang="en">Greek Research and Technology Network - GRNET</mdui:DisplayName>
        <mdui:DisplayName xml:lang="el">Εθνικό Δίκτυο Έρευνας και Τεχνολογίας - ΕΔΥΤΕ</mdui:DisplayName>
        <mdui:InformationURL xml:lang="en">http://www.grnet.gr/</mdui:InformationURL>
        <mdui:InformationURL xml:lang="el">http://www.grnet.gr/</mdui:InformationURL>
        <mdui:Logo  height="54" width="125">/path/to/urllogo</mdui:Logo>
        <mdui:Logo  height="54" width="54">/path/to/urlsmallerlogo</mdui:Logo>
      </mdui:UIInfo>
      <mdui:DiscoHints>
        <mdui:DomainHint>admin.grnet.gr</mdui:DomainHint>
        <mdui:DomainHint>noc.grnet.gr</mdui:DomainHint>
        <mdui:IPHint>195.251.28.0/23</mdui:IPHint>
        <mdui:IPHint>2001:648:2320::/48</mdui:IPHint>
        <mdui:IPHint>83.212.9.0/24</mdui:IPHint>
        <mdui:IPHint>2001:648:2340::/48</mdui:IPHint>
        <mdui:GeolocationHint>geo:37.98709500,23.765772</mdui:GeolocationHint>
      </mdui:DiscoHints>
    </Extensions>
    <KeyDescriptor use="signing">
      <ds:KeyInfo>
        <ds:X509Data>
          <ds:X509Certificate>
                            MIIDKDCCAhCgAwIBAgIJAJzvjilzprtbMA0GCSqGSIb3DQEBBQUAMB0xGzAZBgNV                            BAMTEmlkcC5hZG1pbi5ncm5ldC5ncjAeFw0xNDA0MDgxNjUyNDhaFw0xOTA0MDcx                            NjUyNDhaMB0xGzAZBgNVBAMTEmlkcC5hZG1pbi5ncm5ldC5ncjCCASIwDQYJKoZI                            hvcNAQEBBQADggEPADCCAQoCggEBAOzRbV+pz5RuGqwAOzI4jUXUSQXHFy6IYlLr                            VjisR2+7ZI24GybJAstcpmmNCtFQsM1CKte1x7TFHZFfDWCVv4IVGqtx9lLw+Jhg                            VjV8DzX0EYVIKyJimKWwIEn2BuEKWAcTUMdGehiYVeI4sxS+FgbiJxcWiMgq4bB5                            QV4gBoglEbI/S1vfoBBOEdKoEUamM+MAWCR83E2+JrhMXL+BrK+ojunyq/qyoL21                            GLTKL8jHTUgYMidSOx0oeMzTly8LzTONRDSs6ABxgQODU+b7qgjIqoCwY/kA1mUk                            ziqX/eQkxhK/ycf3HMrUVSwHLyOl5SMfpKWPcPuryTTTuFczmBsCAwEAAaNrMGkw                            SAYDVR0RBEEwP4ISaWRwLmFkbWluLmdybmV0LmdyhilodHRwczovL2lkcC5hZG1p                            bi5ncm5ldC5nci9pZHAvc2hpYmJvbGV0aDAdBgNVHQ4EFgQUb4DIDVDxNLIg4MDr                            gAyL+erDTzkwDQYJKoZIhvcNAQEFBQADggEBAED8EAttcxvYdQGcE8UpuDYyMqZ1                            gH0fSBbs61hh/bv4NjT/2ycWik/xtdiQj5foFcDNRBDD0Z09NteURUqqwaMXZrFX                            0YVUlIIZ6Pi7bTFRD4Rv+uX3edLVOZekPL9H28Rkk1ndCHsFLm2cQYd9Rd+DxjS4                            KRtNmyH8BYTzE8YKdX173kc2gdYKTAPNpt2Vj1+ctB9OgFybzIoEKwVtGA4upU5Z                            9vSsXQxRZQaYacBoPMBhUgU0dY+d7cA1pAwVsMbeXqPigEnmpMHapquvrzfO/a/m                            WuVAvsoC7QBvZa5PSfIqJ5NAA5IfvScAfYCKRZ0xEQudPJZ0SjEo44ZsMWo=
            </ds:X509Certificate>
        </ds:X509Data>
      </ds:KeyInfo>
    </KeyDescriptor>
    <KeyDescriptor use="encryption">
      <ds:KeyInfo>
        <ds:X509Data>
          <ds:X509Certificate>                            MIIDMzCCAhugAwIBAgIUe5nQPyaB8umX6usqdap9CxjvkI0wDQYJKoZIhvcNAQEL                            BQAwHTEbMBkGA1UEAwwSaWRwLmFkbWluLmdybmV0LmdyMB4XDTE3MDExNzEzNTA1                            MVoXDTM3MDExNzEzNTA1MVowHTEbMBkGA1UEAwwSaWRwLmFkbWluLmdybmV0Lmdy                            MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAhcVligXekUGmCkamOVMn                            mTpRorQTGJ1Ivo0mWvyFP+0QoK99X6GxyMH9KYBkVsO54KYWYzpR/pr4ORB1ruK9                            CGT0PWPBES4jNBzYnm0M/lZioIJ4eNBU7H1n114Jw4rveFbM2HoIvMfuve3NRjfp                            dkt+0si8NoMJcw6hTLxoEfz9Ekw1mxijTlbvEneU5rtD5NE2HNRb2jOiDepMFOY6                            swlonvL+BmotCOsXd1Q9p4m3M3GpAANpDdDuizCLAHCyCdUmfVRHSFv+sFFeSECw                            v9sB/3C57leOHTox/XZPAMHuv9Z/vjOH0GreaQr+8RyLtYop58kA/jdkLb+PR8c5                            LwIDAQABo2swaTAdBgNVHQ4EFgQUNbf4/cgy0pgvEJw3t3oB1L3JL6swSAYDVR0R                            BEEwP4ISaWRwLmFkbWluLmdybmV0LmdyhilodHRwczovL2lkcC5hZG1pbi5ncm5l                            dC5nci9pZHAvc2hpYmJvbGV0aDANBgkqhkiG9w0BAQsFAAOCAQEAPaTsz1b7fMPu                            rTHSc8sHxNZ6zXAnSYkFVMFW/YMf34ap9tTsMEjWSOmGJjwwwJg5vaTpkd5W+xyn                            Dn03uWbX11ussggWwRh0dQfLOpUQc1dwPKUEaQ0Jz+hJsRzXBw7/isHAj/77xBKE                            Kc/NysIbHdL6VpBkgAvuYXNBQAXlWntMXAaPohSkl3Bwq/V5PiJzwLoSWC7/Sxwu                            FcY6wMUW8/DTMWRW4zAaujQ6EqI5uYXx9GKWr1RrNoGsw5+ksOfZwIrnpxs1rhMa                            FHAIdXt9R9nljEY0qx9u41mHXqY2Asay6OpSG1UMNpYv+JCwtmZ+2AiceUmAHsVx                            P+fEVz9vow==
          </ds:X509Certificate>
        </ds:X509Data>
      </ds:KeyInfo>
    </KeyDescriptor>
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://idp.admin.grnet.gr/idp/profile/SAML2/Redirect/SLO"/>
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://idp.admin.grnet.gr/idp/profile/SAML2/POST/SLO"/>
    <NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:transient</NameIDFormat>
    <NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:persistent</NameIDFormat>
    <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://idp.admin.grnet.gr/idp/profile/SAML2/POST/SSO"/>
    <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://idp.admin.grnet.gr/idp/profile/SAML2/Redirect/SSO"/>
    <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="https://idp.admin.grnet.gr/idp/profile/SAML2/SOAP/ECP"/>
  </IDPSSODescriptor>
  <Organization>
    <OrganizationName xml:lang="en">Greek Research and Technology Network - GRNET</OrganizationName>
    <OrganizationName xml:lang="el">Εθνικό Δίκτυο Έρευνας και Τεχνολογίας - ΕΔΥΤΕ</OrganizationName>
    <OrganizationDisplayName xml:lang="en">Greek Research and Technology Network - GRNET</OrganizationDisplayName>
    <OrganizationDisplayName xml:lang="el">Εθνικό Δίκτυο Έρευνας και Τεχνολογίας - ΕΔΥΤΕ</OrganizationDisplayName>
    <OrganizationURL xml:lang="en">https://grnet.gr/</OrganizationURL>
    <OrganizationURL xml:lang="el">https://grnet.gr/</OrganizationURL>
  </Organization>
  <ContactPerson contactType="technical">
    <Company>Greek Research and Technology Network</Company>
    <EmailAddress>mailto:support@grnet.gr</EmailAddress>
    <TelephoneNumber>800-11-47638</TelephoneNumber>
    <TelephoneNumber>+30-2152157854</TelephoneNumber>
  </ContactPerson>
  <ContactPerson contactType="support">
    <Company>GRNET headquarters</Company>
    <EmailAddress>mailto:support@admin.grnet.gr</EmailAddress>
    <TelephoneNumber>+30-2107474275</TelephoneNumber>
  </ContactPerson>
  <ContactPerson xmlns:remd="http://refeds.org/metadata" contactType="other" remd:contactType="http://refeds.org/metadata/contactType/security">
    <GivenName>GRNET CERT</GivenName>
    <EmailAddress>mailto:cert@grnet.gr</EmailAddress>
    <TelephoneNumber>+30-2107474274</TelephoneNumber>
  </ContactPerson>
</EntityDescriptor>
```


## Metadata providers

Στο αρχείο metadata-providers.xml, ορίζονται οι πηγές από τις οποίες θα διαβάζει και θα εμπιστεύεται metadata ο ΙdP. Η Ομοσπονδία ΔΗΛΟΣ του ΕΔΥΤΕ παρέχει metadata aggregates για τα Entities της Ομοσπονδίας και για τους SP που δημοσιεύονται στο eduGAIN, αν ο Identity Provider σας συμμετέχει στο eduGAIN.
Στο αρχείο metadata-providers.xml θα πρέπει να ορίσετε τα απαραίτητα <MetadataProvider /> elements έτσι ώστε:

Να καταναλώνονται τα metadata της Ομοσπονδίας ΔΗΛΟΣ του ΕΔΥΤΕ από το
<https://md.aai.grnet.gr/aggregates/grnet-metadata.xml>

Σε περίπτωση που ο IDP σας συμμετέχει στο eduGAIN, να καταναλωνόνται
και τα metadata με τους Service Providers του eduGAIN από το
<https://md.aai.grnet.gr/feeds/edugain-sp-samlmd.xml>

Να χρησιμοποιούνται MetadataProvier τύπου
FileBackedHTTPMetadataProvider

Να ελέγχεται η γνησιότητα των metadata με έλεγχο της ψηφιακής
υπογραφής χρησιμοποιώντας SignatureValidation ΜetadataFilter

Να ελέγχεται η παλαιότητα των metadata έτσι ώστε να μην δέχεται
metadata aggregates με πολύ μεγάλη διάρκεια ή aggregates που δεν
λήγουν ποτέ με την χρήση RequiredValidUntil MetadataFilter

Οι προτάσεις αυτές υλοποιούνται στο παρακάτω υπόδειγμα
metadata-providers.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<MetadataProvider id="ShibbolethMetadata" xsi:type="ChainingMetadataProvider"
   xmlns="urn:mace:shibboleth:2.0:metadata"
   xmlns:security="urn:mace:shibboleth:2.0:security"
   xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
   xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata"
   xmlns:alg="urn:oasis:names:tc:SAML:metadata:algsupport"
   xmlns:ds="http://www.w3.org/2000/09/xmldsig#"
   xmlns:ds11="http://www.w3.org/2009/xmldsig11#"
   xmlns:enc="http://www.w3.org/2001/04/xmlenc#"
   xmlns:enc11="http://www.w3.org/2009/xmlenc11#"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="urn:mace:shibboleth:2.0:metadata http://shibboleth.net/schema/idp/shibboleth-metadata.xsd
                                          urn:mace:shibboleth:2.0:security http://shibboleth.net/schema/idp/shibboleth-security.xsd
                                          urn:oasis:names:tc:SAML:2.0:assertion http://docs.oasis-open.org/security/saml/v2.0/saml-schema-assertion-2.0.xsd
                                          urn:oasis:names:tc:SAML:2.0:metadata http://docs.oasis-open.org/security/saml/v2.0/saml-schema-metadata-2.0.xsd
                                          urn:oasis:names:tc:SAML:metadata:algsupport http://docs.oasis-open.org/security/saml/Post2.0/sstc-saml-metadata-algsupport-v1.0.xsd
                                          http://www.w3.org/2000/09/xmldsig# http://www.w3.org/TR/2002/REC-xmldsig-core-20020212/xmldsig-core-schema.xsd
                                          http://www.w3.org/2009/xmldsig11# http://www.w3.org/TR/2013/REC-xmldsig-core1-20130411/xmldsig11-schema.xsd
                                          http://www.w3.org/2001/04/xmlenc# http://www.w3.org/TR/xmlenc-core/xenc-schema.xsd
                                          http://www.w3.org/2009/xmlenc11# http://www.w3.org/TR/2013/REC-xmlenc-core1-20130411/xenc-schema-11.xsd"
   sortKey="1">
 
 
<MetadataProvider
   xmlns="urn:mace:shibboleth:2.0:metadata"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="urn:mace:shibboleth:2.0:metadata http://shibboleth.net/schema/idp/shibboleth-metadata.xsd"
   xsi:type="FileBackedHTTPMetadataProvider"
   id="GRNETFederation"
   metadataURL="https://md.aai.grnet.gr/aggregates/grnet-metadata.xml"
   backingFile="%{idp.home}/metadata/grnet-metadata.xml">
   <MetadataFilter xsi:type="RequiredValidUntil" maxValidityInterval="P7D" />
   <MetadataFilter xsi:type="SignatureValidation" requireSignedRoot="true"
     certificateFile="/etc/ssl/aai/grnet-mdsigner.crt" />
 </MetadataProvider>
 
<!-- Metadata των Service Providers που συμμετέχουν στο eduGAIN -->
<MetadataProvider
   xmlns="urn:mace:shibboleth:2.0:metadata"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="urn:mace:shibboleth:2.0:metadata http://shibboleth.net/schema/idp/shibboleth-metadata.xsd"
   xsi:type="FileBackedHTTPMetadataProvider"
   id="GRNETEdugainDownstream"
  metadataURL="https://md.aai.grnet.gr/feeds/edugain-sp-samlmd.xml"
   backingFile="%{idp.home}/metadata/edugain-metadata.xml">
  <MetadataFilter xsi:type="RequiredValidUntil" maxValidityInterval="P7D" />
  <MetadataFilter xsi:type="SignatureValidation" requireSignedRoot="true"
    certificateFile="/etc/ssl/aai/grnet-mdsigner.crt" />
</MetadataProvider>
 
 
 
</MetadataProvider>
```

## Παραμετροποίηση αρχείων καταγραφής
Στο αρχείο /opt/shibboleth-idp/conf/logback.xml παραμετροποιούνται τα επίπεδα των Loggers που χρησιμοποιεί ο Identity Provider. Επιγραμματικά, τα σημαντικότερα από αυτά και οι τιμές που προτείνονται:
```xml
<variable name="idp.loglevel.idp" value="INFO" />` : Μηνύματα σχετικά με τον IdP. Χρησιμο να γίνει DEBUG για οποιαδήποτε προσπάθεια troubleshooting.

<variable name="idp.loglevel.ldap" value="WARN" />` : Mηνύματα σχετικα με την επικοινωνία με τον LDAP server. Χρήσιμο να γίνει DEBUG σε περιπτώσεις προβλημάτων αυθεντικοποίησης.

<variable name="idp.loglevel.messages" value="INFO" />` : SAML μηνύματα που λαμβάνει και στέλνει ο IdP.

<variable name="idp.loglevel.encryption" value="INFO" />` : Στο DEBUG level εμφανίζει στο idp-process.log τα SAML Assertions unencrypted. Πολύ χρήσιμο για debugging οποιουδήποτε προβλήματος καθώς μπορείτε να δείτε ακριβως το Assertion που στεέλνεται σε κάθε IdP.

<variable name="idp.loglevel.opensaml" value="INFO" />` : Μηνύματα από την βιβλιοθήκη OpenSAML.
```
Προτείνεται η παραμετροποίηση των παρακάτω τιμών:


Για λόγους προστασίας των προσωπικών δεδομένων των χρηστών σας, προτείνεται να διατηρούνται logs για 7 μέρες. 
```xml
 <variable name ="idp.loghistory" value="7"/>
```
Καταγραφή Client IP address για την αντιμετώπιση επιθέσεων τύπου brute force
```xml
<appender name = "IDP_PROCESS" class = "...<File> ...
 
<rollingPolicy class = "...
<fileNamePattern> ...
<maxHistory > ...
</ rollingPolicy >
 
<encoder class = "...
<charset> ...
<Pattern>%date {ISO8601}  - %level [%logger:%line] - IP:%mdc{idp.remote_addr:-n/a}  - %msg%n%ex{short} </Pattern>
</ encoder >
</ appender >
</encoder>
```


## Παραμετροποίηση επικοινωνίας με LDAP/AD

Στο αρχείο ldap.properties θα πραγματοποιήσετε την απαιτούμενη παραμετροποίηση για να μπορέσει να
χρησιμοποιήσει τον LDAP server σας ο Identity Provider για την αυθεντικοποίηση των χρηστών και την ανάκτηση των στοιχείων τους (attributes)

/opt/shibboleth-idp/conf/ldap.properties

- idp.authn.LDAP.authenticator: Επιλέξτε ποιον τύπο Authenticator θα χρησιμοποιήσει Identity Provider. Οι επιλογές που έχετε για το πως θα γίνει η αυθεντικοποίηση είναι οι ακόλουθες:

- *anonSearchAuthenticator*: Ο Identity Provider Θα επιχειρήσει να κάνει anonymous search για το userDN και στη συνέχεια θα κάνει απόπειρα bind εκ μέρους του χρήστη με DN, password
 
- *bindSearchAuthenticator*: Ο Identity Provider θα επιχειρήσει να κάνει bind με ένα service account DN, password, και στην συνέχεια search για το user DN και απόπειρα bind με DN, password αυτή η μέθοδος χρησιμοποιείται και για AD
 
- *directAuthenticator*: Σε περιπτώσεις όπου το DN των χρήστων έχει γνωστο format πχ CN=user_name,ou=accounts,dc=domain,dc=edu, δηλαδή οι χρήστες είναι όλοι κάτω από το ίδιο OU, δεν γίνεται search αλλά απευθέιας bind
 
- *adAuthenticator* : Παραμετροποίηση που αφορά τη χρήση Active Directory.


Οι ακόλουθες τιμές παραμετροποίησης αφορούν πρόταση της Δ.Ο. της Ομοσπονδίας ΔΗΛΟΣ του ΕΔΥΤΕ για την χρήση bind search σε OpenLDAP server που υποστηρίζει STARTTLS.
- idp.authn.LDAP.authenticator : Ορίστε την τιμή *bindSearchAuthenticator*
- idp.authn.LDAP.ldapURL : Ορίστε την τιμή ως το ldap URL που αντιστοιχει στον host και τo port στο οποίο ακούει για συνδέσεις ο LDAP Server σας. (π.χ. ldap://ldap.grnet.gr:389)

- idp.authn.LDAP.useStartTLS : Ορίστε την τιμή σε *true*

- idp.authn.LDAP.useSSL : Ορίστε την τιμή σε *false*
 
- idp.authn.LDAP.connectTimeout : Ορίστε την τιμή σε *3000*
 
- idp.authn.LDAP.trustCertificates : Ορίστε την τιμή ώς το filepath του αρχείου που περιέχει το πιστοποιητικό που χρησιμοποιεί ο LDAP server σας για TLS σε pem format.
 
- idp.authn.LDAP.baseDN : Ορίστε το baseDN στον LDAP σας όπου θα πρέπει να συνδεθεί ο Identity Provider, π.χ. dc=grnet,dc=gr
 
- idp.authn.LDAP.subtreeSearch : Ορίστε την τιμή σε *true* αν θέλετε να κάνει subtree search στο baseDN σας ο Identity Provider για να βρεί τους χρήστες
 
- idp.authn.LDAP.userFilter : Ορίστε την τιμή ως το `ldap filter <https://tools.ietf.org/search/rfc4515>`_ το οποίο θα χρησιμοποιεί ο Identity Provider για να ψάχνει τους χρήστες σας στον LDAP Server. Το macro *{user}* αντιστοιχεί στο input του χρήστη στην φόρμα σύνδεσης, οπότε αν για παράδειγμα οι χρήστες σας κάνουν login χρησιμοποιώντας το uid τους στον LDAP, η τιμή του userFilter θα ήταν *(uid={user})*
 
- idp.attribute.resolver.LDAP.ldapURL : Ορίστε την τιμή ώς *%{idp.authn.LDAP.ldapURL}*
 
- idp.attribute.resolver.LDAP.baseDN : Ορίστε την τιμή ώς *%{idp.authn.LDAP.baseDN}*
 
- idp.attribute.resolver.LDAP.bindDN : Ορίστε την τιμή ώς *%{idp.authn.LDAP.bindDN}*
 
- idp.attribute.resolver.LDAP.bindDNCredential : Ορίστε την τιμή ώς *%{idp.authn.LDAP.bindDNCredential}*
 
- idp.attribute.resolver.LDAP.useStartTLS : Ορίστε την τιμή ώς *%{idp.authn.LDAP.useStartTLS:true}*
 
- idp.attribute.resolver.LDAP.trustCertificates : Ορίστε την τιμή ώς *%{idp.authn.LDAP.trustCertificates}*
 
- idp.attribute.resolver.LDAP.searchFilter : Ορίστε την τιμή ώς *(uid=$requestContext.principalName)*

Τέλος, για την διευκόλυνση σας, σας παραθέτουμε ένα παράδειγμα το οποίο μπορεί να εφαρμοστεί για OpenLDAP και AD 
```xml
# LDAP authentication (and possibly attribute resolver) configuration
# Note, this doesn't apply to the use of JAAS authentication via LDAP

## Authenticator strategy, either anonSearchAuthenticator, bindSearchAuthenticator, directAuthenticator, adAuthenticator
idp.authn.LDAP.authenticator                   = bindSearchAuthenticator

## Connection properties ##
idp.authn.LDAP.ldapURL                          = ldap://localhost ή ldaurl:389
idp.authn.LDAP.useStartTLS                     = false
# Time in milliseconds that connects will block
idp.authn.LDAP.connectTimeout                  = 3000
# Time in milliseconds to wait for responses
idp.authn.LDAP.responseTimeout                 = 3000
# Connection strategy to use when multiple URLs are supplied, either ACTIVE_PASSIVE, ROUND_ROBIN, RANDOM
#idp.authn.LDAP.connectionStrategy               = ACTIVE_PASSIVE

## SSL configuration, either jvmTrust, certificateTrust, or keyStoreTrust
#idp.authn.LDAP.sslConfig                       = certificateTrust
## If using certificateTrust above, set to the trusted certificate's path
#idp.authn.LDAP.trustCertificates                = %{idp.home}/credentials/ldap-server.crt
## If using keyStoreTrust above, set to the truststore path
#idp.authn.LDAP.trustStore                       = %{idp.home}/credentials/ldap-server.truststore

## Return attributes during authentication
idp.authn.LDAP.returnAttributes                 = passwordExpirationTime,loginGraceRemaining

## DN resolution properties ##

# Search DN resolution, used by anonSearchAuthenticator, bindSearchAuthenticator
# for AD: CN=Users,DC=example,DC=org
idp.authn.LDAP.baseDN                           = ou=People,dc=statusreporting,dc=vm,dc=grnet,dc=gr
#idp.authn.LDAP.subtreeSearch                   = false
idp.authn.LDAP.userFilter                       = (uid={user})
# bind search configuration
# for AD: idp.authn.LDAP.bindDN=adminuser@domain.com
idp.authn.LDAP.bindDN                           = cn=testidis,dc=statusreporting,dc=vm,dc=grnet,dc=gr

# Format DN resolution, used by directAuthenticator, adAuthenticator
# for AD use idp.authn.LDAP.dnFormat=%s@domain.com
#idp.authn.LDAP.dnFormat                         = uid=%s,ou=people,dc=example,dc=org

# pool passivator, either none, bind or anonymousBind
#idp.authn.LDAP.bindPoolPassivator                  = none

# LDAP attribute configuration, see attribute-resolver.xml
# Note, this likely won't apply to the use of legacy V2 resolver configurations
idp.attribute.resolver.LDAP.ldapURL             = %{idp.authn.LDAP.ldapURL}
idp.attribute.resolver.LDAP.connectTimeout      = %{idp.authn.LDAP.connectTimeout:PT3S}
idp.attribute.resolver.LDAP.responseTimeout     = %{idp.authn.LDAP.responseTimeout:PT3S}
idp.attribute.resolver.LDAP.connectionStrategy  = %{idp.authn.LDAP.connectionStrategy:ACTIVE_PASSIVE}
idp.attribute.resolver.LDAP.baseDN              = %{idp.authn.LDAP.baseDN}
idp.attribute.resolver.LDAP.bindDN              = %{idp.authn.LDAP.bindDN}
idp.attribute.resolver.LDAP.useStartTLS         = %{idp.authn.LDAP.useStartTLS:false}
idp.attribute.resolver.LDAP.trustCertificates   = %{idp.authn.LDAP.trustCertificates:false}
idp.attribute.resolver.LDAP.searchFilter        = (uid=$resolutionContext.principal)

# LDAP pool configuration, used for both authn and DN resolution
#idp.pool.LDAP.minSize                          = 3
#idp.pool.LDAP.maxSize                          = 10
#idp.pool.LDAP.validateOnCheckout               = false
#idp.pool.LDAP.validatePeriodically             = true
#idp.pool.LDAP.validatePeriod                   = PT5M
#idp.pool.LDAP.validateDN                       =
#idp.pool.LDAP.validateFilter                   = (objectClass=*)
#idp.pool.LDAP.prunePeriod                      = PT5M
#idp.pool.LDAP.idleTime                         = PT10M
#idp.pool.LDAP.blockWaitTime                    = PT3S
```


Αν επιλεχθεί bindSearchautehnticator όπως στο παράδειγμα μας, θα χρειαστεί να ορίσετε τον κωδικό πρόσβαση για το bind. 
Στην έκδοση 4 του Shibboleth Identity Provider, οι κωδικοί πρόσβαση αποθηκεύονται στο αρχείο
```
/opt/shibboleth-idp/credentials/secrets.properties
```
Προσθέστε την γράμμή 
```
idp.authn.LDAP.bindDNCredential              = topasswordtouCNpoykaneibind
```

## Παραμετροποίηση attribute-resolver.xml



Το αρχείο αυτό περιέχει τους ορισμούς των attributes που μπορεί να έχει διαθέσιμα ο Identity Provider από το αποθετήριο χρηστών

- Να προστεθούν οι απαραίτητοι ορισμοί για τα attributes του schac
σχήματος που χρησιμοποιείτε και υπάρχουν στον LDAP server σας.

- Να ενεργοποιηθεί το Example LDAP Connector, το οποίο και
χρησιμοποιεί παραμετροποίηση που έχει γίνει ήδη στα αρχεία
`idp.properties` και `ldap.properties`

Μια τυπική μορφή του αρχείου attribute-resolver.xml για έναν Identity Provider που συμμετέχει στην Ομοσπονδία ΔΗΛΟΣ του ΕΔΥΤΕ, παρουσιάζεται παρακάτω:


```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
This file is a rudimentary example. While it is semi-functional, it isn't very
interesting. It is here only as a starting point for your deployment process
to avoid any dependency on components like an LDAP directory.
 
Very few attribute definitions and data connectors are demonstrated, and the
data is derived statically from the logged-in username and a static example
connector.
 
The file(s) in the examples directory contain more examples that involve more
complex approaches. Deployers should refer to the documentation for a complete
list of possible components and their options.
-->
<AttributeResolver
        xmlns="urn:mace:shibboleth:2.0:resolver"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="urn:mace:shibboleth:2.0:resolver http://shibboleth.net/schema/idp/shibboleth-attribute-resolver.xsd">
 
 
    <!-- ========================================== -->
    <!--      Attribute Definitions                 -->
    <!-- ========================================== -->
 
    <!--
    The EPPN is the "standard" federated username in higher ed.
    For guidelines on the implementation of this attribute, refer
    to the Shibboleth and eduPerson documentation. Above all, do
    not expose a value for this attribute without considering the
    long term implications.
    -->
    <AttributeDefinition id="eduPersonPrincipalName" xsi:type="Scoped" scope="%{idp.scope}">
        <InputAttributeDefinition ref="uid" />
    </AttributeDefinition>
 
    <!--
    The uid is the closest thing to a "standard" LDAP attribute
    representing a local username, but you should generally *never*
    expose uid to federated services, as it is rarely globally unique.
    -->
    <AttributeDefinition id="uid" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="uid"/>
    </AttributeDefinition>
 
    <AttributeDefinition id="givenName" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="givenName"/>
    </AttributeDefinition>
 
    <AttributeDefinition id="mail" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="mail" />
    </AttributeDefinition>
 
    <AttributeDefinition id="mobile" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="mobile" />
    </AttributeDefinition>
 
    <AttributeDefinition id="sn" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="sn" />
    </AttributeDefinition>
 
    <AttributeDefinition id="cn" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="cn" />
    </AttributeDefinition>
 
    <AttributeDefinition id="o" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="o" />
    </AttributeDefinition>
 
    <AttributeDefinition id="telephoneNumber" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="telephoneNumber" />
    </AttributeDefinition>
 
    <AttributeDefinition id="displayName" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="displayName" />
    </AttributeDefinition>
 
    <AttributeDefinition id="eduPersonEntitlement" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="eduPersonEntitlement"/>
    </AttributeDefinition>
 
    <AttributeDefinition id="eduPersonAffiliaton" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="eduPersonAffiliation"/>
    </AttributeDefinition>
 
    <AttributeDefinition id="eduPersonPrimaryAffiliaton" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="eduPersonPrimaryAffiliation"/>
    </AttributeDefinition>
 
    <AttributeDefinition id="eduPersonAssurance" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="eduPersonAssurance"/>
    </AttributeDefinition>
 
    <AttributeDefinition id="schacUserStatus" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="schacUserStatus"/>
    </AttributeDefinition>
 
    <AttributeDefinition id="schacHomeOrganization" xsi:type="Simple">
        <InputDataConnector ref="myLDAP" attributeNames="schacHomeOrganization"/>
    </AttributeDefinition>
 
    <AttributeDefinition id = "schacPersonalUniqueCode" xsi:type ="Template">
      <InputDataConnector ref="myLDAP" attributeNames="schacPersonalUniqueCode" />
      <DisplayName xml:lang="en">European Student Identifier (ESI)</DisplayName>
      <Template>urn:schac:personalUniqueCode:int:esi:gr:${schacPersonalUniqueCode}</Template>
<!--      <AttributeEncoder xsi:type="SAML2String" name="urn:oid:1.3.6.1.4.1.25178.1.2.14" friendlyName="schacPersonalUniqueCode" encodeType="false" /> -->
    </AttributeDefinition>
    <!--
    In the rest of the world, the email address is the standard identifier,
    despite the problems with that practice. Consider making the EPPN
    value the same as your official email addresses whenever possible.
    -->
<!--
    <AttributeDefinition id="mail" xsi:type="Template">
        <InputAttributeDefinition ref="uid" />
        <Template>
          <![CDATA[
               ${uid}@statusreporting.vm.grnet.gr
          ]]>
        </Template>
    </AttributeDefinition>
-->
 
    <!--
    This is an example of an attribute sourced from a data connector.
    -->
    <AttributeDefinition id="eduPersonScopedAffiliation" xsi:type="Scoped" scope="%{idp.scope}">
        <InputDataConnector ref="myLDAP" attributeNames="affiliation" />
    </AttributeDefinition>
 
 
    <!-- ========================================== -->
    <!--      Data Connectors                       -->
    <!-- ========================================== -->
 
<!--
    <DataConnector id="staticAttributes" xsi:type="Static">
        <Attribute id="affiliation">
            <Value>member</Value>
        </Attribute>
    </DataConnector>
-->
    <DataConnector id="myLDAP" xsi:type="LDAPDirectory"
        ldapURL="%{idp.attribute.resolver.LDAP.ldapURL}"
        baseDN="%{idp.attribute.resolver.LDAP.baseDN}"
        useStartTLS="%{idp.attribute.resolver.LDAP.useStartTLS:true}"
        connectTimeout="%{idp.attribute.resolver.LDAP.connectTimeout}"
        responseTimeout="%{idp.attribute.resolver.LDAP.responseTimeout}">
        <FilterTemplate>
            <![CDATA[
                %{idp.attribute.resolver.LDAP.searchFilter}
            ]]>
        </FilterTemplate>
    </DataConnector>
 <!--
    <DataConnector id="StoredId"
        xsi:type="StoredId"
        generatedAttributeID="persistentId"
        salt="%{idp.persistentId.salt}">
        <InputAttributeDefinition ref="%{idp.persistentId.sourceAttribute}" />
        <BeanManagedConnection>shibboleth.MySQLDataSource</BeanManagedConnection>
    </DataConnector>
 -->
</AttributeResolver>
```
## Παραμετροποίηση απελευθέρωσης Attributes

Η παραμετροποίηση της απελευθέρωσης attributes γίνεται στο αρχείο 
```
/opt/shibboleth-idp/conf/attribute-filter.xml
```
με την μορφή AttributeFilterPolicy elements , όπου ορίζεται ποια attributes επιτρέπεται ή απαγορεύεται να απελευθερώνει ο Identity Provider σε
συγκεκριμένους Service Providers ή ομάδες Service Providers. 
Η παραμετροποίηση αυτή είναι καθαρά ευθύνη του διαχειριστή του κάθε Identity Provider και εξαρτάται από την Πολιτική Ασφαλείας του κάθε
Ιδρύματος. 
Οι προτάσεις της Δ.Ο. της Ομοσπονδίας ΔΗΛΟΣ του ΕΔΥΤΕ είναι: 
- Να λάβετε υπ'όψιν την ανάγκη προστασίας της ιδιωτικότητας των χρηστών σας
- Να λάβετε υπ'όψιν την ανάγκη των χρηστών σας να έχουν πρόσβαση σε υπηρεσίες που τους είναι απαραίτητες για να επιτελέσουν το
ακαδημαϊκό ή ερευνητικό τους έργο.
- Να ενεργοποιήσετε την λειτουργία User Consent για να είναι οι χρήστες σας οι τελικοί υπεύθυνοι για το αν θα επιτρέψουν την
απελευθέρωση των attributes τους προς οποιονδήποτε Service Provider. 

Oι Service Providers που λειτουργεί και διαχειρίζεται το ΕΔΥΤΕ, ανήκουν σε ένα EntityGroup μέσα στα Metadata της Ομοσπονδίας ΔΗΛΟΣ του
ΕΔΥΤΕ με GroupID *http://aai.grnet.gr/entities/grnet/* ώστε να μπορείτε να ρυθμίσετε παρόμοιες πολιτικές απελευθέρωσης για όλους 
Ακόμα, για κάθε υπηρεσία, τα attributes τα οποία είναι απαραίτητα για κάθε υπηρεσία, περιέχονται στα metadata κάθε υπηρεσίας μεσα στο
metadata aggregate της Ομοσπονδίας ΔΗΛΟΣ του ΕΔΥΤΕ. 

Για παράδειγμα η Υπηρεσία e:Presence με SAML EntityID το *https:/new.epresence.grnet.gr/shibboleth*, περιέχει στα metadata της το ακόλουθο: 
```
 <md:RequestedAttribute FriendlyName="givenName" Name="urn:oid:2.5.4.42" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="true"/>
 <md:RequestedAttribute FriendlyName="sn" Name="urn:oid:2.5.4.4" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="true"/>
 <md:RequestedAttribute FriendlyName="mail" Name="urn:oid:0.9.2342.19200300.100.1.3" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="false"/>
 <md:RequestedAttribute FriendlyName="telephoneNumber" Name="urn:oid:2.5.4.20" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="false"/>
 <md:RequestedAttribute FriendlyName="schacHomeOrganization" Name="urn:oid:1.3.6.1.4.1.25178.1.2.9" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="true"/>
```
To οποίο δηλώνει ότι η υπηρεσία για να λειτουργήσει, χρειάζεται από τον Identity Provider να απελευθερώσει για τον χρήστη τα 
```
givenName (όνομα)
sn (επώνυμο)
schacHomeOrganization
```
Ενώ επιθυμητά (αν και όχι αναγκαία) είναι και τα 
```
mail
telephoneNumber 
```
Μια τέτοια πολιτική θα μπορούσε να αποδοθεί στο ακόλουθο Attribute Filter Policy 
```
<AttributeFilterPolicy id="epresence">
  <PolicyRequirementRule xsi:type="Requester" value="https://new.epresence.grnet.gr/shibboleth" />
    <AttributeRule attributeID="email">
      <PermitValueRule xsi:type="ANY" />
    </AttributeRule>
    <AttributeRule attributeID="telephoneNumber">
      <PermitValueRule xsi:type="ANY" />
    </AttributeRule>
    <AttributeRule attributeID="schacHomeOrganization">
      <PermitValueRule xsi:type="ANY" />
    </AttributeRule>
    <AttributeRule attributeID="givenName">
      <PermitValueRule xsi:type="ANY" />
    </AttributeRule>
    <AttributeRule attributeID="surname">
      <PermitValueRule xsi:type="ANY" />
    </AttributeRule>
</AttributeFilterPolicy>
```
Σας δίνουμε και ένα παράδειγμα με προϋπόθεσης απελευθέρωσης αν ικανοποιείται κάποιο κριτήριο
```xml
<AttributeFilterPolicy id = "ESItoSelectedServices">
    <PolicyRequirementRule xsi:type="AND">
        <Rule xsi:type="OR">
            <Rule xsi:type="Requester" value="https://jtestservice/shibboleth" />
        </Rule>
        <!-- Αν είναι φοιτητής απελευθέρωσε το attribute -->
        <Rule xsi:type="Value" attributeID="eduPersonEntitlement" value="member" />
    </PolicyRequirementRule>
    <AttributeRule attributeID="schacPersonalUniqueCode">
    <!--πάρε το παραπάνω attribute από το IdM και πρόσθεσε κάποιο πρόθεμα πριν την απελευθέρωσή του-->
        <PermitValueRule xsi:type="ValueRegex" regex="^urn:schac:personalUniqueCode:int:esi:.*$" />
    </AttributeRule>
</AttributeFilterPolicy>
```


# Παραμετροποίηση Βάσης

## Εγκατάσταση βάσης δεδομένων
Εγκατάσταση βάσης δεδομένων και των απαραίτητων πακέτων για την επικοινωνία IdP και βάσης δεδομένων
```
root@idp:~# apt install mariadb-server mariadb-client libmariadb-java
```

```
root@idp:~# ln -s /usr/share/java/mariadb-java-client.jar /var/lib/tomcat9/lib/mariadb-java-client.jar
```

Επανεκίνηση tomcat για την ενεργοποίηση των νέων ρυθμίσεων:
```
root@idp:~# systemctl restart tomcat9
```

Συνδεθείτε στην βάση σαν χρήστης με τα κατάλληλα δικαιώματα, και στη
συνέχεια

Δημιουργείστε το instance της βάσης που θα χρησιμοποιήσει ο
Shibboleth Identity Provider
```
mysql> CREATE DATABASE shibboleth;
```
Δημιουργείστε ένα χρήστη με τα κατάλληλα credentials που θα
χρησιμοποιεί ο Shibboleth Identity Provider ώστε να διαβάζει και να
γράφει στην βάση. Αντικαταστήστε το *password* με έναν ισχυρό κωδικό
της επιλογής σας.
```
mysql> CREATE USER 'shibboleth'@'localhost' IDENTIFIED BY 'password';
```
Δώστε στον χρήστη που δημιουργήσατε τα κατάλληλα privileges
```
mysql> GRANT ALL PRIVILEGES ON shibboleth . * TO 'shibboleth'@'localhost';
mysql> FLUSH PRIVILEGES;
```
Δημιουργείστε τον πίνακα που θα χρησιμοποιηθεί για το persistent
SAML2 NameID, όπως ορίζεται παρακάτω
```
mysql> use shibboleth
mysql> CREATE TABLE shibpid (
localEntity VARCHAR(255) NOT NULL,
peerEntity VARCHAR(255) NOT NULL,
persistentId VARCHAR(50) NOT NULL,
principalName VARCHAR(50) NOT NULL,
localId VARCHAR(50) NOT NULL,
peerProvidedId VARCHAR(50) NULL,
creationDate TIMESTAMP NOT NULL,
deactivationDate TIMESTAMP NULL,
PRIMARY KEY (localEntity, peerEntity, persistentId)
);
```

Δημιουργείστε τον πίνακα που θα χρησιμοποιηθεί για το server side
storage του Shibboleth Identity Provider, όπως ορίζεται παρακάτω:
```
mysql> use shibboleth
mysql> CREATE TABLE StorageRecords (
context varchar(255) NOT NULL,
id varchar(255) NOT NULL,
expires bigint(20) DEFAULT NULL,
value longtext NOT NULL,
version bigint(20) NOT NULL,
PRIMARY KEY (context,id)
);
```

# Παραμετροποίηση JPA Server Side Storage
Στην ενότητα αυτή περιγράφεται η παραμετροποίηση της επικοινωνίας του Shibboleth Identity Provider με την βάση δεδομένων που χρησιμοποιείται για την αποθήκευση δεδομένων σχετικών με τα Sessions, τα NameIDs των χρηστών κ.α.

Στο  /opt/shibboleth-idp/conf/global.xml είναι απαραίτητη η προσθήκη beans για την επικοινωνία με την βάση δεδομένων
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd"
 
       default-init-method="initialize"
       default-destroy-method="destroy">

    <!-- Use this file to define any custom beans needed globally. -->

    <!-- The following values are default values:
         p:maxActive="100"
         p:maxIdle="100"
        It may be necessary to adjust these values depending on the load on your IdP, 
        as well as the configuration of your MySQL Servers. -->
        <bean id="shibboleth.MySQLDataSource"
              class="%{mysql.class}"
              p:driverClassName="org.mariadb.jdbc.Driver"
              p:url="%{mysql.url}"
              p:username="%{mysql.username}"
              p:password="%{mysql.password}"
              p:maxWait="15000"
              p:testOnBorrow="true"
              p:maxActive="100"
              p:maxIdle="100"
              p:validationQuery="select 1"
              p:validationQueryTimeout="5" />

        <bean id="shibboleth.JPAStorageService"
              class="org.opensaml.storage.impl.JPAStorageService"
              p:cleanupInterval="%{idp.storage.cleanupInterval:PT10M}"
              c:factory-ref="shibboleth.JPAStorageService.EntityManagerFactory" />

        <bean id="shibboleth.JPAStorageService.EntityManagerFactory"
              class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
            <property name="packagesToScan" value="org.opensaml.storage.impl"/>
            <property name="dataSource" ref="shibboleth.MySQLDataSource"/>
            <property name="jpaVendorAdapter" ref="shibboleth.JPAStorageService.JPAVendorAdapter"/>
            <property name="jpaDialect">
                <bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect" />
            </property>
        </bean>

        <bean id="shibboleth.JPAStorageService.JPAVendorAdapter"
              class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"
              p:generateDdl="true"
              p:database="MYSQL"
              p:databasePlatform="org.hibernate.dialect.MySQL5Dialect" />


</beans>
```

Αποθήκευση των διαπιστευτηρίων για την βάση δεδομένων. 

Στο αρχείο /opt/shibboleth-idp/conf/idp.properties όπως ορίσετε:
```
# ...
mysql.class    = org.apache.tomcat.jdbc.pool.DataSource
mysql.url      = jdbc:mysql://localhost:3306/shibboleth
mysql.username = shibboleth
# ...
```
Και στο αρχείο /credentials/secrets.properties
```
mysql.password = topassword που ορίσατε στο βήμα CREATE USER 'shibboleth'@'localhost' IDENTIFIED BY 'password'; της παραμετροποίησης της βάσης
```
Στην συνέχεια Επανεκκινήστε το Tomcat για να βεβαιωθείτε ότι μπορεί να διαβάσει το global.xml
```
root@idp:~# systemctl restart tomcat9
```
## Παραμετροποίηση PersistentId

Η παραμετροποίηση για την απελευθέρωση του persistentId γίνεται σε πολλά αρχεία. Παράγεται από κάποιο υπάρχον attribute του χρήστη και μοα τυχαία τιμή που χρησιμοποιείται σαν salt στο hashing function που υπολογίζει την τιμή του NameID.

Στο αρχείο opt/shibboleth-idp/conf/saml-nameid.properties θα ορίστεί το πηγαίο attribute και ότι θα αποθηκεύεται στην βάση δεδομένων.
Αν Επιλέξετε το π.χ. το  uid για πηγαίο attribute ορίστε το παρακάτω
```
idp.persistentId.sourceAttribute  = uid
# idp.persistentId.useUnfilteredAttributes = true
  
# To use a database, use shibboleth.StoredPersistentIdGenerator
idp.persistentId.generator  = shibboleth.StoredPersistentIdGenerator
# For basic use, set this to a JDBC DataSource bean name:
idp.persistentId.dataSource  = shibboleth.MySQLDataSource
```
Στο αρχείο /opt/shibboleth-idp/credentials/secrets.properties αποθηκεύστε το hash σας
```
idp.persistentId.salt  = my-very-very-long-hash
<!-- openssl rand -base64 64 π.χ. για την δημιουργία του-->
```
Στο αρχείο /opt/shibboleth-idp/conf/saml-nameid.xml θα ενεργοποιήσετε την παραγωγή του PersistentId. 
```xml
<!-- SAML 2 NameID Generation -->
<util:list id="shibboleth.SAML2NameIDGenerators">
 
    <ref bean="shibboleth.SAML2TransientGenerator" />
 
    <!-- Uncommenting this bean requires configuration in saml-nameid.properties. -->
    <ref bean="shibboleth.SAML2PersistentGenerator" />
 
    <!--
    <bean parent="shibboleth.SAML2AttributeSourcedGenerator"
        p:omitQualifiers="true"
        p:format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress"
        p:attributeSourceIds="#{ {'mail'} }" />
    -->
 
</util:list>
```
Στο αρχείο /opt/shibboleth-idp/conf/c14n/subject-c14n.xml ενεργοποιήστε το proccessing του persistentId
```xml
<!-- Handle a SAML 2 persistent ID, provided a stored strategy is in use. -->
<ref bean="c14n/SAML2Persistent" />
```
Τα περισσότερα IdM's επιτρέπουν στους χρήστες να ολοκληρώσουν την είσοδο τους είτε με πεζά είτε με κεφαλαία γραμματα στο username τους. Αυτό μπορεί να προκαλέσει προβλήμτα στην βάση δεδομένων του IdP(case sensitive).
Προτείνεται να οριστεί στο αρχείο /conf/c14n/subject-c14n.properties  κανόνας για το process μόνο πεζών  χαρακτήρων.
```
idp.c14n.simple.lowercase  = true
```
Ενεργοποίηση του Data Connector στο /conf/attribute-resolver.xml
```xml
<DataConnector id="StoredId"
    xsi:type="StoredId"
    generatedAttributeID="persistentId"
    salt="%{idp.persistentId.salt}">
    <InputAttributeDefinition ref="%{idp.persistentId.sourceAttribute}" />
    <BeanManagedConnection>shibboleth.MySQLDataSource</BeanManagedConnection>
</DataConnector>
```
# User consent και session information
ορίστε στο αρχείο /opt/shibboleth-idp/conf/idp.properties τα παρακάτω:
```
...
 # Set to "shibboleth.StorageService" for server-side storage of user sessions
idp.session.StorageService  = shibboleth.JPAStorageService
  
# Set to "shibboleth.StorageService" or custom bean for alternate storage of consent
idp.consent.StorageService  = shibboleth.JPAStorageService
  
# Set to "shibboleth.consent.AttributeConsentStorageKey" to use an attribute
# to key user consent storage records (and set the attribute name)
idp.consent.attribute-release.userStorageKey  = shibboleth.consent.PrincipalConsentStorageKey
idp.consent.attribute-release .userStorageKeyAttribute  = % { idp.persistentId.sourceAttribute }
idp.consent.terms-of-use.userStorageKey  = shibboleth.consent.PrincipalConsentStorageKey
idp.consent.terms-of-use.userStorageKeyd.Attribute  = % { idp } s.pers.Id.
  
# Flags controlling how built-in attribute consent feature operates
# idp.consent.allowDoNotRemember = true
# so that the user must consent at least once for each SP
# the option for global consent should be switched off:
idp.consent.allowGlobal  = false
#idp .consent.allowPerAttribute = false
  
# So that even with a new value of an attribute and with a new terms-of-use text, the
# user has to confirm again. Since a local DB is now available instead of browser cookies
, this useful feature should be activated!
idp.consent.compareValues  = true
```
Στο αρχείο /conf/relying-party.xml
```xml
<beans ...>
  <!-- ... -->
  <bean id="shibboleth.DefaultRelyingParty" parent="RelyingParty">
    <property name="profileConfigurations">
       <list>
         <!-- ... -->
         <bean parent="SAML2.SSO"
               p:postAuthenticationFlows="#{{'terms-of-use', 'attribute-release'}}"
               p:nameIDFormatPrecedence="#{{'urn:oasis:names:tc:SAML:2.0:nameid-format:persistent', 'urn:oasis:names:tc:SAML:2.0:nameid-format:transient'}}"/>
         <!-- ... -->
        </list>
    </property>
  </bean>
  <!-- ... -->
</beans>
```
Στην συνέχεια Επανεκκινήστε το Tomcat.
```
root@idp:/opt/shibboleth-idp# systemctl restart tomcat9
```

# Παραμετροποίηση Single Log Out

Για την ενρεγοποίηση του single log out (SLO), η απαιτούμενη παραμετροποίηση στο αρχείο *conf/idp.properties* είναι η ακόλουθη:
```
- idp.session.trackSPSessions = true

- idp.logout.elaboration = true

- idp.session.secondaryServiceIndex = true
```
# Παραμετροποίηση Λογότυπου φορέα
Μεταφορτώστε το λογότυπό σας (max. 240x180px) και τοποθετήστε το στο  
```
/opt/shibboleth-idp/edit-webapp/images/logo.png.
```
Δημιουργήστε και πάλι το IdP WAR (παρακολουθήστε τα αρχεία καταγραφών!):
```
root@idp:~# JAVA_HOME = /usr/opt/shibboleth-idp/bin/build.sh
```
