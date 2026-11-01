# Glossary

> **Reference.** Power Platform and project words you will meet, in alphabetical order.

| Term | Meaning |
|---|---|
| **appaction** | The Dataverse table that stores a modern command bar button of a model-driven app. The [Command Buttons tool](tools/command-buttons.md) writes these rows. |
| **Associate / Disassociate** | Link or unlink two rows through an N:N relationship (`associate()` in the core). |
| **Business rule** | No-code logic on a form or table (show/hide, set value, validate). Runs alongside your scripts. |
| **Business unit** | A node in the organization tree; security role depth ("Business Unit", "Organization") is measured against it. |
| **Choice** | A column with fixed options: integer value + label. Old names: option set, picklist. A *global* choice is shared by several columns. |
| **Client API / Xrm** | The JavaScript API available in model-driven apps: `Xrm.WebApi`, `Xrm.Navigation`, `Xrm.Utility`, `formContext`. |
| **Command bar / command** | The buttons above a form or view. A command runs a JavaScript function from a web resource (or Power Fx). |
| **Component (React)** | A function that returns JSX for some props. A page is a component made of smaller ones (`Field`, `Row`, `SaveBar`). |
| **Connection reference** | A named slot in a solution that a flow uses; at import you fill it with a connection in that environment. |
| **Connection** | A signed-in account for a connector (your Outlook, a SharePoint account). Belongs to a person; not in solutions. |
| **Connector** | A wrapper around a service's API for flows: Dataverse, SharePoint, Outlook, Teams... Standard or premium. |
| **Dataverse** | The data platform: tables, security, Web API. Old name: Common Data Service (still in connector ids: `shared_commondataserviceforapps`). |
| **deploy.json** | `dist/deploy.json`, written by `pnpm build`: every web resource and command button of the build, read by `pnpm dv:verify` and the Command Buttons tool ([format](configuration.md#distdeployjson)). |
| **Display / logical / schema name** | The names of a table or column; see [Power Platform](../1-learn/01-power-platform.md#names-the-thing-that-trips-everyone). |
| **Entity set name** | The plural used in Web API URLs and `@odata.bind` (`accounts`, `pidev_addresses`). |
| **Entity** | Old name for table. Still in APIs (`EntityDefinitions`, `entityName`). |
| **Environment** | One Dataverse org with its own URL, data and apps (dev, test, prod). |
| **FetchXML** | Dataverse's XML query language; views are stored as FetchXML. `pnpm dv:q` runs it. |
| **Flow Run** | The Dataverse table (`flowrun`) that keeps the runs of solution flows, 28 days by default. The Runs tab of [Flows as Code](tools/flows-as-code.md#runs-tab) reads it. |
| **Form** | A record's edit screen in a model-driven app, designed in the form designer. Forms have OnLoad/OnSave and column OnChange events. |
| **formContext** | The current form in the Client API: its record, columns and controls. Commands receive it as `PrimaryControl`. |
| **Hook (React)** | A function whose name starts with `use` (`useState`, `useEffect`, `useSaveBar`). Components call hooks to keep state or to talk to the outside. |
| **JSX** | The HTML-like syntax inside `.tsx` files. The build turns it into function calls. |
| **Kit** | The shared files every project made from the starter has: core, UI kit, flowkit, tools, tests, configs, shared docs. Listed in `kit.json`; taken with `pnpm kit:update` ([Update the kit](../7-maintain/update-the-kit.md)); a fix made in a project goes back with `pnpm kit:push --pr` ([Send kit changes back](../7-maintain/send-kit-changes-back.md)). |
| **Links file** | A file in the repo that links local files to what they become in Dataverse, with relative paths, so the team shares it: `webresources.links.xml` (web resources) and `flows.links.json` (flows). |
| **Lookup** | A column pointing to a row in another table (a foreign key). |
| **Managed / unmanaged solution** | Unmanaged: editable, used in dev. Managed: locked, imported in test/prod, uninstallable as a unit. |
| **Model-driven app** | An app whose screens are generated from table metadata (forms, views, sitemap). |
| **N:N relationship** | Many-to-many between two tables, stored in a hidden intersect table. |
| **Navigation property** | The name used to write a lookup (`pidev_Manager@odata.bind`). Case-sensitive. |
| **pac** | Power Platform CLI, Microsoft's command-line tool. We use it to sign in and run queries. |
| **Plugin** | C# code registered on Dataverse events (create, update...). Runs server-side; may change or reject your writes. Not the same as an XrmToolBox tool, also called a plugin. |
| **Power Automate (cloud flow)** | Workflows with triggers and actions, built on Azure Logic Apps. |
| **Power Fx** | Excel-like formula language of canvas apps and modern commands. Not used in this starter. |
| **Primary column / primary name** | The column shown as the row's name in lookups and lists (`name`, `fullname`, `subject`). |
| **PrimaryControl** | The command parameter that passes `formContext` to your function. |
| **Props** | The inputs of a React component: `<TextField spec value onChange />`. |
| **Publish / Publish all customizations** | Makes changes to forms, web resources, apps visible to users. Forgetting it is the classic "my change does nothing". |
| **Publisher / prefix** | The owner of a solution's components. Its prefix (`pidev`) starts every custom name; its option value prefix starts custom choice values. |
| **Reducer** | A function `(state, action) → new state`. Keep it outside the component (a plain function, tested without React) when a form has rules between its fields. |
| **Release** | A checked zip of the project, made with `pnpm release` from a clean copy, with its notes ([Make a release](../6-ship/release.md)). |
| **Security role** | A set of privileges (create, read, write, delete, append, append to, assign, share) per table, each at a depth (user, BU, parent-child BU, organization). |
| **Sitemap** | The app's left navigation. |
| **Solution** | The package that carries customizations between environments. |
| **State (React)** | What a component remembers between renders (`useState`, `useReducer`). The screen is drawn from it. |
| **Status / status reason** | `statecode` (Active 0 / Inactive 1) and `statuscode` (a reason per state). |
| **StrictMode** | A React wrapper that runs effects twice in development to expose code that is not safe to repeat. It does nothing in production. |
| **Team (owner / group team)** | A group of users that can own rows and hold security roles. A group team mirrors an Entra ID group. |
| **View** | A saved list of rows (columns, filter, sort) on a table. Stored as FetchXML. |
| **Web API** | Dataverse's OData v4 REST API at `/api/data/v9.2/`. |
| **Web resource** | A file (HTML, JS, CSS, image) stored in Dataverse and served from the org URL. |
| **XrmToolBox** | A Windows app that hosts tools for Dataverse. The team uses [three of them](tools.md) to push to dev. |
