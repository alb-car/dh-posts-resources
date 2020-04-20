# NiFi tutorial: Designing a flow for a real use case
This tutorial provides a step-by-step guide on how to use Apache NiFi for a simple, but realistic use case.

## Requirements
No prior experience with NiFi and its user interface is required, as every concept will be briefly introduced when necessary.
The only requirements are:
-	**An instance of Apache NiFi**. This tutorial is written with version 1.11 as basis, but should be valid for many prior versions.
-	**Access to a database**. This tutorial is written using Postgres as an example, but any similar database will be fine, as only simple features will be used. No knowledge of SQL is required.

## Scenario
We want to download a *GTFS* (standard format for data related to public transportation) file, perform minor adaptations to it, and store it into a database.

We will design a process with NiFi that creates the recipient table on a Postgres database, requests and downloads the file from its HTTP address, interprets and alters the data within, and finally stores it into the recipient table.

## Building the flow
The basic building brick in NiFi is a ***processor***, a unit that performs a single operation on data (retrieval, modification, routing, storing, etc.).

We will now proceed to build a chain of processors, called ***flow***, to cover our designed scenario.

Access your NiFi instance. Drag the <img width="22" height="22" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/ui_pg.png"> icon from the top menu bar to the square-patterned area, enter a name (for example, “Handle GTFS”) and click *ADD*.
<img align="right" width="280" height="131" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_pg.png">

A rectangle will appear in the square-patterned area: this is a ***process group***, which we will use to keep our flows neatly organized. You can think of it as equivalent to a folder on your computer’s file system.

Double click on the process group to enter it. The path on the bottom will change. This is where we will create the flow.
<img align="right" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-tutorial-gtfs/images/t_path.png">

### 1. Root processor: creating the recipient table
**Recipient tables are usually created separately, outside of NiFi**. Normally, NiFi’s role is to automatically retrieve, adapt and store data, while preparing the recipient system is a one-time ordeal handled manually or with a different tool.

We will include creation of the recipient table in the flow, to learn something more and ensure the table is compatible with this tutorial.

Drag the <img width="22" height="22" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/ui_proc.png"> icon from the top menu bar to the square-patterned area and release it. The *Add Processor* prompt will appear. This is where the processor type is selected. Type “*ExecuteSQL*” to filter the list and double click on the **ExecuteSQL** entry.

The processor will appear on the flow. This type of processor executes a query on a database. Double click on the processor to configure it.
