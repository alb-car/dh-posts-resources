# Apache NiFi - What it is and how to use it

This article provides a guide for beginners to Apache NiFi, explaining what the software is about and guiding the user through its features and interface.

## Introduction

### What is NiFi for?

*Apache NiFi* is a tool to process large amounts of data in an automated and scalable way, by building a chain of operations to apply on such data, modifying and adapting it depending on its contents.

<img align="right" width="400" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/role.png">

Here are some sample use cases:
- Execute different operations on data based on contents.
-	Periodically process new data to extract useful information and start processes that use the results.
- Adapt data coming from different sources, with the same contents but in different formats, to the same schema.

### What is working with NiFi like?

The NiFi user builds a ***flow***, which consists in a sequence of operations data is subject to.

The *flow* is built by adding ***processors*** and connecting them together. Each *processor* performs a specific operation, depending on its type.

Among many others, there are processors designed to:
-	Perform an HTTP request (for example to download a file, or do an API call)
-	Convert a file from a type to another
-	Execute a query on a database
-	Decide what path to route data to depending on its contents

When a processor is added, some configuration is necessary: a processor that executes a query on a database will need to know what database to connect to and the credentials to use.

<img align="right" width="400" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/flow.png">

The image on the side illustrates an abstraction of a flow to download data from a URL and save it as a Comma-Separated Values (*CSV*) file.

Each rectangle is a **processor** and each arrow connects a pair of processors through a **relationship**, deciding which path data should follow depending on the results of the processor’s operation.

It is possible to decide what to do when an operation fails (invalid address, file cannot be converted into CSV, etc.), but this has been omitted from the example for the sake of simplicity.

## Using NiFi

This section explains NiFi’s UI and guides you through its main features. **You do not need to worry about saving your work while working with NiFi**: every action you perform will automatically save the current state of the entire flow.

Images used in this chapter refer to version 1.11 of NiFi. Previous versions may have slight differences, but they will not affect the features described here, except for the [Parameters/variables](#parameters-and-variables) section, which will explain how to handle differences depending on your version.

### Main UI

The main screen displays a large, square-patterned area, which is where processors will be placed.

<img align="right" width="400" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/ui.png">

**On the top are two toolbars**: the one above is to add elements to the flow, while the lower one gives an overview of the flow’s status.

**On the left are two small menus**: *Navigate* offers a navigable map of the flow; *Operate* contains buttons to change the state of processors, plus some additional features.

*Navigate* and *Operate may be reduced by clicking on <img width="20" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/min.png">, as most actions they involve may be performed by interacting directly on the flow.

Add a ***process group*** by dragging the <img width="25" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/ui_pg.png"> icon to the square-patterned area, input “*A quick flow test*” as name and click *ADD*.

For the sake of this guide, **process groups** may be considered equivalent to directories in your computer: just like you wouldn’t want to clutter your *home* directory with loads of unrelated files, and prefer to organize them in a tree-like structure made of directories, you’d rather organize your flows in NiFi with a similar tree-like structure, made of *process groups*.

<img align="left" width="200" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/process_group.png">

A rectangle will appear on the flow, with the name you chose and some numbers that will indicate the status of processors within.

**Enter the process group by double-clicking it**.

<img align="right" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/path.png">

The flow will appear empty again, but you can see that the path at the bottom has changed.

You could navigate back to the main flow by clicking on *NiFi Flow*, but don’t do that yet.

Before adding any processors, let’s add some ***parameters* or *variables***, which are useful for any value that may need to be used multiple times in your flow.

### Parameters and variables

There are some values that you may need multiple times in your flow, like the **root address of a server** that offers many API end-points that you need to send requests to, or the **name of a database schema** that contains many tables you need to interact with.

If this value changes, you would rather update it only in 1 place, instead of searching for all processors that use that value.

**Parameters/variables serve this purpose and they are very similar**: this redundancy is because parameters were added later (version *1.10*) to include new features, that required a different design.

The current version of NiFi supports both parameters and variables, but **variables may be dropped** at some point.

For the sake of helping users working with previous versions, this section explains how to use both parameters and variables, but, if possible, it’s recommended to use parameters.

The only difference important for this tutorial is:
-	**A variable is assigned to a process group** and can be used by that process group and all sub-process groups contained within.
-	**Parameters are grouped into *parameter contexts***, which is essentially a list of parameters. **A parameter context is independent of process groups**, so it may be assigned to different, unrelated process groups. However, a process group may only have access to 1 parameter context: sub-process groups cannot see the parameters of their parent process group, unless the same context is explicitly assigned to the sub-process group as well.

Let’s add a parameter/variable containing the name of some food you like. There is no need to add both a parameter and a variable, so only add a variable if your version does not support parameters (versions < *1.10*).

#### Adding a parameter

<img align="right" width="150" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/pc_settings.png">

Click on the menu button in the top right (<img width="25" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/button_menu.png">) and select “*Parameter Contexts*”. Click on <img width="20" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/button_plus.png"> in the top right to start adding a new parameter context.

In the *SETTINGS* tab, pick a name for the context (description is optional), then switch to the *PARAMETERS* tab and click on the <img width="20" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/button_plus.png"> button there.

<img align="left" width="240" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/pc_add_param.png">

*Name* will be how you refer to this parameter, while the important field is *Value*, which is the value NiFi will use whenever it encounters this parameter.

*Sensitive Value* is for things like passwords, to avoid displaying them as plain text.

Click *APPLY* to add the parameter, then click *APPLY* again to complete the creation of the parameter context.

The new context will now be listed in the *NiFi Parameter Contexts* menu, which you may close with <img width="20" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/button_x.png">.

<img align="right" width="200" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/pg_general.png">

Finally, assign the parameter context to the current process group: right click in the square-patterned area and click *Configure*.

In the *GENERAL* tab, select the context you created under the “*Process Group Parameter Context*” option, and click *APPLY*.

You are now ready to add a processor that uses this parameter. You can skip the [Adding a variable](#adding-a-variable) subsection, as it is equivalent to what you just did.

#### Adding a variable

Right-click on the square-patterned area and click *Variables*. Click <img width="20" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/button_plus.png"> in the menu that appears to start adding a new variable.

<img align="right" width="400" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/menu_variables.png">

Input the *Variable Name*, which is how you will refer to this variable, click *OK*, then input its value, which is the value NiFi will use whenever it encounters this variable, and click *OK*.

Click *APPLY* to finish adding variables.

You are now ready to add a processor that uses this variable.

### Configuring a processor

We will add and configure a processor that simply generates a ***flowfile*** containing some text. A **flowfile** is simply the name of data as it travels through the flow: it will not be saved on your computer, since we won’t add any processors to store it.

<img align="right" width="360" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/menu_add_proc.png">

The type of this processor is called *GenerateFlowFile*: it is generally useful for testing or hard-coding some values.

Drag the <img width="25" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/ui_proc.png"> icon into the square-patterned area to trigger the *Add Processor* prompt.

Since we already know the name of the processor, we can type it to filter it.

When you’re looking for a certain feature, but do not know the type’s name, you can try filtering by *tag*, like “*json*” or “*database*”.

The tag system is, unfortunately, not completely functional: **don’t type more than 1 tag**, or it may not list processors even if they do contain all the tags you listed, just because it expected them in a different order.

<img align="right" width="230" src="https://github.com/alb-car/dh-posts-resources/blob/master/nifi-beginner-guide/images/ui_proc_on_flow.png">

Double-click the desired type and the processor will appear in the square-patterned area. Double-click it to configure it.

There are 4 tabs: *SETTINGS*, *SCHEDULING*, *PROPERTIES* and *COMMENTS*. These 4 tabs are present no matter the type of the processor, but the contents of the *PROPERTIES* tab may vary significantly.
