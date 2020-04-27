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

Drag the <img width="22" height="22" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/ui_proc.png"> icon from the top menu bar to the square-patterned area and release it. The *Add Processor* prompt will appear. This is where the processor type is selected. Type “*ExecuteSQL*” to filter the list and double click on the **ExecuteSQL** entry.

The processor will appear on the flow. This type of processor executes a query on a database. Double click on the processor to configure it.
