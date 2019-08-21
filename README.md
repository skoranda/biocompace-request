# Biocompace-Request

---
## Overview

>The African Centers of Excellence in Bioinformatics program leverages an existing African Centers of Excellence program in Mali and provide high-performance computing infrastructure and training on advanced biomedical data analysis to research institutes in Africa. It empowers African researchers to utilize computing resources and perform advanced biomedical data analysis, while creating an independent community of Bioinformatics specialists to encourage sharing of data, best practices and collaborative projects. 

>This repository was created to make a workflow for the Biocompace project. It creates a platform for researchers to create proposals to request software tools from Bioinformatics. Reviewers of the project can then approve or deny the proposal. At least two of the reviewers must approve for the proposal to continue in the workspace. The proposal is then moved onto the system administrator who can review the approved request before starting the project.

>This repository built upon the Imixs Workflow project. The Imixs project is an open source workflow engine based on the Java Enterprise Architecture (JEE). Imixs provides components to build human-centric workflow applications within a flexible framework. It controls business processes and distributes tasks within an organization as well as ensures that all tasks are processed in accordance to your Compliance Guidelines and Business Rules.


See [Imixs-Workflow](https://www.imixs.org/)

---
## 1. Build the Application
The Biocompace-request is based on Maven to build the project from sources run

```
$ mvn clean install
```

You can also download the application from the [latest release](https://github.com/ekoranda/biocompace-request/releases).

----
## 2. Deploy the Application

* **Install Java**

>You can get latest version of java from [here](https://www.java.com/en/download/mac_download.jsp).

* **Install WildFly 17.0.0.Final**

>You can get the link from [here](https://tomee.apache.org/download-ng.html). 

>Use wget to download the WildFly archive into the /tmp directory

```
$ wget https://download.jboss.org/wildfly/17.0.0.Final/wildfly-17.0.0.Final.tar.gz -P /tmp
```

>Unpack the tar.zg and move it to the /opt directory

```
sudo tar xf /tmp/wildfly-17.0.0.Final.tar.gz -C /opt/
```

* **Create a database using PostgreSQL**

>**1. Install From the CentOS Repositories**

```
sudo yum install postgresql-server postgresql-contrib
```

>**2. Initialize PostgreSQL**

```
sudo postgresql-setup initdb
sudo systemctl start postgresql
```

>**3. Change the password of the Linux user** *postgres*

```
sudo passwd postgres
su - postgres
psql -d template1 -c "ALTER USER postgres WITH PASSWORD 'newpassword';"
```

>**4. Create a role**

```
createuser examplerole --pwprompt
```

>You will be prompted to create a password for the new user

>**5. Create a database with the new user**

```
createdb mytestdb -O examplerole
```

>**6. Change the driver details in standalone.xml**

>Add the following driver in /opt/wildfly-17.0.0.Final/standalone/configuration/standalone.xml

```
<drivers>
...
	<driver name="postgres" module="org.postgres">
		<driver-class>org.postgresql.Driver</driver-class>
	</driver>
...
</drivers>
```

>**7. Change the datasource details in standalone.xml**

>Add the following datasource in /opt/wildfly-17.0.0.Final/standalone/configuration/standalone.xml with the database you created.

```
<datasources>
...
	<datasource jndi-name="java:/jdbc/biocompace" pool-name="biocompace">
     	<connection-url>jdbc:postgresql://127.0.0.1:5432/biocompace</connection-url>
		<driver>postgres</driver>
     	<security>
     		<user-name>username</user-name>
         	<password>password</password>
      	</security>
      </datasource>
	
...
</datasources>
```

>**8. Change the persistence details in persistence.xml in source of biocompace-request**

>Edit the following file: biocompace-request/src/main/resources/META-INF/persistence.xml

>Change the following line of code where *biocompace* is the name of your database: 

```
<jta-data-source>jdbc/biocompace</jta-data-source>
```

* **Create a user**

>**1. Create the file /opt/wildfly-17.0.0.Final/standalone/configuration/imixsrealm.properties**

>Add the following lines:

```
IMIXS-WORKFLOW-Reader=org.imixs.ACCESSLEVEL.READERACCESS
IMIXS-WORKFLOW-Author=org.imixs.ACCESSLEVEL.AUTHORACCESS
IMIXS-WORKFLOW-Editor=org.imixs.ACCESSLEVEL.EDITORACCESS
IMIXS-WORKFLOW-Manager=org.imixs.ACCESSLEVEL.MANAGERACCESS
```

>**2. Create the file /opt/wildfly-17.0.0.Final/standalone/configuration/sampleapp-users.properties**

>Add a username and password:

```
username:password
```

>**3. Create the file /opt/wildfly-17.0.0.Final/standalone/configuration/sampleapp-roles.properties**

>Add the followig line where **username** is the user you created:

```
username=IMIXS-WORKFLOW-Manager
```

* **Check that WildFly is working**

>Start WildFly with the following lines:

```
$ cd /opt/wildfly-17.0.0.Final
$ ./bin/standalone.sh -b=0.0.0.0 -bmanagement=0.0.0.0
```

>The WildFly console can be accessed at **http://localhost:8080/**

>Sign in using the username and passsword you added in sampleapp-users.properties.

* **Copy the biocompace-request.war file into /wildfly/standalone/deployments/**

>Stop Wildfly with Control+c

>Safe copy the war file into /tmp

>Copy the file into standalone/deployments/

```
$ cp /tmp/biocompace-request.war standalone/deployments/
```

* **Push the model up to the sever**

>Use the username and password created in sampleeapp-users.properties to curl the model 

```
$ cd biocompace-request/src/workflow
$ curl --user username:password --request POST -Tproposals.bpmn http://localhost:8080/api/model/bpmn
```

* **Deploy the Application**

>Start Wildfly

```
$ ./bin/standalone.sh -b=0.0.0.0 -bmanagement=0.0.0.0
```

>The application can be accessed at **http://localhost:8080/biocompace**

>Sign in using the username and passsword you added in sampleapp-users.properties.





----
## 3. Develop the Application
* **Install Maven**

>You can find the latest version from [here](https://maven.apache.org/download.cgi)

* **Download Eclipse IDE**

>Download Eclipse IDE from [here](https://www.eclipse.org/downloads/packages/release/2019-06/r/eclipse-ide-enterprise-java-developers)

* **Download Eclipse Modeling Tool**

>Download Eclipse Modeling tool from [here](https://www.eclipse.org/downloads/packages/release/2019-06/r/eclipse-modeling-tools)

* **Create a new webapp in Eclipse IDE using Maven**

```
$ mvn archetype:generate 
	-DgroupId={project-packaging}
	-DartifactId={project-name}
	-DarchetypeArtifactId={maven-template} 
	-DinteractiveMode=false
```

> Click file >import

> Click maven >existing maven projects

> Browse to find biocompace-request

>Create the new project

* **Edit the application in Eclipse IDE**

>Right click on pom.xml fild and select Run as >maven build

>Enter 'clean verify' in goals and press run

>Copy the war file into /tmp file 

```
$ scp biocompace-request/target/biocompace-request.war localhost:/tmp
```

>Copy the file into /standalone/depoloyments/

```
$ cp /tmp/biocompace-request.war standalone/deployments/
```

>Start Wildfly

```
$ cd /opt/wildfly-17.0.0.Final
$ ./bin/standalone.sh -b=0.0.0.0 -bmanagement=0.0.0.0
```

* **Edit the workflow using Eclipse-Modeling tool**

>Click on 'File >Open Projects From File System'

>Click on 'Import Source:' to browse for /biocompace-request/src/workflow/proposals.bpmn

>When ever you edit the model curl the model up to the server:

```
$ curl --user username:password --request POST -Tproposals.bpmn http://localhost:8080/api/model/bpmn
```

* **Add a new user to the application**

>Edit sampleapp-users.properties and add a new username and password

```
username:password
```

>Edit sampleappp-roles.properties and add a role to the user


```
username=IMIXS-WORKFLOW-Manager,IMIXS-WORKFLOW-Reader,IMIXS-WORKFLOW-Author,IMIXS-WORKFLOW-Editor
```

>Read [here](https://www.imixs.org/doc/engine/acl.html) to find information on the Imixs roles.

* **To find more information about how the Workflow engine works and how to edit a workitem read about Imixs [here](https://www.imixs.org/doc/index.html).**

----
## Roadmap

* Allow the user to reopen their proposal to edit it before it is opened by reviewer
* Create custom acl groups outside of the model for the workflow engine to map users to actor groups
* Add emails to users in order for mass emails to be sent out to an entire group instead of manually adding emails to model