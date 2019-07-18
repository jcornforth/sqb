# Submit Quote Bind (SQB) process

## Process Model

This model takes you through the process of wholesale insurance submission, quotation and binding, which is a standard process in the London Insurance Market, as follows:

 - a submission is made by an insurance broker, who completes risk details such as Name and Country of Insured, Risk Class (Casualty, Property etc.), Inception Date, Period of Insurance and Sum Insured.  The broker can also upload an attached document containing additional risk details, which is accessible for the duration of the process and can be stored on a file system location of your choice;

 - following submission, DRL is executed for either auto-rejection or auto-rating the submission, based on a combination of Country, Risk Class and Sum Insured.  If the submission is acceptable but not auto-rated (i.e. no premium amount is assigned), it is referred to an underwriter who can either manually enter a premium quote, decline the submission with a reason - referred to as “Not Taken Up” (NTU) - or request more information from the broker.  The loop of requesting and supplying information can continue until the underwriter is satisfied;

 - once a quote has been entered, the broker has the opportunity to accept or reject the quote;

 - once the quote is accepted, the underwriter can bind indicating that the customer is now on risk and the SQB process is completed;

 - email notification is sent to broker and underwriter at each stage where human task intervention is required;

 - all form data captured as part of the process is persisted to a database for downstream integration / reporting;

 - the current rules for auto-rejection / auto-rating are simple and should be extended for practical business usage.  They could also be replaced with a web service call to a rating engine, where available.

## Project Contents

The contents of the project are as follows:

 - a BPMN2 process model.  Due to the use of Human Task Notifications, it is recommended that the model be edited with the legacy process designer;

 - a data model with a single persistabl data object - Submission;

 - 2 DRL files containing auto-rejection and auto-rating rules;

 - a process form for creating a new submission;

 - task forms for each subsequent human task of the process: quote, provide additional info, accept quote and bind.

All forms are auto-generated in PAM and are based on a "task form with nested subform" design, where the task form contains a nested subform bound to the Submission data object, plus a Document field.  Each subform contains the relevant fields bound to individual data attributes.

Submission and document process variables are passed into and out of human tasks via Data Assignments, and a dedicated script task exists to populate additional process variables used in the body of email notifications.

## Project Prerequisites

The project requires the following PAM setup / dependencies:

 - installation of PAM 7.3 on EAP 7.2

 - at least two application users: one member of the 'brokers' group, and one member of the 'underwriters' group.  The users and groups can be added to application-users.properties and application-roles.properties simultaneously using the add-user.sh command (sample files provided)

 - a MySQL database with corresponding driver deployed and datasource configured in standalone-full.xml (sample file provided)

 - the following system properties defined in standalone-full.xml:
```
	<property name="org.kie.server.persistence.ds" value="java:jboss/datasources/PAMDS"/>
	<property name="org.kie.server.persistence.tm" value="org.hibernate.service.jta.platform.internal.JBossStandAloneJtaPlatform"/>
	<property name="org.kie.server.persistence.dialect" value="org.hibernate.dialect.MySQL5InnoDBDialect"/>
	<property name="org.kie.server.persistence.schema" value="hibernate.default_schema"/>
```

For email notifications to work properly, the following additional dependencies are required:

 - an SMTP server for sending task notification emails: the sample EAP configuraiton uses localhost:25 as the mail socket binding.  Mails are only sent if a task is not actioned after 5 minutes

 - a userinfo.properties file containing email addresses for notify_broker, notify_underwriter and Administrator users: this file should be placed in the WEB-INF/classes directory of kie-server.war (sample file provided)

 - ensure the standalone-full.xml includes the following system properties:
```
	<property name="org.kie.mail.session" value="java:jboss/mail/Default"/>
```

 - (optional) to specify a particular location for document storage, add the following system property:
```
	<property name="org.jbpm.document.storage" value="/path/to/documents"/>
```

## Usage Instructions

You can use this model in the following ways:

1. Check out the source code to your own PAM installation, then build and deploy the project.
 - ensure the necessary pre-requisites (app users and database settings) are in place
 - in Business Central, select Menu, Projects, Import Project
 - enter the git URL of this repo e.g. https://github.com/jcornforth/sqb.git
 - click Import
 - select SubmitQuoteBind
 - click Ok
 - the project assets should appear and you can now click Deploy to build and deploy the project
 - select Menu, Process Definitions - the SQB process definition should be available to start


2. Download the built JAR file and deploy it to your own kie-server (runtime only).
 - ensure the necessary pre-requisites (app users and database settings) are in place
 - in Business Central, click on the Settings gear:
 - select Artifacts, Upload, browse for JAR file & upload - should see a POM and a JAR
 - select Menu, Execution Servers, Add Deployment Unit:
 - click Select to fill in details from the JAR
 - click Finish
 - click Start to start kie-server
 - select Menu, Process Definitions - the SQB process definition should be available to start
