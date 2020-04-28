# NiFi tutorial: Designing a flow for a real use case
This tutorial provides a step-by-step guide on how to use Apache NiFi for a simple, but realistic use case.

## Requirements
No prior experience with NiFi and its user interface is required, as every concept will be briefly introduced when necessary.
The only requirements are:
-	**An instance of Apache NiFi**. This tutorial is written with version 1.11 as basis, but should be valid for many prior versions.
-	**Access to a database**. This tutorial is written using Postgres as an example, but any similar database will be fine, as only simple features will be used. No knowledge of SQL is required.

## Scenario
We will download a *GTFS* (standard format for data related to public transportation) file of the Italian city of Trento, perform minor adaptations to it, and store it into a database.

We will design a process with NiFi that creates the recipient table on a Postgres database, requests and downloads the file from its HTTP address, interprets and alters the data within, and finally stores it into the recipient table.

GTFS data usually comes as a zipped folder, containing multiple text files with the same format but different contents (data on routes, stops, etc.). For simplicity, we will focus on the **routes.txt** file, which contains general information on the routes provided by public transportation lines.

## Building the flow
The basic building brick in NiFi is a ***processor***, a unit that performs a single operation on data (retrieval, modification, routing, storing, etc.).

We will build a chain of processors, called ***flow***, to cover our designed scenario.

Access your NiFi instance. Drag the <img width="22" height="22" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/ui_pg.png"> icon from the top menu bar to the square-patterned area, enter a name (for example, “Handle GTFS”) and click *ADD*.
<img align="right" width="280" height="131" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_pg.png">

A rectangle will appear in the square-patterned area: this is a ***process group***, which we will use to keep our flows neatly organized. You can think of it as equivalent to a folder on your computer’s file system.

Double click on the process group to enter it. The path on the bottom will change. This is where we will create the flow.

Before adding processors, we will create some parameters/variables.
<img align="right" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_path.png">

### Parameters/Variables
Parameters and variables are handy to keep multiple values configured in a single place. The flow will refer to these parameters/variables, so if we decide to change a value, we only have to update the parameter/variable once and every occurrence of it will use its new value.

#### Parameters
<img align="right" width="200" height="233" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/pc_config.png">

**Parameters** were introduced in **version 1.10**, with the intent of replacing variables. Users with versions >=1.10 should use parameters, while **users of previous versions should skip to the next paragraph** to add variables.

Right-click in the square-patterned area and click *Configure*. Switch to the *GENERAL* tab, expand *Process Group Parameter Context* and click *Create new parameter context...*.

The *Add Parameter Context* prompt will appear, with the *SETTINGS* tab selected. Enter any *Name* (for example, ***Public Transport***), then switch to the *PARAMETERS* tab. Click on <img width="22" height="22" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/button_plus.png"> and the *Add Parameter* prompt will show up.

<img align="right" width="260" height="143" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/pc_parameters.png">

Type ***schema*** for *Name* and ***trento*** for *Value*. *Sensitive Value* indicates whether the value should be hidden, while *Description* is purely for convenience. There is no need to change these two properties.

Click *APPLY*, then click on <img width="22" height="22" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/button_plus.png"> again and add another parameter named ***routes_table*** with value ***lines***.

The two parameters' values will be used with Postgres, which tends to force names to lower-case. It is possible to use upper-case characters, but to keep things simple, make sure both ***trento*** and ***lines*** are lower-case.

Click **APPLY** until NiFi says "*Process group configuration successfully saved.*", then click *OK* and close the *Handle GTFS Configuration* prompt. Skip the *Variables* paragraph and we will start adding processors to the flow.

#### Variables
Right-click in the square-patterned area and click **Variables**. The *Variables* prompt will appear.

Click on <img width="22" height="22" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/button_plus.png"> and the *New Variable* prompt will appear. Insert ***schema*** for *Variable Name*, click *OK* and insert ***trento*** in the white box that appears, then click *OK*.

Click on <img width="22" height="22" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/button_plus.png"> again and add another variable, named ***routes_table***, with value ***lines***.

The two parameters' values will be used with Postgres, which tends to force names to lower-case. It is possible to use upper-case characters, but to keep things simple, make sure both ***trento*** and ***lines*** are lower-case.

Click *APPLY* and, once NiFi is done updating changes, click *CLOSE*. We are ready to start adding processors to the flow.

### 1. Root processor: creating the recipient table
**Recipient tables are usually created separately, outside of NiFi**. Normally, NiFi’s role is to automatically retrieve, adapt and store data, while preparing the recipient system is a one-time ordeal handled manually or with a different tool.

We will include creation of the recipient table in the flow, to learn something more and ensure the table is compatible with this tutorial.

<img align="right" width="300" height="201" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_1_add.png">

Drag the <img width="22" height="22" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/ui_proc.png"> icon from the top menu bar to the square-patterned area and release it. The *Add Processor* prompt will appear. This is where the processor type is selected. Type “*executesql*” to filter the list and double click on the **ExecuteSQL** entry.

The processor will appear on the flow. This type of processor executes a query on a database.

Double click on the processor to configure it.

#### SETTINGS

<img align="right" width="300" height="100" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_1_settings.png">

In the ***SETTINGS*** tab, put a check mark on ***failure***, under *Automatically Terminate Relationships*, on the right side. This means that, if the operation fails, the processor should not forward the data to any of the processors we will add later. You may also change its *Name* if you'd like to.

#### SCHEDULING

In the ***SCHEDULING*** tab, insert ***1 day*** under *Run Schedule*. The processor would execute once every 24 hours, starting from the moment we will decide to run it. Once the flow is complete and its execution finished, we will stop all processors, so we will not actually let it execute daily.

<img align="right" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_1_scheduling.png">

**It is however important to set it to 1 day**: root processors are responsible for providing data to the flow, and with the default value, *0 sec*, NiFi would attempt to execute this processor as many times as possible. While it wouldn't cause any damage with this tutorial, **imagine if the first processor were to query a pay-per-use API: forgetting to set this value properly may cause thousands of useless queries, in just a few seconds, that give the same result but cost a lot of money**.

*0 sec* on any non-root processor simply means that the processor should execute as soon as upstream data is available, so we only need to worry about this property for the first processor.

#### PROPERTIES

The ***PROPERTIES*** tab is the most unique, where configuration differs depending on processor type. **Bold** properties are required, while non-bold ones are optional. Most properties have a default value anyway, so we will only change a few of them.

<img align="right" width="260" height="232" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_1_add_cs.png">

Next to *Database Connection Pooling Service*, click on *No value set* to display a drop-down menu: drop down the selection and pick *Create new service...*. The *Add Controller Service* prompt will be displayed. We will use the default controller service, so click CREATE.

You will notice the property now has the value *DBCPConnectionPool*. We need to configure this controller, so click on the arrow on the right.

<img width="500" height="41" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_1_property_set.png">

***Controller services*** offer a service that may be used by different processors. This one offers a connection to a database. We will also later use this same controller service for another processor, but we only need to configure it once.

Click on <img src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/ui_gear.png">, on the far right, to open the configuration panel and switch to the *PROPERTIES* tab.

The *Database Connection URL* format depends on the database you plan on using for storing the data. In this tutorial, we will use Postgres, so the format is as follows:
```
jdbc:postgresql://<host>:<port>/<database_name>
```
For example, if the database is hosted locally, at default port *5432*, and the database is called *public_transport*:
```
jdbc:postgresql://localhost:5432/public_transport
```

<img align="right" width="400" height="189" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_1_config_cs.png">

For *Database Driver Class Name* enter `org.postgresql.Driver`, while for *Database Driver Location(s)* enter `https://jdbc.postgresql.org/download/postgresql-42.2.7.jar`.

Values for *Database User* and *Password* depend on how your Postgres user is configured. If you set up a fresh Postgres installation for the tutorial, user and password are both `postgres` by default. You will notice NiFi later hides the value for *Password*.

Click *APPLY* and a <img src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/ui_enable.png"> icon will appear: click it and click *ENABLE*. Close all prompts and double-click on the processor again so that we can finish configuring it.

All that's left to do is to set the query to execute. Copy and paste the following in *SQL select query* (if you're using **variables, replace # occurrences with $**), then click *OK*:
```
CREATE SCHEMA IF NOT EXISTS #{schema};
CREATE TABLE IF NOT EXISTS #{schema}.#{routes_table} (
  line_id varchar,
  type varchar,
  full_name varchar,
  shortened varchar
);
```

<img align="right" width="400" height="176" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_1_configured.png">

The command above will create a schema and a table. As mentioned before, we will focus on general information about public transportation lines. The table has 4 fields:
- **line_id** - Short identifier for public transport lines, may contain letters. For buses, it's often referred to as *bus number*.
- **type** - Bus, tram, subway, etc.
- **full_name** - The full name of the line, usually comprised of the names of starting and ending stops.
- **shortened** - A convenience name, which contains both the line ID and a shortened version of the name.

<img align="left" width="240" height="90" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_1_flow.png">

Click *APPLY*. The processor is fully configured, but still displays a <img src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/icon_invalid.png"> icon: we need to direct *successful* output of this root processor to a new processor.

### 2. Obtaining the data with a HTTP request
