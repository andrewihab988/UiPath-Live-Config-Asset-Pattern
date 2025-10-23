# UiPath Live Config Asset Pattern

A UiPath template and guide for moving project configurations from static `Config.xlsx` files to a single, dynamic JSON Text Asset in Orchestrator.  
This pattern allows you to update your bot's settings in real time **without redeploying** the package.

---

## The Problem: Static Configs are Inefficient

The traditional `Config.xlsx` file is a major bottleneck in a mature RPA environment.  
Every time a simple environment-specific setting (URL, path, retry count, feature flag) changes, the entire package must be:
1. Opened in Studio  
2. Edited  
3. Re-published to Orchestrator  
4. Updated in the environment  

This workflow violates the DevOps principle of separating *Code* from *Configuration*.

---

## The Solution: A Single JSON Text Asset

Use a single **Orchestrator Text Asset** to store your entire configuration as a JSON string.  
A simple `Get Asset` and `Deserialize JSON` activity loads the settings into a `JObject` variable during initialization.

### Benefits

- Real-time updates: configuration changes apply immediately on the next run  
- Zero redeployment needed  
- Centralized, auditable configuration  
- Structured JSON for nested, complex settings  
- DevOps-aligned design separating code and config

---

## Installation & Usage Guide 🚀

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


---

### Step 2: Create the Orchestrator Asset

1. Log in to **UiPath Orchestrator**.  
2. Navigate to the correct folder for your process.  
3. Go to the **Assets** tab.  
4. Click **+ Add Asset** → **Create a new asset**.  
5. Configure the asset:
   - **Name:** `Config_InvoiceProcessing`
   - **Type:** `Text`
   - **Value:** Paste the JSON from Step 1  
6. Click **Create**.

---

## Step 3: Configure Your UiPath Project

Place this logic in your **Initialization** sequence  
(e.g., `InitAllSettings.xaml` in REFramework).

### Add Required Package
Install **UiPath.WebAPI.Activities** (includes `Deserialize JSON`).

### Create Arguments and Variables

| Name             | Direction | Type                              | Description                         |
|------------------|------------|------------------------------------|-------------------------------------|
| `out_Config`     | Out        | `Newtonsoft.Json.Linq.JObject`     | Serialized configuration object     |
| `str_ConfigJson` | In/Out     | `String`                           | Temporary variable to store JSON    |


### Build the Workflow

1. **Add a Get Asset activity**
   - **AssetName:** `Config_InvoiceProcessing`
   - **Value (Output):** `str_ConfigJson`

2. **Add a Deserialize JSON activity**
   - **JsonString (Input):** `str_ConfigJson`
   - **JsonObject (Output):** `out_Config`

---

## Step 4: Use the Configuration

Access configuration values dynamically across workflows.

### Example Usage in Assign Activity

```
Assign Activity:
’ Simple Value str_SAP_URL = out_Config(“AppURLs”)(“SAP_Login”).ToString
’ Nested Value str_InputPath = out_Config(“FilePaths”)(“InputFolder”).ToString
’ Integer/Number int_MaxRetries = CInt(out_Config(“MaxRetries”).ToString)
’ Boolean Feature Flag bool_SendEmail = CBool(out_Config(“FeatureFlags”)(“SendEmail”).ToString)

```

**Best Practice:**  
Assign all configuration values to *local variables* during the **Init** phase to ensure fail-fast validation and improve readability.

---

## Step 5: Test the Live Update

1. Run your robot and log config values (e.g., `SAP_Login` URL).  
2. Edit the `Config_InvoiceProcessing` asset in Orchestrator.  
3. Modify any setting (e.g., `SAP_Login`).  
4. Click **Update**, then run the bot again — no redeployment required.

