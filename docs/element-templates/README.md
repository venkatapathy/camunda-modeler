> The feature Element Templates may be subject to change in the future. As an example, this may lead to changes in the way you configure templates in JSON.


# Element Templates

Element templates allow you create pre-defined configurations for BPMN elements such as service and user tasks. Once applied via the properties panel they provide configured custom inputs to the user.

![Element template applied](./overview.png)


## Configuring Templates

To use element templates put a JSON file with the templates ([see definition below](#define-templates)) into the `resources/element-templates` directory relative to the modeler executable or the modelers data directory. Restart the modeler to let it recognize the templates.

The location of the modelers data directory differs across operating systems:
  **Windows**: `%APPDATA%/camunda-modeler`
  **Mac OS X**: `~/Library/Application Support/camunda-modeler`

> For example, add a JSON file to the following folder: `~/Library/Application Support/camunda-modeler/resources/element-templates` (Mac) or `%APPDATA%/camunda-modeler/resources/element-templates` (Windows).
You may have to create the `resources` and `element-templates` folders.


## Using Templates

If a template matches a selected diagram element, the template chooser will be shown in the properties panel.

![Template Chooser](./chooser.png)

Assigning a template will store it via the `camunda:modelerTemplate` property on the selected element:

```xml
<bpmn:serviceTask id="MailTask" camunda:modelerTemplate="com.mycompany.MailTask" />
```

It will also setup custom fields on the diagram element and make these available to the user for inspection and editing.


## Defining Templates

Templates are defined in template descriptor files as a JSON array:

```json
[
  {
    "name": "Template 1",
    "id": "sometemplate",
    "appliesTo": [
      "bpmn:ServiceTask"
    ],
    "properties": [
      ...
    ]
  },
  {
    "name": "Template 2",
    ...
  }
]
```

As seen in the code snippet a template consist of a number of important components:

* `name`: Identifier to appear in the template chooser
* `id`: Unique id of the template
* `appliesTo`: List of BPMN types the template can be assigned to
* `properties`: List of properties that are defined on the template
* `entriesVisible`: Visibility customizer for default properties panel entries


### Defining Template Properties

With each template you define a number of user-editable fields as well as their mapping to BPMN 2.0 XML as well as Camunda extension elements.

Let us consider the following example that defines a template for a mail sending task:

```json
{
  "name": "Mail Task",
  "id": "com.camunda.example.MailTask",
  "appliesTo": [
    "bpmn:ServiceTask"
  ],
  "properties": [
    {
      "label": "Implementation Type",
      "type": "String",
      "value": "com.mycompany.MailTaskImpl",
      "editable": false,
      "binding": {
        "type": "property",
        "name": "camunda:class"
      }
    },
    {
      "label": "Sender",
      "type": "String",
      "binding": {
        "type": "camunda:inputParameter",
        "name": "sender"
      },
      "constraints": {
        "notEmpty": true
      }
    },
    {
      "label": "Receivers",
      "type": "String",
      "binding": {
        "type": "camunda:inputParameter",
        "name": "receivers"
      },
      "constraints": {
        "notEmpty": true
      }
    },
    {
      "label": "Template",
      "description": "By the way, you can use freemarker templates ${...} here",
      "value": "Hello ${firstName}!",
      "type": "Text",
      "binding": {
        "type": "camunda:inputParameter",
        "name": "messageBody",
        "scriptFormat": "freemarker"
      },
      "constraints": {
        "notEmpty": true
      }
    },
    {
      "label": "Result Status",
      "description": "The process variable to which to assign the send result to",
      "type": "String",
      "value": "mailSendResult",
      "binding": {
        "type": "camunda:outputParameter",
        "source": "${ resultStatus }"
      }
    }
  ]
}
```

The example defines five custom fields, each mapped to different technical properties:

* _Implementation Type_ is mapped to the `camunda:class` property in BPMN 2.0 XML
* _Sender_, _Receivers_ and _Template_ properties are mapped to [input parameters]()
* _Result Status_ is mapped back from the Java Delegate into a process variable via an [output parameter]()

All but the _Implementation Type_ are editable by the user through the properties panel as shown in the following screenshot:

![Custom Fields](custom-fields.png)


As seen in the example the important attributes in a property definition are:

* `label`: A descriptive text shown with the property
* `type`: Defining the visual apperance in the properties panel (may be any of `String`, `Text`, `Boolean`, `Dropdown` or `Hidden`)
* `value`: An optional default value to be used if the property to be bound is not yet set
* `binding`: Specifying how the property is mapped to BPMN or Camunda extension elements and attributes (may be any of `property`, `camunda:property`, `camunda:inputParameter`, `camunda:outputParameter`, `camunda:in`, `camunda:out`, `camunda:executionListener`)
* `constraints`: A list of editing constraints to apply to the template


#### Types

The input types `String`, `Text`, `Boolean`, `Dropdown` and `Hidden` are available. As seen above `String` maps to a single-line input, `Text` maps to a multi-line input.


###### Boolean / Checkbox Type

The `Boolean` type maps to a checkbox that can be toggled by the user. It renders as shown below:

![Boolean / Checkbox control](field-boolean.png)


###### Dropdown Type

The `Dropdown` type allows users to select from a number of pre-defined options that are stored in a custom properties `choices` attribute as `{ name, value }` pairs:

```json
...
  "properties": [
    ...
    {
      "label": "Task Priority",
      "type": "Dropdown",
      "value": "50",
      "choices": [
        { "name": "low", "value": "20" },
        { "name": "medium", "value": "50" },
        { "name": "height", "value": "100" }
      ]
    }
  ]
...
```

The resulting properties panel control looks like this:

![properties panel drop down](field-dropdown.png)


#### Bindings

The following ways exist to map a custom field to the underlying BPMN 2.0 XML:

* `property`: Maps to a named property in the BPMN 2.0 XML, i.e. `<bpmn:serviceTask {name}={userInput} />`
* `camunda:property`: Maps to a `<camunda:property name="{name}" value="{userInput}" />` extension element
* `camunda:inputParameter`: Maps to `<camunda:inputParameter name="{name}" />`
* `camunda:outputParameter`: Maps to `<camunda:outputParameter name="{userInput}" />`
* `camunda:in`: Maps to `<camunda:in target="{target}" />`
* `camunda:out`: Maps to `<camunda:out source="{source}" />`
* `camunda:executionListener`: Maps to `<camunda:executionListener event="{event}" />`

#### Constraints

Custom Fields may have a number of constraints associated with them:

* `notEmpty`: Input must be non-empty
* `minLength`: Minimal length for the input
* `maxLength`: Maximal length for the input
* `pattern`: Regular expression to match the input against


##### Regular Expression

Together with the `pattern` constraint you may define your custom error messages:

```json
...
  "properties": [
    {
      "label": "Web service URL",
      "type": "String",
      "binding": { ... },
      "constraints": {
        "notEmpty": true,
        "pattern": {
          "value": "https://.*",
          "message": "Must be https URL"
        }
      }
    }
  ]
```


### Controling Default Entry Visibility

_TODO_


## Development Workflow

When creating custom element templates the modeler will give you detailed validation error messages.

Templates will be loaded on application load and reload. To reload the application with updated templates, open the developer tools `F12` and press `CtrlOrCmd+R`. This will clear all unsaved diagrams **!**


## Supported BPMN Types

Currently element templates may be used on the following BPMN elements:

* `bpmn:Activity` (including tasks, service tasks and others)
* `bpmn:SequenceFlow` (for maintaining `condition`)
* `bpmn:Process`


## Learn more

Try playing around with custom elements and some [example templates](https://github.com/camunda/camunda-modeler/blob/master/resources/element-templates/samples.json).

If you get stuck, ask for help [in our forums](https://forum.camunda.org/c/modeler).
