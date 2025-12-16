# ValueHelperWrapper Control - User Manual

## Overview

The `ValueHelperWrapper` is a custom SAP UI5 control that provides an enhanced value help dialog functionality. It wraps the standard `sap.m.MultiInput` control and extends it with powerful filtering, selection, and configuration capabilities through a programmatically created value help dialog. The control now includes **two-way data binding** support for automatic token synchronization with your JSON model.

## Features

- **Two-Way Data Binding**: Automatically synchronize tokens with your JSON model
- **Dynamic Configuration**: Fully configurable through a JSON configuration object
- **Smart Filtering**: Built-in filter bar with customizable filter fields
- **Single/Multi Selection**: Support for both single and multiple selection modes
- **Responsive Design**: Works on desktop, tablet, and mobile devices
- **Busy State Management**: Smart loading indicators with configurable delays
- **Token Management**: Full token support for selected values
- **Hidden Mode**: Can be rendered as invisible for programmatic use only

## Installation

1. Replace `yourAppId` in the control definition with your actual application ID:
```javascript
return Control.extend("yourAppId.controls.ValueHelperWrapper", {
```

2. Ensure the control file is placed in your project's controls folder (e.g., `webapp/controls/ValueHelperWrapper.js`)

3. Load the control in your view or controller:
```javascript
sap.ui.define([
    "yourAppId/controls/ValueHelperWrapper"
], function(ValueHelperWrapper) {
    // Your code here
});
```

## Configuration

### Basic Configuration Object Structure

The control is configured through a comprehensive configuration object that defines the data source, fields, and behavior:

```javascript
var valueHelpConfig = {
    entitySet: "/ValueHelpSet",
    filters: [
        new Filter("EntitySet", FilterOperator.EQ, "Country")
    ],
    fields: [{
        code: "Vhkey",
        label: this.getResourceBundle().getText("countryId"),
        filter: true
    }, {
        code: "Text",
        label: this.getResourceBundle().getText("countryAr"),
        filter: true
    }, {
        code: "AdditionText1",
        label: this.getResourceBundle().getText("countryEn"),
        filter: true
    }],
    selectedKey: "Vhkey",
    selectedDescription: "AdditionText1",
    
    // NEW: Optional binding configuration
    binding: {
        path: "/selectedCountry",
        idProperty: "id",
        descProperty: "desc",
        multiSelect: false,
        modelName: "viewModel"
    }
};
```

### Configuration Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `entitySet` | String | Yes | The OData entity set path (e.g., "/ValueHelpSet") |
| `filters` | Filter[] | No | Array of initial filters to apply to the data |
| `fields` | Object[] | Yes | Array of field definitions for table columns |
| `selectedKey` | String | Yes | Property name to use as the token key |
| `selectedDescription` | String | Yes | Property name to use as the token display text |
| `binding` | Object | No | Binding configuration for automatic token synchronization |

### Field Configuration

Each field object in the `fields` array supports:

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `code` | String | Yes | The OData property name |
| `label` | String | Yes | Display label for the column header |
| `filter` | Boolean | No | Whether this field should appear in the filter bar |

### Binding Configuration (Optional)

The `binding` object enables automatic synchronization between tokens and your model:

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `path` | String | Yes | - | The binding path in your model (e.g., `/selectedCountry`) |
| `idProperty` | String | Yes | - | The property name for the token key in your model object |
| `descProperty` | String | Yes | - | The property name for the token text in your model object |
| `multiSelect` | Boolean | No | `false` | Set to `true` if binding to an array for multi-selection |
| `modelName` | String | No | `"viewModel"` | The name of the model to bind to |

## Usage Examples

### 1. Basic Setup in Controller (Without Binding)

```javascript
onInit: function() {
    // Define your configuration
    var valueHelpConfig = {
        entitySet: "/ValueHelpSet",
        filters: [
            new Filter("EntitySet", FilterOperator.EQ, "Country")
        ],
        fields: [{
            code: "Vhkey",
            label: this.getResourceBundle().getText("countryId"),
            filter: true
        }, {
            code: "Text",
            label: this.getResourceBundle().getText("countryAr"),
            filter: true
        }, {
            code: "AdditionText1",
            label: this.getResourceBundle().getText("countryEn"),
            filter: true
        }],
        selectedKey: "Vhkey",
        selectedDescription: "AdditionText1"
    };

    // Create and set the view model
    this.oViewModel = new JSONModel({
        busy: false,
        delay: 100,
        countryConfig: valueHelpConfig
    });

    this.getView().setModel(this.oViewModel, "viewModel");
}
```

### 2. Setup with Two-Way Binding (Single Selection)

```javascript
onInit: function() {
    var valueHelpConfig = {
        entitySet: "/CountrySet",
        fields: [{
            code: "CountryId",
            label: "Country ID",
            filter: true
        }, {
            code: "CountryName",
            label: "Country Name",
            filter: true
        }],
        selectedKey: "CountryId",
        selectedDescription: "CountryName",
        
        // Enable binding for automatic synchronization
        binding: {
            path: "/selectedCountry",
            idProperty: "id",
            descProperty: "name",
            multiSelect: false
        }
    };

    var oViewModel = new JSONModel({
        countryConfig: valueHelpConfig,
        selectedCountry: {
            id: "US",
            name: "United States"
        }
    });

    this.getView().setModel(oViewModel, "viewModel");
}
```

### 3. Setup with Two-Way Binding (Multi-Selection)

```javascript
onInit: function() {
    var valueHelpConfig = {
        entitySet: "/SupplierSet",
        fields: [{
            code: "SupplierId",
            label: "Supplier ID",
            filter: true
        }, {
            code: "SupplierName",
            label: "Supplier Name",
            filter: true
        }],
        selectedKey: "SupplierId",
        selectedDescription: "SupplierName",
        
        // Enable binding for multi-selection
        binding: {
            path: "/selectedSuppliers",
            idProperty: "id",
            descProperty: "name",
            multiSelect: true  // Array binding
        }
    };

    var oViewModel = new JSONModel({
        supplierConfig: valueHelpConfig,
        selectedSuppliers: [
            { id: "SUP001", name: "Supplier A" },
            { id: "SUP002", name: "Supplier B" }
        ]
    });

    this.getView().setModel(oViewModel, "viewModel");
}
```

### 4. XML View Declaration

```xml
<yourNamespace:ValueHelperWrapper
    id="countryValueHelper"
    config="{viewModel>/countryConfig}"
    busy="{viewModel>/busy}"
    busyIndicatorDelay="{viewModel>/delay}"
    singleMode="true"
    selectionChange="onCountrySelectionChange"
    editable="true"
    width="100%"
    placeholder="Select Country..." />
```

### 5. Programmatic Usage

```javascript
// Open value help dialog programmatically
onOpenValueHelp: function() {
    var oValueHelper = this.byId("countryValueHelper");
    oValueHelper.openValueHelpDialog();
},

// Handle selection changes
onCountrySelectionChange: function(oEvent) {
    var aSelectedTokens = oEvent.getParameter("selectedTokens");
    console.log("Selected tokens:", aSelectedTokens);
    
    // Process selected values
    aSelectedTokens.forEach(function(oToken) {
        console.log("Key:", oToken.getKey());
        console.log("Text:", oToken.getText());
    });
},

// Clear selections
onClearSelection: function() {
    var oValueHelper = this.byId("countryValueHelper");
    oValueHelper.clearTokens();
},

// Get current tokens
onGetCurrentSelection: function() {
    var oValueHelper = this.byId("countryValueHelper");
    var aTokens = oValueHelper.getTokens();
    return aTokens;
}
```

### 6. Programmatic Model Updates (With Binding)

```javascript
// Update the model - control automatically reflects changes
onUpdateCountry: function() {
    var oViewModel = this.getView().getModel("viewModel");
    
    oViewModel.setProperty("/selectedCountry", {
        id: "DE",
        name: "Germany"
    });
    // Control tokens are automatically updated!
},

// Clear selection
onClearCountry: function() {
    var oViewModel = this.getView().getModel("viewModel");
    oViewModel.setProperty("/selectedCountry", null);
    // Control tokens are automatically cleared!
}
```

## Control Properties

### Custom Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `config` | Object | {} | Configuration object defining behavior and data |
| `hidden` | Boolean | false | Renders control as invisible but accessible |
| `singleMode` | Boolean | false | Enables single selection mode |
| `busy` | Boolean | false | Shows busy indicator on the table |
| `busyIndicatorDelay` | Integer | 1000 | Delay before showing busy indicator (ms) |

### Inherited Properties

The control inherits all properties from `sap.m.MultiInput`, including:
- `editable`
- `enabled` 
- `width`
- `placeholder`
- `value`
- And many more...

## Events

### Custom Events

| Event | Parameters | Description |
|-------|------------|-------------|
| `selectionChange` | `selectedTokens: sap.m.Token[]` | Fired when user confirms selection in the dialog |

### Inherited Events

All `sap.m.MultiInput` events are supported except `valueHelpRequest` (handled internally).

## Hidden Property Usage

The `hidden` property is a powerful feature that allows you to render the ValueHelperWrapper control as invisible while keeping it functional. This enables you to trigger value help dialogs from other UI elements like buttons, combo boxes, or any custom controls.

### Setting up Hidden Mode

```xml
<!-- Hidden ValueHelperWrapper - invisible but functional -->
<yourNamespace:ValueHelperWrapper
    id="hiddenCountryHelper"
    config="{viewModel>/countryConfig}"
    hidden="true"
    singleMode="true"
    selectionChange="onCountrySelectionChange" />

<!-- Visible trigger button -->
<Button
    id="openCountryDialogBtn"
    text="Select Country"
    press="onOpenCountryDialog"
    icon="sap-icon://value-help" />
```

### Triggering from Buttons

```javascript
// Controller method to trigger value help from button
onOpenCountryDialog: function() {
    var oHiddenValueHelper = this.byId("hiddenCountryHelper");
    oHiddenValueHelper.openValueHelpDialog();
},

// Handle selection from hidden control
onCountrySelectionChange: function(oEvent) {
    var aSelectedTokens = oEvent.getParameter("selectedTokens");
    
    if (aSelectedTokens.length > 0) {
        var oSelectedToken = aSelectedTokens[0];
        
        // Update your UI with selected value
        var oButton = this.byId("openCountryDialogBtn");
        oButton.setText("Country: " + oSelectedToken.getText());
        
        // Store selected key for business logic
        this.getModel("viewModel").setProperty("/selectedCountryKey", oSelectedToken.getKey());
    }
}
```

### Benefits of Hidden Mode

1. **Flexible UI Design**: Trigger value help from any UI element without being constrained by the MultiInput appearance
2. **Space Optimization**: Save screen real estate by not showing the input field
3. **Custom Interactions**: Create unique user experiences like tile-based selection or wizard-style flows
4. **Conditional Value Help**: Show different value helps based on user selections or application state
5. **Integration Flexibility**: Easily integrate with existing forms and controls

## Advanced Configuration Examples

### 1. Multi-Selection with Custom Filters

```javascript
var multiSelectConfig = {
    entitySet: "/EmployeeSet",
    filters: [
        new Filter("Department", FilterOperator.EQ, "IT"),
        new Filter("Status", FilterOperator.EQ, "Active")
    ],
    fields: [{
        code: "EmployeeId",
        label: "Employee ID",
        filter: true
    }, {
        code: "FirstName", 
        label: "First Name",
        filter: true
    }, {
        code: "LastName",
        label: "Last Name", 
        filter: true
    }, {
        code: "Email",
        label: "Email",
        filter: false
    }],
    selectedKey: "EmployeeId",
    selectedDescription: "FirstName",
    binding: {
        path: "/selectedEmployees",
        idProperty: "id",
        descProperty: "name",
        multiSelect: true
    }
};
```

### 2. Single Selection without Initial Filters

```javascript
var singleSelectConfig = {
    entitySet: "/ProductSet",
    fields: [{
        code: "ProductCode",
        label: "Product Code",
        filter: true
    }, {
        code: "ProductName",
        label: "Product Name",
        filter: true
    }, {
        code: "Category",
        label: "Category",
        filter: true
    }],
    selectedKey: "ProductCode", 
    selectedDescription: "ProductName",
    binding: {
        path: "/selectedProduct",
        idProperty: "code",
        descProperty: "name",
        multiSelect: false
    }
};
```

## Best Practices

1. **Resource Bundle Usage**: Always use resource bundles for labels to support internationalization
2. **Filter Optimization**: Only set `filter: true` for fields that users commonly search by
3. **Key Selection**: Choose unique, stable properties for `selectedKey`
4. **Description Selection**: Use human-readable properties for `selectedDescription`
5. **Busy State**: Configure appropriate `busyIndicatorDelay` based on expected response times
6. **Memory Management**: The control automatically cleans up dialog resources on close
7. **Binding Configuration**: Use binding when you need automatic synchronization; use manual token management for more control
8. **Model Structure**: When using binding, ensure your model objects match the `idProperty` and `descProperty` names

## Troubleshooting

### Common Issues

1. **Control not rendering**: Ensure `yourAppId` is correctly replaced in the control definition
2. **No data showing**: Verify the `entitySet` path and any initial `filters`
3. **Columns not displaying**: Check that field `code` values match OData property names
4. **Selection not working**: Ensure `selectedKey` property exists in your data
5. **Filtering not working**: Verify that fields with `filter: true` have valid property names
6. **Binding not synchronizing**: Check that the binding path exists in your model and property names match

### Debug Tips

```javascript
// Log current configuration
console.log("Config:", oControl.getConfig());

// Check current tokens
console.log("Tokens:", oControl.getTokens());

// Monitor selection changes
oControl.attachSelectionChange(function(oEvent) {
    console.log("Selection changed:", oEvent.getParameters());
});

// Check binding configuration
console.log("Binding config:", oControl.getConfig().binding);

// Verify model data
var oViewModel = this.getView().getModel("viewModel");
console.log("Model data:", oViewModel.getData());
```

## Browser Support

The control supports all browsers supported by SAP UI5, including:
- Chrome (latest)
- Firefox (latest) 
- Safari (latest)
- Edge (latest)
- Internet Explorer 11+ (with UI5 compatibility)

## Version Compatibility

- SAP UI5 1.60+
- Tested with UI5 versions up to 1.120+
- Compatible with both Fiori 2.0 and Fiori 3.0 themes

---

## Public API Reference

### Token Management Methods

#### `getTokens()`
Returns all currently selected tokens.

**Signature:**
```javascript
getTokens() → sap.m.Token[]
```

**Example:**
```javascript
var aTokens = oControl.getTokens();
console.log("Token count:", aTokens.length);
```

---

#### `setTokens(aTokens)`
Sets the tokens for the control, replacing any existing tokens.

**Signature:**
```javascript
setTokens(aTokens: sap.m.Token[]) → this
```

**Parameters:**
- `aTokens` (sap.m.Token[]): Array of Token objects to set

**Example:**
```javascript
var oToken = new sap.m.Token({
    key: "US",
    text: "United States"
});
oControl.setTokens([oToken]);
```

---

#### `addToken(oToken)`
Adds a single token to the control.

**Signature:**
```javascript
addToken(oToken: sap.m.Token) → this
```

**Parameters:**
- `oToken` (sap.m.Token): The token to add

**Example:**
```javascript
var oToken = new sap.m.Token({
    key: "DE",
    text: "Germany"
});
oControl.addToken(oToken);
```

---

#### `removeToken(oToken)`
Removes a specific token from the control.

**Signature:**
```javascript
removeToken(oToken: sap.m.Token) → this
```

**Parameters:**
- `oToken` (sap.m.Token): The token to remove

**Example:**
```javascript
var aTokens = oControl.getTokens();
if (aTokens.length > 0) {
    oControl.removeToken(aTokens[0]);
}
```

---

#### `removeAllTokens()`
Removes all tokens from the control.

**Signature:**
```javascript
removeAllTokens() → this
```

**Example:**
```javascript
oControl.removeAllTokens();
```

---

#### `clearTokens()`
Clears all tokens from the control (alias for `removeAllTokens()`).

**Signature:**
```javascript
clearTokens() → this
```

**Example:**
```javascript
oControl.clearTokens();
```

---

### Dialog Management Methods

#### `openValueHelpDialog()`
Opens the value help dialog programmatically.

**Signature:**
```javascript
openValueHelpDialog() → void
```

**Example:**
```javascript
oControl.openValueHelpDialog();
```

---

### Property Management Methods

#### `setBusy(bBusy)`
Sets the busy state of the control's table.

**Signature:**
```javascript
setBusy(bBusy: boolean) → this
```

**Parameters:**
- `bBusy` (boolean): `true` to show busy indicator, `false` to hide it

**Example:**
```javascript
oControl.setBusy(true);
// ... perform operation
oControl.setBusy(false);
```

---

#### `getBusy()`
Returns the current busy state.

**Signature:**
```javascript
getBusy() → boolean
```

**Example:**
```javascript
if (oControl.getBusy()) {
    console.log("Control is busy");
}
```

---

#### `setBusyIndicatorDelay(iDelay)`
Sets the delay before showing the busy indicator.

**Signature:**
```javascript
setBusyIndicatorDelay(iDelay: integer) → this
```

**Parameters:**
- `iDelay` (integer): Delay in milliseconds

**Example:**
```javascript
oControl.setBusyIndicatorDelay(500);
```

---

#### `getBusyIndicatorDelay()`
Returns the busy indicator delay.

**Signature:**
```javascript
getBusyIndicatorDelay() → integer
```

**Example:**
```javascript
var iDelay = oControl.getBusyIndicatorDelay();
console.log("Busy delay:", iDelay, "ms");
```

---

#### `setEditable(bEditable)`
Sets whether the control is editable.

**Signature:**
```javascript
setEditable(bEditable: boolean) → this
```

**Parameters:**
- `bEditable` (boolean): `true` to make editable, `false` to make read-only

**Example:**
```javascript
oControl.setEditable(false);
```

---

#### `getEditable()`
Returns whether the control is editable.

**Signature:**
```javascript
getEditable() → boolean
```

**Example:**
```javascript
if (oControl.getEditable()) {
    console.log("Control is editable");
}
```

---

#### `setConfig(oConfig)`
Sets the configuration object for the control.

**Signature:**
```javascript
setConfig(oConfig: object) → this
```

**Parameters:**
- `oConfig` (object): Configuration object with entitySet, fields, selectedKey, etc.

**Example:**
```javascript
var oNewConfig = {
    entitySet: "/NewSet",
    fields: [...],
    selectedKey: "Id",
    selectedDescription: "Name"
};
oControl.setConfig(oNewConfig);
```

---

#### `getConfig()`
Returns the current configuration object.

**Signature:**
```javascript
getConfig() → object
```

**Example:**
```javascript
var oConfig = oControl.getConfig();
console.log("Entity set:", oConfig.entitySet);
```

---

#### `setSingleMode(bSingleMode)`
Sets whether the control operates in single selection mode.

**Signature:**
```javascript
setSingleMode(bSingleMode: boolean) → this
```

**Parameters:**
- `bSingleMode` (boolean): `true` for single selection, `false` for multi-selection

**Example:**
```javascript
oControl.setSingleMode(true);
```

---

#### `getSingleMode()`
Returns whether the control is in single selection mode.

**Signature:**
```javascript
getSingleMode() → boolean
```

**Example:**
```javascript
if (oControl.getSingleMode()) {
    console.log("Single selection mode enabled");
}
```

---

#### `setHidden(bHidden)`
Sets whether the control is hidden (invisible but functional).

**Signature:**
```javascript
setHidden(bHidden: boolean) → this
```

**Parameters:**
- `bHidden` (boolean): `true` to hide, `false` to show

**Example:**
```javascript
oControl.setHidden(true);
```

---

#### `getHidden()`
Returns whether the control is hidden.

**Signature:**
```javascript
getHidden() → boolean
```

**Example:**
```javascript
if (oControl.getHidden()) {
    console.log("Control is hidden");
}
```

---

### Event Methods

#### `attachSelectionChange(fnFunction, oListener)`
Attaches a handler to the `selectionChange` event.

**Signature:**
```javascript
attachSelectionChange(fnFunction: function, oListener?: object) → this
```

**Parameters:**
- `fnFunction` (function): Handler function
- `oListener` (object, optional): Context for the handler

**Example:**
```javascript
oControl.attachSelectionChange(function(oEvent) {
    var aTokens = oEvent.getParameter("selectedTokens");
    console.log("Selection changed:", aTokens);
});
```

---

#### `detachSelectionChange(fnFunction, oListener)`
Detaches a handler from the `selectionChange` event.

**Signature:**
```javascript
detachSelectionChange(fnFunction: function, oListener?: object) → this
```

**Parameters:**
- `fnFunction` (function): Handler function to remove
- `oListener` (object, optional): Context for the handler

**Example:**
```javascript
oControl.detachSelectionChange(myHandler);
```

---

#### `fireSelectionChange(mArguments)`
Fires the `selectionChange` event manually.

**Signature:**
```javascript
fireSelectionChange(mArguments?: object) → this
```

**Parameters:**
- `mArguments` (object, optional): Event parameters including `selectedTokens`

**Example:**
```javascript
oControl.fireSelectionChange({
    selectedTokens: oControl.getTokens()
});
```

---

## Summary

The ValueHelperWrapper control provides a comprehensive solution for value help dialogs in SAP UI5 applications. With support for both manual token management and automatic two-way binding, it offers flexibility for various use cases while maintaining a clean and intuitive API.

