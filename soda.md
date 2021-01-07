# SODA in Autonomous Database

## **Introduction**

This workshop aims to help you understanding SODA

Estimated time: This lab takes approximately 20 minutes.

### About SODA in the Oracle Database

Simple Oracle Document Access (SODA) is a set of NoSQL-style APIs that let you create and store collections of documents (in particular JSON) in Oracle Database, retrieve them, and query them, without needing to know Structured Query Language (SQL) or how the documents are stored in the database.

There are separate SODA implementations for use with different languages (Java, Python, Node.js, PL/SQL) and with the representational state transfer (REST) architectural style. SODA for REST can itself be accessed from almost any programming language. It maps SODA operations to Uniform Resource Locator (URL) patterns.

### Prerequisites

This lab assumes you have completed the following labs:
* Lab: Provision and connect to Autonomous Database

## **Step 1**: Connect to ADB with SQL Developer Web
1.  Open up your SQL Developer Web worksheet, which is connected to your Autonomous Database, from your database OCI Console as you did in [Lab 1](?lab=lab-1-provision-connect-autonomous#STEP3:ConnecttoyourADBwithSQLDeveloperWeb). Sign in, if necessary; here, we are using the **ADMIN** user.

    ![](./images/ClearSDW.png " " )


    Please note that you <i>cannot</i> use your local (on-desktop) SQL Developer for this workshop. 
    However SQLcl can be used.

## **Step 2**: Create a collection

Create a collection named emp.

    <copy>
    soda create emp
    </copy>

![](./images/soda-create.png " " )

List the existing SODA collections, using command soda list.

    <copy>
    soda list
    </copy>

## **Step 2**: Insert JSON documents into the collection

Insert five JSON documents into the collection, one by one.

    <copy>
    soda insert emp {"name" : "Blake", "job" : "Intern", "salary" : 30000}
    soda insert emp {"name" : "Smith", "job" : "Programmer", "salary" : 80000}
    soda insert emp {"name" : "Miller", "job" : "Programmer", "salary" : 90000}
    soda insert emp {"name" : "Clark", "job" : "Manager", "salary" : 100000}
    soda insert emp {"name" : "King", "job" : "President", "salary" : 200000, "email" : "king@example.com"}
    soda insert emp {"name" : "Queen", "job" : "President", "salary" : 200000, "emailAddress" : "queen@example.org"}
    </copy>

Notice how with JSON each document can have is own schema. The last 2 entries of the collection have different schemas than the rest of the entries.

## **Step 3**: Query JSON documents from the collection

Count the number of documents in the collection

    <copy>
    soda count emp
    </copy>

Get (retrieve) documents, filtering the collection with a SODA query-by-example (QBE) pattern that matches "Miller" as the name. (Switch -f means list the documents that match the QBE.)

    <copy>
    soda get emp -f {"name":"Miller"}
    </copy>

 Get the documents that have a salary field whose value is at least 50,000. The QBE pattern uses SODA greater-than-or-equal operator, $ge, comparing target field salary, with the value 100,000.

    <copy>
    soda get emp -f {"salary" : {"$ge" : 100000}}
    </copy>

## **Step 3**: Modify JSON documents from the collection


Get the key ID for the employee named "Blake"

    <copy>
    soda get emp -f {"name" : "Blake"}
    </copy>

Capture the <b>key</b> value, e.g. "F19DDE2D3B6C4D2C876F1295BAA13537"

Update the document using the key

    <copy>
    soda replace emp  <put_the_key_here> {"name" : "Blake", "job" : "Junior Programmer", "salary" : 40000}
    </copy>

Delete the document


    <copy>
    soda delete emp -k <put_the_key_here> 
    </copy>

## **Step 4**: Full-text search in the collection

Create a search index 

    <copy>
    CREATE SEARCH INDEX idx ON emp(json_document) for JSON;
    </copy>


Do a full text search on the email for email containing the "@example.com" substring.


    <copy>
    soda get -f {"email": {"$contains":"@example.com"}}
    </copy>

Do a fuzzy search on the email

    <copy>
    soda get emp -f {"email": {"$contains":"fuzzy(qing)"}}
   </copy>

## **Step 4**: Access SODA collections with REST
Oracle REST Data Services (ORDS) makes it easy to develop REST interfaces for relational data in a JSON database. ORDS is a mid-tier Java application that maps HTTP(S) verbs, such as GET, POST, PUT, DELETE, and so on, to database transactions, and returns any results as JSON data.

The Oracle REST Data Services (ORDS) application in Autonomous JSON Database is preconfigured and fully managed.

<b>Get the URL for the service</b>

- On the Autonomous Database details page click Service Console.
- Click Development.
- The RESTful Services and SODA card shows the base URL.
- Click Copy URL to copy the URL.

SODA for REST is deployed in ORDS under the following URL pattern, where schema corresponds to a REST-enabled database schema.

    <copy>
    /ords/schema/soda/latest/*
    </copy>

<b>Query a document with REST</b>

In a shell window on your desktop, try querying the REST service.

You need to replace "URL" with the URL you got in the previous step.
    <copy>
    curl -X POST -u 'ADMIN:Pwd4testPwd4test#'  -H "Content-Type: application/json" --data '{"name":"Miller"}'  "https://<URL>/admin/soda/latest/emp?action=query"
    </copy>

<b>Insert a document with REST</b>
    <copy>
    curl -X POST -u 'ADMIN:Pwd4testPwd4test#' \
    -H "Content-Type: application/json" --data '{"name" : "Jackson", "job" : "Programmer", "salary" : 40000}' \
    "https://<URL>/admin/soda/latest/emp
    </copy>


## **Step 5**: Access SODA with SQL
Although SODA doesn't require any SQL knowledge, you can access and act directly on the backing-store tables that underlie SODA collections.

Check the "emp" table that backs the SODA "emp" collection

<copy>
desc EMP
select * from EMP
</copy>

JSON data is stored in Oracle's native binary format (aka OSON) in the <i>json_document<i> column.

Select each of the documents in the collection using the <i>json_serialize</i> function.

<copy>
SELECT json_serialize(json_document) FROM emp;
</copy>

Query the collection, projecting out the value of each of the fields from each document, as a SQL value.

<copy>
SELECT e.json_document.name,
       e.json_document.job,
       e.json_document.salary,
       e.json_document.email
  FROM emp e;
</copy>


