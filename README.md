# ValueHelperWrapper Control - User Manual

## Overview

The `ValueHelperWrapper` is a custom SAP UI5 control that provides an enhanced value help dialog functionality. It wraps the standard `sap.m.MultiInput` control and extends it with powerful filtering, selection, and configuration capabilities. The control now includes **two-way data binding** and support for attaching **custom data fields** to tokens.

## Features

- **Two-Way Data Binding**: Automatically synchronize tokens with your JSON model.
- **Custom Data Fields**: Attach any number of extra data fields to tokens for use in your application.
- **Dynamic Configuration**: Fully configurable through a JSON configuration object.
- **Smart Filtering**: Built-in filter bar with customizable filter fields.
- **Single/Multi Selection**: Support for both single and multiple selection modes.
- **Responsive Design**: Works on desktop, tablet, and mobile devices.
- **Busy State Management**: Smart loading indicators with configurable delays.
- **Token Management**: Full token support for selected values.
- **Hidden Mode**: Can be rendered as invisible for programmatic use only.

## Installation

1.  Replace `yourAppId` in the control definition with your actual application ID:
    ```javascript
    return Control.extend("yourAppId.controls.ValueHelperWrapper", {
    ```

2.  Ensure the control file is placed in your project's controls folder (e.g., `webapp/controls/ValueHelperWrapper.js`)

3.  Load the control in your view or controller:
    ```javascript
    sap.ui.define([
        "yourAppId/controls/ValueHelperWrapper"
    ], function(ValueHelperWrapper) {
        // Your code here
    });
    ```

## Configuration

### Basic Configuration Object Structure

The control is configured through a comprehensive configuration object:

```javascript
var valueHelpConfig = {
    entitySet: "/Suppliers",
    fields: [
        { code: "SupplierID", label: "Supplier ID", filter: true },
        { code: "CompanyName", label: "Company Name", filter: true },
        { code: "Country", label: "Country" }
    ],
    selectedKey: "SupplierID",
    selectedDescription: "CompanyName",
    
    // NEW: Define custom data fields to attach to tokens
    customDataFields: ["Country"],
    
    // Optional binding configuration
    binding: {
        path: "/selectedSuppliers",
        idProperty: "id",
        descProperty: "name",
        multiSelect: true,
        modelName: "viewModel",
        // NEW: Also specify custom data fields for binding
        customDataFields: ["Country"]
    }
};
```

### Configuration Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `entitySet` | String | Yes | The OData entity set path (e.g., "/ValueHelpSet"). |
| `filters` | Filter[] | No | Array of initial filters to apply to the data. |
| `fields` | Object[] | Yes | Array of field definitions for table columns. |
| `selectedKey` | String | Yes | Property name to use as the token key. |
| `selectedDescription` | String | Yes | Property name to use as the token display text. |
| `customDataFields` | String[] | No | Array of extra field names to attach to tokens. |
| `binding` | Object | No | Binding configuration for automatic token synchronization. |

### Field Configuration

Each field object in the `fields` array supports:

| Property | Type | Required | Description |
|---|---|---|---|
| `code` | String | Yes | The OData property name. |
| `label` | String | Yes | Display label for the column header. |
| `filter` | Boolean | No | Whether this field should appear in the filter bar. |

### Binding Configuration (Optional)

The `binding` object enables automatic synchronization between tokens and your model:

| Property | Type | Required | Default | Description |
|---|---|---|---|---|
| `path` | String | Yes | - | The binding path in your model (e.g., `/selectedCountry`). |
| `idProperty` | String | Yes | - | The property name for the token key in your model object. |
| `descProperty` | String | Yes | - | The property name for the token text in your model object. |
| `multiSelect` | Boolean | No | `false` | Set to `true` if binding to an array for multi-selection. |
| `modelName` | String | No | `"viewModel"` | The name of the model to bind to. |
| `customDataFields` | String[] | No | `[]` | Array of extra field names to include in the model data. |

## Working with Custom Data

The `customDataFields` property allows you to attach extra data from your entity set to the tokens. This is useful when you need more than just the key and text of the selected item.

### 1. Configuration

Add the `customDataFields` array to both the main configuration and the `binding` object. The array should contain the names of the fields from your entity set that you want to store in the tokens.

```javascript
var valueHelpConfig = {
    entitySet: "/Suppliers",
    fields: [
        { code: "SupplierID", label: "Supplier ID" },
        { code: "CompanyName", label: "Company Name" },
        { code: "Country", label: "Country" },
        { code: "ContactName", label: "Contact Name" }
    ],
    selectedKey: "SupplierID",
    selectedDescription: "CompanyName",
    
    // Specify which extra fields to attach to the tokens
    customDataFields: ["Country", "ContactName"],
    
    binding: {
        path: "/selectedSuppliers",
        idProperty: "id",
        descProperty: "name",
        multiSelect: true,
        // Also specify the fields for the model data
        customDataFields: ["Country", "ContactName"]
    }
};
```

### 2. Retrieving Custom Data

In your `selectionChange` event handler, you can retrieve the custom data using the `oToken.data()` method. This returns a JavaScript object containing all the custom fields you defined.

```javascript
onSupplierSelectionChange: function(oEvent) {
    var aSelectedTokens = oEvent.getParameter("selectedTokens");
    
    if (aSelectedTokens.length > 0) {
        var oToken = aSelectedTokens[0];
        
        // Get the custom data object from the token
        var oCustomData = oToken.data();
        
        console.log("Token Key:", oToken.getKey());
        console.log("Token Text:", oToken.getText());
        
        // Access your custom fields directly
        console.log("Country:", oCustomData.Country);
        console.log("Contact Name:", oCustomData.ContactName);
    }
}
```

## XML View Declaration

```xml
<yourNamespace:ValueHelperWrapper
    id="supplierValueHelper"
    config="{viewModel>/supplierConfig}"
    busy="{viewModel>/busy}"
    singleMode="false"
    selectionChange="onSupplierSelectionChange"
    editable="true"
    width="100%"
    placeholder="Select Suppliers..." />
```

## Control Properties

### Custom Properties

| Property | Type | Default | Description |
|---|---|---|---|
| `config` | Object | {} | Configuration object defining behavior and data. |
| `hidden` | Boolean | false | Renders control as invisible but accessible. |
| `singleMode` | Boolean | false | Enables single selection mode. |
| `busy` | Boolean | false | Shows busy indicator on the table. |
| `busyIndicatorDelay` | Integer | 1000 | Delay before showing busy indicator (ms). |

### Inherited Properties

The control inherits all properties from `sap.m.MultiInput`, including `editable`, `enabled`, `width`, `placeholder`, etc.

## Events

### Custom Events

| Event | Parameters | Description |
|---|---|---|
| `selectionChange` | `selectedTokens` (sap.m.Token[]) | Fired when the selection of tokens changes. |

### Inherited Events

The control inherits all events from `sap.m.MultiInput` (except `valueHelpRequest`).
