# How to build a React scheduler with Bryntum and Microsoft PowerApps Component Framework

The [Power Apps component framework](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/overview) allows developers to create custom code components for [Microsoft Power Apps](https://learn.microsoft.com/en-us/power-apps/powerapps-overview) applications using HTML, CSS, and JavaScript. 

You can connect your Power Apps applications to different [data sources](https://learn.microsoft.com/en-us/training/modules/get-started-with-powerapps/4-powerapps-ways-to-build) such as Excel, Microsoft SQL Server, and [Microsoft Dataverse](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-intro). 
Dataverse is the cloud data platform for the [Microsoft Power Platform](https://www.microsoft.com/en-us/power-platform), which Power Apps is a part of. 

The Power Apps component framework has [various APIs](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/) to simplify development, such as the [Web API](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/webapi) that provides methods to perform database CRUD operations. The Power Apps component framework supports using React, which the Power Apps platform uses, to build components. 

[Bryntum Scheduler](https://bryntum.com/products/scheduler/) is a performant, feature-rich, and highly customizable scheduling UI component that you can use with popular JavaScript frameworks such as React, Angular, and Vue.

In this tutorial, we’ll create a Bryntum scheduler PCF component and add it to a custom page in a Power Apps app.

We’ll do the following:

- Create a Power Apps app in the online Power Apps platform.
- Create Dataverse tables for the scheduler events, resources, dependencies, and assignments using example data imported from CSV files.
- Create a Bryntum Scheduler Power Apps component framework React component locally.
- Bundle all the code component elements into a solution file and then import the solution file into Dataverse using the VS Code Power Platform Tools extension.
- Add the Bryntum Scheduler React component to the Power Apps app.

You can see the code for the completed Bryntum Scheduler Power Apps component in our [GitHub repository](https://github.com/MattDClarke/bryntum-scheduler-power-apps-components).

## Prerequisites

To follow along, you'll need the following software installed on your machine:

- [Node.js](https://nodejs.org/en/)
- [VS Code editor](https://code.visualstudio.com/Download/)
- Build Tools for Visual Studio from [Visual Studio Downloads](https://visualstudio.microsoft.com/downloads#build-tools-for-visual-studio-2022)

You'll also need to install the VS Code [Power Platform Tools extension](https://marketplace.visualstudio.com/items/?itemName=microsoft-IsvExpTools.powerplatform-vscode).

## Getting access to Microsoft Power Apps

To start using Power Apps, sign in to your organizational [Microsoft 365](https://www.microsoft.com/microsoft-365) account or create a [Microsoft account](https://account.microsoft.com/account) using your work email address and join the [Microsoft 365 Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program) with that Microsoft account.

Then go to the Power Apps Maker Portal at [make.powerapps.com](https://make.powerapps.com/), where you can begin creating your app.

Sign up for a [30-day Power Apps trial plan](https://signup.microsoft.com/get-started/signup?products=83d3609a-14c1-4fc2-a18e-0f5ca7047e46) if you don’t already have a Power Apps license or a license through Office 365.

The trial plan provides temporary access to the following activities:

- Create and run Power Apps [canvas apps](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/getting-started) that connect to Microsoft Dataverse and more than 200 other data sources, including premium connectors and on-premises data.
- Create and run Power Apps [model-driven apps](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/model-driven-app-overview).
- Create and manage environments and Dataverse databases.

## Creating an app in Power Apps

We'll now create a Power Apps app in the Power Apps Maker Portal. Follow these steps to create an app:

- Go to [https://make.powerapps.com](https://make.powerapps.com/home).
- Click the "+ Create" menu item in the navigation on the left.
- Click the Start from Blank App card to create an app from scratch.
- Select Blank App based on Dataverse and click the "Create" button.
- Give your app the name "Bryntum Scheduler".
- Add a page to your navigation by clicking the "+ Add page" button.
- Select the "Custom page" button in the **Add page** popup.
- In the **Select a custom page** form, click the "+ Create custom page" button.
- Name the page "Bryntum Scheduler" and then click the "Create" button.

We'll create a custom Bryntum Scheduler React component to add to this custom page. But first, we'll create Dataverse tables for the Scheduler events, resources, dependencies, and assignments.

## Creating custom Dataverse tables

We'll store our data in Dataverse tables. There are multiple benefits to using Dataverse as a data store for a Power Apps app, including:

- [Security and access management](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/why-dataverse-overview#security)
- [Importing and exporting data](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/import-export-data?source=recommendations)
- [Storing any type of data](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/work-with-any-data)
- A [REST API](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/perform-operations-web-api)

We'll create four Dataverse tables to store our scheduler data and import example CSV data into them.

### Creating a Dataverse table to store events data

Follow these steps to create a Dataverse table for our events and populate it with example CSV data:

- Go to https://make.powerapps.com.
- Click the "Tables" menu item in the navigation on the left.
- Click the "Create with Excel or .CSV file" card.

Copy and paste the following data into a `bryntum-scheduler-events.csv` file.

```
name,startDate,endDate,readOnly,timeZone,draggable,resizable,children,allDay,duration,durationUnit,exceptionDates,recurrenceRule,cls,eventColor,eventStyle,iconCls,style
Conference call,2024/07/01 09:00,2024/07/01 10:30,,,,,,,,,,,,,,,
Sprint planning,2024/07/01 11:30,2024/07/01 13:00,,,,,,,,,,,,,,,
Team meeting,2024/07/01 12:00,2024/07/01 13:30,,,,,,,,,,,,,,,
```

The headings in this CSV represent most of the [fields for a Bryntum Scheduler event](https://bryntum.com/products/scheduler/docs/api/Scheduler/model/EventModel).

Now upload this CSV file. Once the CSV file is uploaded, you’ll see the following screen:

![Dataverse table view in the Power Apps maker portal](ritza_bryntum-powerapps-scheduler_dataverse-table-create.png)

Click on the "Edit table properties" button. Change the following properties of the table:

- Display name: Bryntum Scheduler Events
- Schema name: bryntumschedulerevents
- Primary column: name

Save the table properties.

![Edit table properties popup](ritza_bryntum-powerapps-scheduler_dataverse-table-edit-table.png)

We'll use the schema name when making queries to the database table. Note that the names of tables and columns have an auto-generated prefix that makes their names unique.

The primary column is used when a table has a [relationship](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-entity-lookup) to another table. You can use a lookup column to show data from another table. If another table has a lookup column that links to this Bryntum Scheduler Events table, you can add rows to the lookup column using a dropdown list. The dropdown list will contain the events of the linked Bryntum Scheduler Events table. Each list item will be an event name, which is the primary column of the Bryntum Scheduler Events table. 

We'll now edit each column by clicking on the column header and then clicking on the "Edit column" pop-up: 

![Edit column popup](ritza_bryntum-powerapps-scheduler_dataverse-table-edit-column.png)

Make sure that the "Use first row as column headers" toggle at the top-right of the page is on.

Change the column properties as follows:

| Display name   | Schema name    | Data type                 | Default choice |
| -------------- | -------------- | ------------------------- | -------------- |
| name           | name           | Single line of plain text |                |
| startDate      | startdate      | Date and time             |                |
| endDate        | enddate        | Date and time             |                |
| readOnly       | readonly       | Yes/no Choice             | No             |
| timeZone       | timezone       | Single line of plain text |                |
| draggable      | draggable      | Yes/no Choice             | Yes            |
| resizable      | resizable      | Single line of plain text |                |
| children       | children       | Single line of plain text |                |
| allDay         | allday         | Yes/no Choice             | No             |
| duration       | duration       | Float                     |                |
| durationUnit   | durationunit   | Single line of plain text |                |
| exceptionDates | exceptiondates | Single line of plain text |                |
| recurrenceRule | recurrencerule | Single line of plain text |                |
| cls            | cls            | Single line of plain text |                |
| eventColor     | eventcolor     | Single line of plain text |                |
| eventStyle     | eventstyle     | Single line of plain text |                |
| iconCls        | iconcls        | Single line of plain text |                |
| style          | style          | Single line of plain text |                |

Click the "Create" button at the bottom-right of the screen to create the table.

You can find your custom Dataverse table by opening the "Tables" menu item in the navigation on the left and then clicking the "Custom" button above the table of tables:

![Show custom Dataverse tables](ritza_bryntum-powerapps-scheduler_dataverse-table-view-custom.png)

### Creating a Dataverse table to store resources data

We'll now create a table for the resources data.

Copy and paste the following data into a `bryntum-scheduler-resources.csv` file.

```
name,eventColor,readOnly
Peter,,
Kate,,
Winston,,
```

The headings in this CSV text represent most of the [fields for a Bryntum Scheduler resource](https://bryntum.com/products/scheduler/docs/api/Scheduler/model/ResourceModel).

Select "Create with Excel or .CSV file" and upload this CSV file.

Click on the "Edit table properties" button. Change the following properties of the table:

- Display name: Bryntum Scheduler Resources
- Primary column: name
- Schema name: bryntumschedulerresources

Change the column properties as follows:

| Display name | Schema name | Data type                 | Default choice |
| ------------ | ----------- | ------------------------- | -------------- |
| name         | name        | Single line of plain text |                |
| eventColor   | eventcolor  | Single line of plain text |                |
| readOnly     | readonly    | Yes/no Choice             | No             |

Click the "Create" button to create the table.

### Creating a Dataverse table to store dependencies data

We'll now create a table for the dependencies data.

Copy and paste the following data into a `bryntum-scheduler-dependencies.csv` file.

```
type,cls,lag,lagUnit,active,fromSide,toSide
2,,0,day,end,start
```

The headings in this CSV text represent most of the [fields for a Bryntum Scheduler dependency](https://bryntum.com/products/scheduler/docs/api/Scheduler/model/DependencyModel).

Select "Create with Excel or .CSV file" and upload this CSV file.

Click on the "Edit table properties" button. Change the following properties of the table:

- Display name: Bryntum Scheduler Dependencies
- Primary column: cls
- Schema name: bryntumschedulerdependencies

Change the column properties as follows:

| Display name | Schema name | Data type                 |
| ------------ | ----------- | ------------------------- |
| type         | type        | Whole number              |
| cls          | cls         | Single line of plain text |
| lag          | lag         | Float                     |
| lagUnit      | lagunit     | Single line of plain text |
| fromSide     | fromside    | Single line of plain text |
| toSide       | toside      | Single line of plain text |

Note that you can use the Choice data type to restrict the possible values of a column row when there's a limited set of valid values. You could do this for the [type](https://bryntum.com/products/scheduler/docs/api/Scheduler/model/DependencyModel#field-type), [lagUnit](https://bryntum.com/products/scheduler/docs/api/Scheduler/model/DependencyModel#field-lagUnit), [fromSide](https://bryntum.com/products/scheduler/docs/api/Scheduler/model/DependencyModel#field-fromSide), and [toSide](https://bryntum.com/products/scheduler/docs/api/Scheduler/model/DependencyModel#field-toSide) columns.

Click the "Create" button to create the table.

Open the Bryntum Scheduler Dependencies table in a separate tab, click on the "+18 more" button on the right of the dependencies table, and then check the "Bryntum Scheduler Dependencies" column:

![Show id column](ritza_bryntum-powerapps-scheduler_dataverse-table-show-id-column.png)

This id column uses globally unique identifiers (GUIDs) and is auto-generated by Dataverse. If you open the edit column popup for this column, you'll see that its data type is "Unique identifier".

Open the edit column popup for one of the custom columns that you created. You see a new "Logical name" property under the Advanced options dropdown. We'll use this property when we create and update records using the [Dataverse Web API](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/overview) from the custom Bryntum Scheduler Power Apps component that we'll create.

#### Create from and to columns for the dependencies table with relationships to the events table

We'll now create "from" and "to" lookup columns for the dependencies table that are related to the events table. These columns will indicate which events are connected in a dependency.

Click the "New column" button to the right of the "+17 more" button in the **Bryntum Scheduler Dependencies columns and data** table.

Set the following values in the popup form and then click the save button:

- Display name: from
- Data type: Lookup
- Related table: Bryntum Scheduler Events
- Schema name: from

![New from lookup column form](ritza_bryntum-powerapps-scheduler_dataverse-table-from-lookup-column-form.png)

You'll now be able to add values to the "from" column using an auto-complete input. The displayed values are the primary column (name column) values of the columns of the linked events table. Set the "from" value for the example dependency to "Conference call".

![Lookup table auto-complete](ritza_bryntum-powerapps-scheduler_dataverse-table-from-lookup-column-autocomplete.png)

Create another column with the following values:

- Display name: to
- Data type: Lookup
- Related table: Bryntum Scheduler Events
- Schema name: to

Set the "to" value for the example dependency to "Sprint planning".

### Creating a Dataverse table to store assignments data

We'll now create a table for the assignments data. Create a table by clicking on the "Start with a blank table" card. 

Click on the "Edit table properties" button. Change the following properties of the table:

- Display name: Bryntum Scheduler Assignments
- Schema name: bryntumschedulerassignments

The primary column will be the "New column" column that's added by default. We won't use this column for our scheduler. Make sure that the Required value for the "New column" is set to "optional".

Create a column by clicking on the "+"  New column button to the right of the column header and add the following values:

- Display name: eventId
- Data type: Lookup
- Related table: Bryntum Scheduler Events
- Schema name: eventid

Create another column with the following values:

- Display name: resourceId
- Data type: Lookup
- Related table: Bryntum Scheduler Resources
- Schema name: resourceid

Click the "Create" button to create the table.

Create three rows with the following values in the **Bryntum Scheduler Assignments columns and data** table:

| eventId         | resourceId |
| --------------- | ---------- |
| Conference call | Peter      |
| Sprint planning | Kate       |
| Team meeting    | Winston    |

Now that our Dataverse database tables are ready, let's create a Bryntum Scheduler Power Apps component to add to our Power Apps page.

## Create a Bryntum Scheduler Power Apps component framework component

We'll use the [Power Apps component framework](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/overview) to create a Bryntum Scheduler code component that we can use in our Power Apps app.

We'll use the [Microsoft Power Platform CLI](https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction?tabs=windows#install-microsoft-power-platform-cli) to create a PCF project and push the component to the Power Apps platform.

Make sure that you've installed the prerequisite VS Code [Power Platform Tools extension](https://marketplace.visualstudio.com/items/?itemName=microsoft-IsvExpTools.powerplatform-vscode) in VS Code. This extension enables you to use the Microsoft Power Platform CLI commands in a PowerShell terminal within Visual Studio Code on Windows 10, Windows 11, Linux, and macOS.

Run the following command in a new directory to create a PCF React project template:

```shell
pac pcf init --name BryntumScheduler --namespace BryntumScheduler --template field --framework react
```

The [`pac pcf init`](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/pcf#pac-pcf-init) command initializes a directory with a new PCF project.

- `--name`: Name of the component.
- `--namespace`: Namespace for the component.
- `--template`: Template for the component. The value can be `field` or `dataset`.
- `--framework`: Rendering framework for the control. The value can be `none` or `react`. The default value is `none`, which means HTML is used for rendering.

Now install the project dependencies:

```shell
npm install
```

Next, install the [Bryntum Scheduler packages for React](https://bryntum.com/products/scheduler/docs/guide/Scheduler/quick-start/react#install-bryntum-scheduler-packages). Make sure that you have [access to the Bryntum npm repository](https://bryntum.com/products/scheduler/docs/guide/Scheduler/npm-repository).

```shell
npm install @bryntum/scheduler@5.6.0 @bryntum/scheduler-react@5.6.0
```

### Updating the code component's manifest

The [manifest](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/manifest) `BryntumScheduler/ControlManifest.Input.xml` is an XML metadata file that describes the component. We'll update it to accurately describe our component.

Replace the contents of `BryntumScheduler/ControlManifest.Input.xml` with the following code:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest>
    <control namespace="BryntumSchedulerComponent" constructor="BryntumScheduler" version="2.0.1"
        display-name-key="BryntumSchedulerComponent" description-key="BryntumSchedulerComponent description"
        control-type="virtual">
        <!--external-service-usage
        node declares whether this 3rd party PCF control is using external service or not, if yes,
        this control will be considered as premium and please also add the external domain it is
        using.
        If it is not using any external service, please set the enabled="false" and DO NOT add any domain
            below. The "enabled" will be false by default.
        Example1:
        <external-service-usage enabled="true">
            <domain>www.Microsoft.com</domain>
        </external-service-usage>
        Example2:
        <external-service-usage enabled="false">
        </external-service-usage>
        -->
        <external-service-usage enabled="false">
        <!--UNCOMMENT
        TO ADD EXTERNAL DOMAINS
        <domain></domain>
        <domain></domain>
        -->
        </external-service-usage>
        <!-- property node identifies a specific, configurable piece of data that the control
        expects from CDS -->
        <property name="BryntumScheduler" display-name-key="BryntumScheduler"
            description-key="BryntumScheduler description" of-type="SingleLine.Text" usage="bound"
            required="false" />

        <resources>
            <code path="index.ts" order="1" />
            <platform-library name="React" version="16.8.6" />
            <css path="css/BryntumSchedulerComponent.css" order="1" />
            <css path="css/scheduler.stockholm.css" order="2" />
        </resources>

        <feature-usage>
            <uses-feature name="WebAPI" required="true" />
        </feature-usage>

    </control>
</manifest>
```

Here we add the CSS resources we'll create in the next step in the `<resources>` tag.

In the `<feature-usage>` tag, we add the [WebAPI feature](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/webapi), which provides properties and methods to perform CRUD operations on our Dataverse tables.

### Adding CSS styling

Let's create the CSS resource files that we added to the manifest.

Create a `css` folder in the `BryntumScheduler` component folder. Copy the `scheduler.stockholm.css` file from the `node_modules/@bryntum/scheduler` folder and add it to the `css` folder.

We need to add this CSS file to our component folder because [bundling font resources is currently not supported by the Power Apps component framework](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/faq#can-i-bundle-font-resources). Bryntum components use icons that are based on the [Font Awesome 6 Free](https://fontawesome.com/icons?d=gallery&m=free) solid font. This [Bryntum Scheduler CSS theme](https://bryntum.com/products/scheduler/docs/guide/Scheduler/customization/styling#using-a-theme) file uses these Font Awesome font resources. We'll use the Font Awesome CDN to get these font resources in our component.

In the `css/scheduler.stockholm.css` file, find the four occurrences of the following `@font-face` source URL:

```css
"fonts/fa-solid-900.woff2"
```

Replace each of them with this CDN URL:

```css
"https://use.fontawesome.com/releases/v6.0.0/webfonts/fa-solid-900.woff2"
```

Now find the four occurrences of the following `@font-face` source URL:

```css
"fonts/fa-solid-900.ttf"
```

Replace each of them with this CDN URL:

```css
"https://use.fontawesome.com/releases/v6.0.0/webfonts/fa-solid-900.ttf"
```

In the `BryntumScheduler/css` folder, create a `BryntumSchedulerComponent.css` file and add the following styles to it:

```css
#root {
  height: 100vh;
}

.loader-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  width: 100%;
}

.loader {
 border: 16px solid #f3f3f3;
 border-radius: 50%;
 border-top: 16px solid #3498db;
 width: 80px;
 height: 80px;
 -webkit-animation: spin 2s linear infinite;
 /* Safari */
 animation: spin 2s linear infinite;
}

/* Safari */
@-webkit-keyframes spin {
  0% {
    -webkit-transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
  }
}

@keyframes spin {
  0% {
     transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
```

### Creating the Bryntum Scheduler component

Before we create the Bryntum Scheduler React component, first create a `constants.ts` file in the `BryntumScheduler` folder and add the following lines of code to it:

```ts
export const databasePrefix = 'cr8a7_';
export const eventsDataverseTableName = 'bryntumschedulerevents';
export const resourcesDataverseTableName = 'bryntumschedulerresources';
export const dependenciesDataverseTableName = 'bryntumschedulerdependencies';
export const assignmentsDataverseTableName = 'bryntumschedulerassignments';
```

Replace the `databasePrefix` value with the database prefix value in your Dataverse tables.

We'll show a loading spinner React component while the Dataverse data is being fetched. Create a file called `LoadingSpinner.tsx` in the `BryntumScheduler` folder and add the following lines of code to it:

```typescript
import * as React from "react";

function LoadingSpinner() {
  return (
    <div className="loader-container">
      <div className="loader" />
    </div>
  );
}

export default LoadingSpinner;
```

Create a `BryntumSchedulerComponent.types.ts` file in the `BryntumScheduler` folder and add the following types to it:

```typescript
import { databasePrefix } from "./constants";
import { IInputs } from "./generated/ManifestTypes";

export interface IBryntumSchedulerComponentProps {
  context?: ComponentFramework.Context<IInputs>;
}

type RecordItem = {
  data:
    | SchedulerEvent
    | SchedulerResource
    | SchedulerDependency
    | SchedulerAssignment;
  meta: {
    modified:
      | Partial<SchedulerEvent>
      | Partial<SchedulerResource>
      | Partial<SchedulerDependency>
      | Partial<SchedulerAssignment>;
  };
} & SchedulerEvent & SchedulerResource & SchedulerDependency & SchedulerAssignment;

export type SyncData = {
  action: "dataset" | "add" | "remove" | "update";
  records: RecordItem[];
  store: {
    id: "events" | "resources" | "dependencies" | "assignments";
  };
};

export type SchedulerEvent = {
  id: string;
  name: string;
  startDate: string;
  endDate: string;
  readOnly: number;
  timeZone: string;
  draggable: number;
  resizable: string;
  children: string;
  allDay: number;
  duration: number;
  durationUnit: string;
  exceptionDates: string;
  recurrenceRule: string;
  cls: string;
  eventColor: string;
  eventStyle: string;
  iconCls: string;
  style: string;
};

const KEY_ID = `${databasePrefix}id` as const;
const KEY_NAME = `${databasePrefix}name` as const;
const KEY_START_DATE = `${databasePrefix}startdate` as const;
const KEY_END_DATE = `${databasePrefix}enddate` as const;
const KEY_READONLY = `${databasePrefix}readonly` as const;
const KEY_TIME_ZONE = `${databasePrefix}timezone` as const;
const KEY_DRAGGABLE = `${databasePrefix}draggable` as const;
const KEY_RESIZABLE = `${databasePrefix}resizable` as const;
const KEY_CHILDREN = `${databasePrefix}children` as const;
const KEY_ALL_DAY = `${databasePrefix}allday` as const;
const KEY_DURATION = `${databasePrefix}duration` as const;
const KEY_DURATION_UNIT = `${databasePrefix}durationunit` as const;
const KEY_EXCEPTION_DATES = `${databasePrefix}exceptiondates` as const;
const KEY_RECURRENCE_RULE = `${databasePrefix}recurrencerule` as const;
const KEY_CLS = `${databasePrefix}cls` as const;
const KEY_EVENT_COLOR = `${databasePrefix}eventcolor` as const;
const KEY_EVENT_STYLE = `${databasePrefix}eventstyle` as const;
const KEY_ICON_CLS = `${databasePrefix}iconcls` as const;
const KEY_STYLE = `${databasePrefix}style` as const;

export type SchedulerEventDataverse = {
  [KEY_ID]: string;
  [KEY_NAME]: string;
  [KEY_START_DATE]: string;
  [KEY_END_DATE]: string;
  [KEY_READONLY]: number;
  [KEY_TIME_ZONE]: string;
  [KEY_DRAGGABLE]: number;
  [KEY_RESIZABLE]: string;
  [KEY_CHILDREN]: string;
  [KEY_ALL_DAY]: number;
  [KEY_DURATION]: number;
  [KEY_DURATION_UNIT]: string;
  [KEY_EXCEPTION_DATES]: string;
  [KEY_RECURRENCE_RULE]: string;
  [KEY_CLS]: string;
  [KEY_EVENT_COLOR]: string;
  [KEY_EVENT_STYLE]: string;
  [KEY_ICON_CLS]: string;
  [KEY_STYLE]: string;
};

export type SchedulerResource = {
  id: string;
  name: string;
  eventColor: string;
  readOnly: string;
};

export type SchedulerResourceDataverse = {
  [KEY_ID]: string;
  [KEY_NAME]: string;
  [KEY_EVENT_COLOR]: string;
  [KEY_READONLY]: number;
};

export type SchedulerDependency = {
  id: string;
  type: number;
  cls: string;
  lag: number;
  lagUnit: string;
  fromSide: string;
  toSide: string;
  from: string;
  to: string;
};

const KEY_TYPE = `${databasePrefix}type` as const;
const KEY_LAG = `${databasePrefix}lag` as const;
const KEY_LAG_UNIT = `${databasePrefix}lagunit` as const;
const KEY_FROM_SIDE = `${databasePrefix}fromside` as const;
const KEY_TO_SIDE = `${databasePrefix}toside` as const;
const KEY_FROM = `${databasePrefix}from@odata.bind` as const;
const KEY_TO = `${databasePrefix}to@odata.bind` as const;

export type SchedulerDependencyDataverse = {
  [KEY_ID]: string;
  [KEY_TYPE]: number;
  [KEY_CLS]: string;
  [KEY_LAG]: number;
  [KEY_LAG_UNIT]: string;
  [KEY_FROM_SIDE]: string;
  [KEY_TO_SIDE]: string;
  [KEY_FROM]: string;
  [KEY_TO]: string;
};

export type SchedulerAssignment = {
  id: string; 
  eventId: string;
  resourceId: string;
};

const KEY_EVENT_ID = `${databasePrefix}eventid@odata.bind` as const;
const KEY_RESOURCE_ID = `${databasePrefix}resourceid@odata.bind` as const;

export type SchedulerAssignmentDataverse = {
  [KEY_ID]: string;
  [KEY_EVENT_ID]: string;
  [KEY_RESOURCE_ID]: string;
};
```

These are the TypeScript types we'll use for the Bryntum Scheduler and Dataverse data.

We'll now define some basic configurations for our Bryntum Scheduler. Create a file called `schedulerConfig.ts` in the `BryntumScheduler` folder and add the following lines of code to it:

```typescript
import type { BryntumSchedulerProps } from "@bryntum/scheduler-react";

const schedulerConfig: Partial<BryntumSchedulerProps> = {
  features: {
    stripe: true,
    dependencies: true,
  },
  columns: [{ text: "Name", field: "name", width: 250 }],
  viewPreset: "hourAndDay",
  startDate: new Date(2024, 6, 1, 9),
  endDate: new Date(2024, 6, 1, 17),
  barMargin: 10,
  selectionMode: {
    multiSelect: false,
  },
};

export { schedulerConfig };
```

Now create the Bryntum Scheduler React component. Create a `BryntumSchedulerComponent.tsx` file in the `BryntumScheduler` folder and add the following lines of code to it:


```typescript
import { BryntumScheduler as OriginalBryntumScheduler } from "@bryntum/scheduler-react";
import * as React from "react";
import { FunctionComponent, useRef } from "react";
import {
    IBryntumSchedulerComponentProps,
    SchedulerAssignment,
    SchedulerDependency,
    SchedulerEvent,
    SchedulerResource,
} from "./BryntumSchedulerComponent.types";
import LoadingSpinner from "./LoadingSpinner";
import { schedulerConfig } from "./schedulerConfig";

const BryntumScheduler: React.ComponentType<any> = OriginalBryntumScheduler as any;

const BryntumSchedulerComponent: FunctionComponent<IBryntumSchedulerComponentProps> = (
  props
) => {
  const [data, setData] = React.useState<{
    events: SchedulerEvent[];
    resources: SchedulerResource[];
    dependencies: SchedulerDependency[];
    assignments: SchedulerAssignment[];
  }>();

  const scheduler = useRef<OriginalBryntumScheduler>(null);

  return data ? (
    <BryntumScheduler
      ref={scheduler}
      events={data.events}
      resources={data.resources}  
      dependencies={data.dependencies}
      assignments={data.assignments}
      {...schedulerConfig}
    />
  ) : (
    <LoadingSpinner />
  );
};

export default BryntumSchedulerComponent;
```

We render the Bryntum Scheduler React component and pass in `schedulerConfig` as a prop. We'll store the data in the `data` state variable and pass in the events, resources, dependencies, and assignments data properties as props. When there's no data, the `LoadingSpinner` component is displayed.

To render the component, we need to add the PCF component logic to the auto-generated `index.ts` file. This file has a class that creates an object which implements the following methods that control the lifecycle of the PCF component:

- [`init`](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/control/init) (Required)
- [`updateView`](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/control/updateview) (Required)
- [`getOutputs`](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/control/getoutputs) (Optional)
- [`destroy`](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/control/getoutputs). (Required)

Add the following lines of code to the `init` function:

```typescript
    Object.defineProperty(window, "globalThis", {
      value: window,
      writable: false,
    });
```

Replace the `updateView` function with the following lines of code:

```typescript
  public updateView(
    context: ComponentFramework.Context<IInputs>
  ): React.ReactElement {
    const props: IBryntumSchedulerComponentProps = { context };

    return React.createElement(BryntumSchedulerComponent, props);
  }
```

We render our `BryntumSchedulerComponent` in this function, which is called when any value in the property bag has changed. The property bag includes field values, datasets, and global values like container height and width.

Import the component and the prop types:

```ts
import BryntumSchedulerComponent from './BryntumSchedulerComponent';
import { IBryntumSchedulerComponentProps } from './BryntumSchedulerComponent.types';
```

You can remove the imports from the `HelloWorld.tsx` file and delete the file, too.

Let's create a build of the component.

```shell
npm run build
```

This command creates an `out` folder in the root directory. The folder contains the component's manifest, CSS, and JavaScript bundle.

Now run the local [browser test harness](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/debugging-custom-controls#debugging-using-the-browser-test-harness) to view the component:

```shell
npm run start
```

A browser window or tab will automatically open to the local test harness URL: [http://localhost:8181/](http://localhost:8181/). You'll see the loading spinner:

![Test harness showing loading spinner](ritza_bryntum-powerapps-scheduler_test-harness-loader.png)

You can use your browser React dev tools to change the `data` state variable to an empty array. If you do this, you'll see the Bryntum Scheduler with no data:

![Empty Bryntum Scheduler](ritza_bryntum-powerapps-scheduler_test-harness-empty-scheduler.png)

Let's fetch the Dataverse data and add it to our Scheduler component.

## Fetching data using the Web API `retrieveMultipleRecords` method

We'll use the Web API [`retrieveMultipleRecords`](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/webapi/retrievemultiplerecords) method to fetch the events, resources, dependencies, and assignment records from our Dataverse tables.

Add the following lines of code to the `BryntumSchedulerComponent.tsx` file, below the `scheduler` variable declaration:

```typescript
  const fetchRecords = async () => {
    try {
      const eventsPromise = props?.context?.webAPI.retrieveMultipleRecords(
        `${databasePrefix}${eventsDataverseTableName}`
      );
      const resourcesPromise =
        props?.context?.webAPI.retrieveMultipleRecords(
        `${databasePrefix}${resourcesDataverseTableName}`
        );
      const dependenciesPromise:
        | Promise<ComponentFramework.WebApi.RetrieveMultipleResponse>
        | undefined = props?.context?.webAPI.retrieveMultipleRecords(
        `${databasePrefix}${dependenciesDataverseTableName}`,
        `?$select=${databasePrefix}type,${databasePrefix}lag,${databasePrefix}lagunit,${databasePrefix}fromside,${databasePrefix}toside,${databasePrefix}from,${databasePrefix}to&$expand=${databasePrefix}from($select=${databasePrefix}${eventsDataverseTableName}id),${databasePrefix}to($select=${databasePrefix}${eventsDataverseTableName}id)`
      );
      const assignmentsPromise:
        | Promise<ComponentFramework.WebApi.RetrieveMultipleResponse>
        | undefined = props?.context?.webAPI.retrieveMultipleRecords(
        `${databasePrefix}${assignmentsDataverseTableName}`,
        `?$select=${databasePrefix}eventid,${databasePrefix}resourceid&$expand=${databasePrefix}eventid($select=${databasePrefix}${eventsDataverseTableName}id),${databasePrefix}resourceid($select=${databasePrefix}${resourcesDataverseTableName}id)`
      );

      const [events, resources, dependencies, assignments] = await Promise.all([
        eventsPromise,
        resourcesPromise,
        dependenciesPromise,
        assignmentsPromise,
      ]);

      if (events && resources && dependencies && assignments) {
        setData({
          events: removeEventsDataColumnPrefixes(events.entities) as SchedulerEvent[],
          resources: removeResourcesDataColumnPrefixes(resources.entities) as SchedulerResource[],
          dependencies: removeDependenciesDataColumnPrefixes(
            dependencies.entities
          ) as SchedulerDependency[],
            assignments: removeAssignmentsDataColumnPrefixes(
                assignments.entities
            ) as SchedulerAssignment[],
        });
      }
    } catch (e) {
      if (e instanceof Error) {
        if (e.name === "PCFNonImplementedError") {
          console.log("PCFNonImplementedError: ", e.message);
          // You can add fallback data state for the development mode browser test harness
          // webAPI is not available in the test harness
        }
      }
      throw e;
    }
  };

  React.useEffect(() => {
    fetchRecords();
  }, []);
```

When the component is mounted, we call the `fetchRecords` function, which calls the `retrieveMultipleRecords` method to fetch the events, resources, dependencies, and assignments data. The logical name of the Dataverse table is passed in as an argument to the `retrieveMultipleRecords` method. We access the Web API through the [`context`](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/context) object that we passed in from our `index.ts` file as a prop.

The `removeEventsDataColumnPrefixes`, `removeResourcesDataColumnPrefixes`, `removeDependenciesDataColumnPrefixes` and `removeAssignmentsDataColumnPrefixes` functions transform the Dataverse data to match the format expected by the Bryntum Scheduler.

Add the following definitions for these functions at the top of the file:

```typescript
function removeEventsDataColumnPrefixes(entities: any[]) {
  return entities.map((entity: ComponentFramework.WebApi.Entity) => {
    let event: Partial<SchedulerEvent> = {};
    if (entity[`${databasePrefix}${eventsDataverseTableName}id`]) {
        event.id = entity[`${databasePrefix}${eventsDataverseTableName}id`];
    }
    if (entity[`${databasePrefix}name`]) {
        event.name = entity[`${databasePrefix}name`];
    }
    if (entity[`${databasePrefix}startdate`]) {
        event.startDate = entity[`${databasePrefix}startdate`];
    }
    if (entity[`${databasePrefix}enddate`]) {
        event.endDate = entity[`${databasePrefix}enddate`];
    }
    if (entity[`${databasePrefix}readonly`]) {
        event.readOnly = entity[`${databasePrefix}readonly`];
    }
    if (entity[`${databasePrefix}timezone`]) {
        event.timeZone = entity[`${databasePrefix}timezone`];
    }
    if (entity[`${databasePrefix}draggable`]) {
        event.draggable = entity[`${databasePrefix}draggable`];
    }
    if (entity[`${databasePrefix}resizable`]) {
        event.resizable = entity[`${databasePrefix}resizable`];
    }
    if (entity[`${databasePrefix}children`]) {
        event.children = entity[`${databasePrefix}children`];
    }
    if (entity[`${databasePrefix}allday`]) {
        event.allDay = entity[`${databasePrefix}allday`];
    }
    if (entity[`${databasePrefix}duration`]) {
        event.duration = entity[`${databasePrefix}duration`];
    }
    if (entity[`${databasePrefix}durationunit`]) {
        event.durationUnit = entity[`${databasePrefix}durationunit`];
    }
    if (entity[`${databasePrefix}exceptiondates`]) {
        event.exceptionDates = entity[`${databasePrefix}exceptiondates`];
    }
    if (entity[`${databasePrefix}recurrencerule`]) {
        event.recurrenceRule = entity[`${databasePrefix}recurrencerule`];
    }
    if (entity[`${databasePrefix}cls`]) {
        event.cls = entity[`${databasePrefix}cls`];
    }
    if (entity[`${databasePrefix}eventcolor`]) {
        event.eventColor = entity[`${databasePrefix}eventcolor`];
    }
    if (entity[`${databasePrefix}eventstyle`]) {
        event.eventStyle = entity[`${databasePrefix}eventstyle`];
    }
    if (entity[`${databasePrefix}iconcls`]) {
        event.iconCls = entity[`${databasePrefix}iconcls`];
    }
    if (entity[`${databasePrefix}style`]) {
        event.style = entity[`${databasePrefix}style`];
    }
    return event;
  });
}

function removeResourcesDataColumnPrefixes(entities: any[]) {
  return entities
    .map((entity: ComponentFramework.WebApi.Entity) => {
        let resource: Partial<SchedulerResource> = {};
        if (entity[`${databasePrefix}${resourcesDataverseTableName}id`]) {
            resource.id =
            entity[`${databasePrefix}${resourcesDataverseTableName}id`];
        }
        if (entity[`${databasePrefix}name`]) {
            resource.name = entity[`${databasePrefix}name`];
        }
        if (entity[`${databasePrefix}eventcolor`]) {
            resource.eventColor = entity[`${databasePrefix}eventcolor`];
        }
        if (entity[`${databasePrefix}readonly`]) {
            resource.readOnly = entity[`${databasePrefix}readonly`];
        }
        return resource;
    });
}

function removeDependenciesDataColumnPrefixes(entities: any[]) {
  return entities
    .map((entity: ComponentFramework.WebApi.Entity) => {
      let dependency: Partial<SchedulerDependency> = {};
      if (entity[`${databasePrefix}${dependenciesDataverseTableName}id`]) {
        dependency.id =
          entity[`${databasePrefix}${dependenciesDataverseTableName}id`];
      }
      if (entity[`${databasePrefix}type`]) {
          dependency.type = entity[`${databasePrefix}type`];
      }
      if (entity[`${databasePrefix}cls`]) {
          dependency.cls = entity[`${databasePrefix}cls`];
      }
     if (entity[`${databasePrefix}lag`]) {
         dependency.lag = entity[`${databasePrefix}lag`];
     }
     if (entity[`${databasePrefix}lagunit`]) {
         dependency.lagUnit = entity[`${databasePrefix}lagunit`];
     }
     if (entity[`${databasePrefix}fromside`]) {
         dependency.fromSide = entity[`${databasePrefix}fromside`];
     }
     if (entity[`${databasePrefix}toside`]) {
       dependency.toSide = entity[`${databasePrefix}toside`];
     }
     if (entity[`${databasePrefix}from`]) {
       dependency.from = entity[`${databasePrefix}from`][
         `${databasePrefix}${eventsDataverseTableName}id`
       ];
     }
     if (entity[`${databasePrefix}to`]) {
       dependency.to = entity[`${databasePrefix}to`][
         `${databasePrefix}${eventsDataverseTableName}id`
      ];
    }
    return dependency;
    });
}

function removeAssignmentsDataColumnPrefixes(entities: any[]) {
  return entities
    .map((entity: ComponentFramework.WebApi.Entity) => {
      let assignment: Partial<SchedulerAssignment> = {};
      if (entity[`${databasePrefix}${assignmentsDataverseTableName}id`]) {
        assignment.id =
          entity[`${databasePrefix}${assignmentsDataverseTableName}id`];
      }
     if (entity[`${databasePrefix}eventid`]) {
       assignment.eventId = entity[`${databasePrefix}eventid`][
         `${databasePrefix}${eventsDataverseTableName}id`
       ];
     }
     if (entity[`${databasePrefix}resourceid`]) {
       assignment.resourceId = entity[`${databasePrefix}resourceid`][
         `${databasePrefix}${resourcesDataverseTableName}id`
       ];
     }
     return assignment;
    });
}
```

Add the imports for the constant values:

```ts
import {
  databasePrefix,
  eventsDataverseTableName,
  resourcesDataverseTableName,
  dependenciesDataverseTableName,
  assignmentsDataverseTableName,
} from "./constants";
```

The Web API methods currently can't be tested in the test harness. We need to publish and host the component in a Microsoft Power Platform environment.

## Creating a component solution package

You need to deploy your component to a Microsoft Dataverse environment before using it in Power Apps. The first step is to package the component into a solution that you can import. There are two ways to do this:

1. During development, use the Microsoft Power Platform CLI `push` command to create a temporary solution file that pushes the component to a Microsoft Dataverse environment. The solution file consists of a bundle of all of the component elements.
2. For build pipelines or manual deploys, create a solution for the component and import it separately into your Dataverse environment.

We'll use the first way.

To push the Bryntum Scheduler component to your Microsoft Power Platform, do the following:

- Open your Bryntum Scheduler PCF component project in VS Code, open the **Power Platform** tab, and select **Add Auth Profile**.
- Look for a **Sign in to your account** popup and sign in with your Microsoft account.
- The profile you added will show up under **Auth Profiles** and you'll also see the profile's **Environments & Solutions**. Select the UNIVERSAL profile and the MSFT (default) environment.

⚠️ To add custom components to a Power Apps app, you need to enable the Power Apps component framework for canvas apps feature in the environment where you want to use the component. You can do this by following the [Microsoft guide to enabling the Power Apps component framework feature](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/component-framework-for-canvas-apps).

- Build your project by running the build command:

```shell
npm run build
```

- Push your custom component to the Power Apps platform environment by running the following command:

```shell
pac pcf push --publisher-prefix dev
```

This command imports your Power Apps component framework project into the current Dataverse organization. The publisher prefix is the customization prefix value for the Dataverse [solution publisher](https://learn.microsoft.com/en-us/power-platform/alm/solution-concepts-alm#solution-publisher). You can change it.

⚠️ If the push fails due to the size of the component, try the following to increase the file size limit:

- Go to [https://make.powerapps.com/](https://make.powerapps.com/).
- Click the **Settings** button at top right and open **Advanced Settings**. This opens a separate ....crm4.dynamics.com browser tab.
- In the Dynamics 365 tab, open the **Settings** dropdown. Under **System**, click on the **Administration** link.
- Click on **System Settings**. In the **Email** tab, under **Set File Size Limit for Attachments**, set the maximum file size to 32,120.

You should now be able to use the component in a Power Apps app.

## Adding the Bryntum Scheduler React component to the Power Apps app

Go back to the custom Power Apps app page that you created and do the following to add the Bryntum Scheduler component to your page:

- Click the **+** insert button on the left navigation.
- Click the **Get more components** button below the "Search" input in the **Insert** column.
- In the **Import components** panel, select the **Code** tab and select the "BryntumSchedulerComponent" component.
- Click the **Import** button.

![Import component](ritza_bryntum-powerapps-scheduler_import-component.png)

- In the **Insert** column, there will now be a **Code components** dropdown item. Click on it and select the "BryntumSchedulerComponent" component. This will add the custom Bryntum Scheduler component to the page:
- Resize the component, which will show a loading spinner, to take up the whole page.

![Imported component](ritza_bryntum-powerapps-scheduler_component-added-to-page.png)

- Click the "Save" button at the top-right.
- Once the save is complete, go back to the previous Bryntum Scheduler Power Apps app browser tab.
- Click the "Publish" button at the top-right of the screen.
- Click the "Play" button next to the "Publish" button to view the React Bryntum Scheduler Power Apps component:

![Bryntum Scheduler Power Apps component](ritza_bryntum-powerapps-scheduler_component-with-data.png)

⚠️ Note that you may need to refresh the page multiple times to show the updated page with the Bryntum Scheduler.

Now let's make the component save changes to the Bryntum Scheduler in the Dataverse tables.

## Creating a `syncData` function to sync data changes

Add the following `onDataChange` property to the `BryntumScheduler` React component in the `BryntumSchedulerComponent.tsx` file:

```ts
    onDataChange={syncData}
```

When a data change occurs in the Bryntum Scheduler, the [`dataChange`](https://www.bryntum.com/products/scheduler/docs/api/Grid/view/GridBase#event-dataChange) event will be fired and the `syncData` function will be called.

Define the `syncData` function in the `BryntumSchedulerComponent` component:

```ts
  const syncData = ({ store, action, records }: SyncData) => {
    const storeId = store.id;
    if (storeId === "events") {
      if (action === "add") {
      }
      if (action === "remove") {
      }
      if (action === "update") {
      }
    }
    if (storeId === "resources") {
      if (action === "add") {
      }
      if (action === "remove") {
      }
      if (action === "update") {
      }
    }
    if (storeId === "dependencies") {
      if (action === "add") {
      }
     if (action === "remove") {
      }
      if (action === "update") {
      }
    }
    if (storeId === "assignments") {
      if (action === "add") {
      }
      if (action === "remove") {
      }
      if (action === "update") {
      }
    }
  };
```

We get information about the `store`, `action`, and `records` from the `dataChange` event. The `store` is used to determine which data store has been changed: `"events"`, `"resources"`, `"dependencies"`, or `"assignments"`. The `action` determines the type of data change: `"add"`, `"remove"`, or `"update"`.
	
Now add the following type imports to the top of the `BryntumSchedulerComponent.tsx` file:
	 
```ts
import {
  SyncData,
  SchedulerEventDataverse,
  SchedulerResourceDataverse,
  SchedulerDependencyDataverse,
  SchedulerAssignmentDataverse,
} from "./BryntumSchedulerComponent.types";
```

## Creating Dataverse data using the Web API `createRecord` method

Add the following code to the `if` statement in the `syncData` function where `storeId === "events"` and `action === "add"` are:




