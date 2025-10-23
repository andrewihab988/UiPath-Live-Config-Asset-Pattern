# UiPath Live Config Asset Pattern

A UiPath template and guide for moving project configurations from static `Config.xlsx` files to a single, dynamic JSON Text Asset in Orchestrator. This pattern allows you to update your bot's settings in real-time without redeploying the package.

## The Problem: Static Configs are Inefficient

The traditional `Config.xlsx` file is a major bottleneck in a mature RPA environment. Every time a simple, environment-specific setting needs to be changed—such as a target URL, a shared folder path, a retry count, or a feature flag—the entire automation package must be:

1.  Opened in Studio.
2.  Edited.
3.  Re-published to Orchestrator.
4.  Updated in the environment.

This process is slow, inefficient, and introduces unnecessary risk for what should be a simple configuration change. It violates the core DevOps principle of separating **Code** from **Configuration**.

## The Solution: A Single JSON Text Asset

This pattern leverages a single **Orchestrator Text Asset** to hold your *entire* configuration as a JSON string.

A simple `Get Asset` and `Deserialize JSON` in your robot's `Init` phase loads all settings into a dynamic `JObject` variable. This gives you the power to update any bot setting in real-time, directly from Orchestrator, with **zero redeployment**.



## Key Benefits

* **Real-Time Updates:** Change a setting in Orchestrator, and the bot uses it on its very next run.
* **Zero Redeployment:** No need to stop processes or republish packages for simple config changes.
* **Centralized & Scalable:** Manage all your process settings from one central, auditable location.
* **Structured Data:** Use the power of JSON for nested and complex configurations (e.g., `Config("Timeouts")("Short")`) far superior to a flat Excel file.
* **DevOps Alignment:** Truly separates your static code (the package) from your dynamic configuration (the asset).

---

## Installation & Usage Guide

Here is the step-by-step guide to implement this pattern in your own projects.

### Step 1: Create the JSON Configuration

Create a text file to define all your settings in JSON format. This structure is flexible and can be nested.
**Example `config.json`:**
```json
{
  "ProcessName": "InvoiceProcessing",
  "MaxRetries": 3,
  "FeatureFlags": {
    "SendEmail": true,
    "UploadToSharePoint": false
  },
  "AppURLs": {
    "SAP_Login": "[https://sap.company.com/login](https://sap.company.com/login)",
    "Invoices_Portal": "[https://portal.company.com/invoices](https://portal.company.com/invoices)"
  },
  "FilePaths": {
    "InputFolder": "\\\\fileserver\\Invoices\\Input",
    "ArchiveFolder": "\\\\fileserver\\Invoices\\Archive"
  },
  "Timeouts": {
    "Short_ms": 3000,
    "Medium_ms": 10000,
    "Long_ms": 30000
  }
}
```

### Step 2: Configure Your UiPath Project
Now you can write normal text here and it should not be grey.

### Step 2: Configure Your UiPath Project
Now you can write normal text here and it should not be grey.
Step 2: Create the Orchestrator Asset

Log in to UiPath Orchestrator and navigate to the correct folder.

Go to the Assets tab.

Click + Add Asset and select "Create a new asset".

Configure the asset:

Asset name: Config_InvoiceProcessing (Use a clear, consistent naming convention).

Type: Select Text.

Value: Copy and paste your entire JSON string from Step 1 into this box.

Click Create.

Step 3: Configure Your UiPath Project

This logic should be placed in your Initialization workflow (e.g., InitAllSettings.xaml in the REFramework).

Add Package: Ensure you have the UiPath.WebAPI.Activities package installed (it includes Deserialize JSON).

Create Arguments/Variables:

out_Config (Type: JObject): This will hold your configuration. (Browse for type: Newtonsoft.Json.Linq.JObject).

str_ConfigJson (Type: String): A temporary variable to hold the text from the asset.

Build the Workflow:

Add a Get Asset activity.

AssetName: "Config_InvoiceProcessing" (or pass this in as an argument).

Value (Output): str_ConfigJson

Add a Deserialize JSON activity.

JsonString (Input): str_ConfigJson

JsonObject (Output): out_Config

Step 4: Use the Configuration

You can now pass the out_Config object to all other workflows. To access a value, use this syntax:
```
Assign Activity:
// Simple Value
str_SAP_URL = out_Config("AppURLs")("SAP_Login").ToString

// Nested Value
str_InputPath = out_Config("FilePaths")("InputFolder").ToString

// Integer/Number
int_MaxRetries = CInt(out_Config("MaxRetries").ToString)

// Boolean Feature Flag
bool_SendEmail = CBool(out_Config("FeatureFlags")("SendEmail").ToString)
As a best practice, you should assign all these values to local variables during the Init phase. This "fails fast" if a key is misspelled and makes your code much more readable.
```
Step 5: Test the "Live" Update

Run your robot. Note the values it's using in the logs (e.g., SAP_Login URL).

Go to Orchestrator, edit the Config_InvoiceProcessing asset, and change the SAP_Login URL. Click Update.

Run the robot again without redeploying.

Observe the logs. The robot will now be using the new URL.
