# Data validation

By combining the features of pre-existing validation and profiling libraries, as well as developing new software for additional purposes, we have created a platform to aid the whole process of data validation, from the initial step of acquiring the data, to archiving and inspecting the results.

This article teaches the reader how to use the tools involved.

## Table of contents

* [Introduction](#introduction)
  * [Scenario](#scenario)
  * [Structure](#structure)
* [Requirements](#requirements)
  * [Tools](#tools)
  * [Data](#data)

## Introduction

This section describes the common scenario for data validation and provides a brief overview of the tools we will use.

### Scenario

Before processing data, we generally assume that each record has a certain structure and certain properties hold true for its contents.
For example, with tabular data, we expect entries to have a specific number of fields and each field to respect some rules: cannot be null, must be unique, must be a number, must follow a certain format...

Whether it's due to a bug, an unforeseen edge case, a conversion between formats, or something inserted by hand, it is not rare to have invalid values. When processed, these may halt execution, propagate errors, or generate misleading statistics.

Data validation aims to find these irregularities before they become problems. This means that, along with the data, another document, called *schema*, must be provided, listing what properties must be verified.

### Structure

The platform we have developed is built to combine aspects of data validation and profiling, offered by pre-existing libraries, together with new features for storage and presentation of the results, to ease their inspection.

The following components form its structure. We have developed the latter three and this post will focus on the use of Datajudge and the UI.
* Storage for the data to be validated and its schema (S3, Azure, FTP, ...).
* Datajudge: a *Python* library which retrieves the data, combines the usage of pre-existing libraries for validation (currently supports [Frictionless](https://framework.frictionlessdata.io/)) and profiling (currently supports [pandas-profiling](https://github.com/pandas-profiling/pandas-profiling)) and generates a number of documents describing results and run environment.
* Back-end: using a MongoDB instance for storage, it provides *Spring* REST APIs for CRUD operations on the documents generated.
* UI: developed with *React-Admin*, to inspect the validation documents.

## Requirements

### Tools

Each component is needed: either installed and configured locally, or through a remote instance.

* **Datajudge**, which requires **Python >= 3.7**.
* An instance of the **back-end**, as well as a **MongoDB** instance to store documents on.
* An instance of the **UI**.

An instance of [AAC](https://github.com/scc-digitalhub/AAC) is needed to act as identity provider for the components.

### Data

Obviously some data and its schema must be available, either locally or on the cloud. You can download [this sample data]() and [its schema]() to follow the tutorial.