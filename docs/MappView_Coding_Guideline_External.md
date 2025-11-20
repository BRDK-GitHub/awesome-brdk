**BRDK MappView**

Programming guidelines

# Requirements

Development tool: Automation Studio 6.0 or newer

# Table of Contents

1. [Motivation](#motivation)
2. [Architecture](#architecture)
    2.1. [Folder structure in logical view](#folder-structure-in-logical-view)
    2.2. [Folder structure in configuration view](#folder-structure-in-configuration-view)
3. [Naming convention](#naming-convention)
    3.1. [Pages](#pages)
    3.2. [Widgets](#widgets)
    3.3. [Bindings](#bindings)
4. [Features](#features)
    4.1. [Snippets](#snippets)
    4.2. [Expressions](#expressions)
    4.3. [Session variables](#session-variables)
    4.4. [Event scripts](#event-scripts)
    4.5. [Dialogs and messages](#dialogs-and-messages)
5. [Widgets](#widgets-1)
    5.1. [Preferred widgets](#preferred-widgets)
    5.2. [Not recommended widgets](#not-recommended-widgets)
    5.3. [Widget class](#widget-class)
6. [Theme](#theme)
7. [Security options](#security-options)

# Motivation

The purpose of this document is to describe the architecture of MappView projects in BRDK, as well as defining fixed rules for naming conventions, structures, etc.

This is to ensure that all projects made by BRDK follow the same guidelines, so that all code can live to the same high standards and making it is easier for engineers to switch between projects.

# Architecture

It is important that the project is structured in a logical way that adhere to the following standards to improve the user experience for the developers both new and original.

## Folder structure in logical view

The MappView folder should be structured as follows:

**Figure 1:** Folder structure in Logical View

```text
Logical View
└── MappView
    ├── Resources
    │   ├── Snippets
    │   └── Texts
    └── Visualization
        ├── Common
        │   ├── Header.content
        │   └── Footer.content
        ├── PageName
        │   ├── PageName.page
        │   ├── PageName.content
        │   └── PageName.tmx
        └── DialogName
            ├── DialogName.dialog
            ├── DialogName.content
            └── DialogName.tmx
```

* There should always be a folder for common contents such as header, footer, common buttons etc.
* Each page should have its own folder with its page file, content or contents and a text file if texts are used.
* Each dialog should have its own folder with its dialog file, content or contents and text file.
* Snippets should always be in the resources folder inside the visualization folder.

## Folder structure in configuration view

The MappView folder should be structured as follows:

**Figure 2:** Folder structure in Configuration View

```text
Configuration View
└── MappView
    └── Visualization
        ├── Common
        ├── PageName
        │   ├── PageName.binding
        │   ├── PageName.eventbinding
        │   └── PageName.eventscript
        └── DialogName
            ├── DialogName.binding
            └── DialogName.eventbinding
```

* The folder structure in configuration view should directly reflect the structure seen in logical view.
* Every page should have its own folder where all binding, event bindings and event scripts should be located.

# Naming convention

It is important that all projects follow the same naming convention to increase reusability and improve navigation across projects.

## Pages

Pages should always be named with a name describing what is on that page. If a page has multiple contents they must all use the page name as prefix. Same goes for folders such as common see (Figure 1).

## Widgets

Widgets must be named for easier debugging in binding files. Widgets must always have a suffix in their names describing the type of widgets they are. Example buttons are called someBt, numeric outputs are called someNumOut etc.

## Bindings

Bindings will automatically have the same name as the content they are bound to. These will automatically update if content is renamed.

# Features

MappView comes with an array of build-in features to help the developer make a modern and useful HMI.

## Snippets

Snippets can be necessary to use but should be kept to a minimum since they make debugging difficult. They can be used to dynamically update texts or to embed an OPC-UA variable into a text. If event scripts are used in the projects most applications of snippets should be solved with these.

## Expressions

Expressions are sessions based and run on client-side. They are used for small changes to variables. Typically, things like scaling or formatting. They should be avoided since these can easily be solved in the PLC or with event scripts. This is because expressions are very difficult to debug and tend to create confusion as to why a value is not what you expect it to be.

## Session variables

Session variables are used to show things that are only relevant to the HMI. They should only be used for small internal things in the HMI, and one should always keep in mind that they are session unique so if there are multiple clients that both need the changes this needs to be handled on the PLC.

## Event scripts

Event scripts are a replacement for event bindings and are always executed on target. These are implemented as normal JavaScript and can therefore be used for more complex HMI bindings and conversion. Event scripts should be used to replace as many event bindings, snippets and expressions as possible to improve debugging capabilities and make streamline the code.

They require an extra license so this needs to be agreed with the customer beforehand.

## Dialogs and messages

* Dialogs can be used to help make the HMI more compact. When using dialogs, it should be considered that pop ups can be very distracting so they should never appear without an action taken by the operator. Typical uses of dialogs are wizards, configurations or other small information.
* Messages are normal pop-ups that require immediate attention. These should generally not be used. They can be used as confirmations for crucial changes like changeovers, safety acknowledge, critical alarms etc.

# Widgets

Widgets are all the elements that can be added to a page. These are used to show information to the user or for the user to modify relevant variables.

## Preferred widgets

* All [base widgets](https://help.br-automation.com/#/en/6/visualization/mappview/widgets/widgets.html)
* Paper
* ProgressBar
* Tab control
* Table
* ToggleSwitch

## Not recommended widgets

* ContentCarousel
* FlyOut

## Widget class

There are 3 different types of widget classes. Class A, B and C. These are categories that describe the complexity of a widget and are important to keep in mind depending on the hardware used. For example, it is not recommended to use class C widgets on a T50. For more information please see the [help](https://help.br-automation.com/#/en/6/visualization/mappview/guides/performanceguideline/common_hardwareguidance.html). Do keep in mind that these are only recommendations and of course vary depending on the rest of the application.

# Theme

All widgets have needs to use the default theme and additional styles can be defined in the theme. A theme is needed to ensure a cohesive look and feel across the entire HMI and to the reduce time needed to develop a professional looking HMI.

To quickly edit a theme across all widgets please use the ThemeModify script found on [BRDK GitHub](https://github.com/BRDK-GitHub/BrdkScripts).

Please use the BRDK standard theme if the look and feel is not defined by the customer.

# Security options

In the config.mappview file set the protocol to be HTTP, the maximum client to be at least 2 (one for the screen and one for remote access) and change the startup user to be anonymous token.

The diagnostic page should be enabled for debugging purposes and an appropriate role should be chosen for access.

A default visualization must always be given for the best possible user experience.

**Figure 3:** Default Visualization Setting

| Setting | Value |
| :--- | :--- |
| Default ID | Visu |

# Footer

www.br-automation.com
BRDK Architecture / V0.0.0.1
©2024/04/19 by B&R. All rights reserved.
All registered trademarks are the property of their respective owners. We reserve the right to make technical changes.
