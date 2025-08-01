# Azure-VM-Status-Automation-
A complete solution for automated daily VM availability reporting using Azure Logic Apps and Automation Runbooks. This system generates daily reports showing which VMs are running, stopped, or deallocated across your Azure subscriptions


## 🏗️ Architecture Overview

This solution uses:
- **Azure Automation Runbook** - PowerShell script to collect VM status data
- **Azure Logic App** - Orchestrates the workflow and sends email reports
- **Recurrence Trigger** - Runs daily at scheduled time
- **HTML Email Reports** - Formatted table showing VM status

## 📋 Prerequisites

- Azure subscription with appropriate permissions
- VMs deployed in your subscription
- Email account for receiving reports (Office 365 or Gmail)
- Resource group for the automation resources

## 🚀 Quick Start

### Step 1: Create Azure Automation Account

1. In Azure Portal, create a new **Automation Account**
2. Enable **System Managed Identity**
3. Assign **Reader** role to the Managed Identity at subscription level from IAM permissions in the subscriptions tab

### Step 2: Create the Runbook

1. In your Automation Account, go to **Runbooks**
2. Create a new **PowerShell** runbook
3. Copy the script from [`scripts/Get-VMStatus.ps1`](scripts/Get-VMStatus.ps1)
4. Update the `$subscriptionId` variable with your subscription ID
5. **Save** and **Publish** the runbook
6. You may test the runbook. The output should look like [`images/runbook-result.png`](runbook-result.png)

### Step 3: Deploy the Logic App

1. Create the logic app following the steps in [`images/logic-app-flow.png`](logic-app-flow.png)
2. Configure email connector with your credentials

### Step 4: Test the Solution

1. Manually run the Logic App to test
2. Verify email delivery and HTML formatting
3. Check that all VMs are being reported correctly

## 📁 Repository Structure

```
├── scripts/
│   └── Get-VMStatus.ps1          # PowerShell runbook script
├── images/
│   └── logic-app-flow.png        # Logic App workflow diagram
└── README.md                     # This file
```

## 🔧 PowerShell Script Details

The automation runbook performs these actions:

1. **Connects** using System Managed Identity (no credentials needed)
2. **Retrieves** all VMs and their power states using `Get-AzVM -Status`
3. **Formats** data into a structured object with:
   - VM Name
   - Resource Group
   - Location
   - Power State
   - Timestamp
4. **Outputs** clean JSON (using `Out-Null` to suppress connection messages)

### Key Features:
- ✅ Uses Managed Identity for secure authentication
- ✅ Suppresses unwanted output for clean JSON response
- ✅ Includes timestamp for each report
- ✅ Captures all essential VM metadata

```powershell
# Suppress default output from Connect-AzAccount
Connect-AzAccount -Identity | Out-Null

# Set subscription
$subscriptionId = "<YOUR_SUBSCRIPTION_ID>"
Set-AzContext -SubscriptionId $subscriptionId | Out-Null

# Get all VMs and their power states
$vms = Get-AzVM -Status

# Build report
$report = @()
foreach ($vm in $vms) {
    $report += [PSCustomObject]@{
        VMName        = $vm.Name
        ResourceGroup = $vm.ResourceGroupName
        Location      = $vm.Location
        Status        = $vm.PowerState
        TimeStamp     = (Get-Date).ToString("yyyy-MM-dd HH:mm:ss")
    }
}

# Only output JSON
$report | ConvertTo-Json -Depth 5
```

## 📧 Logic App Workflow

The Logic App orchestrates the entire process:

1. **Recurrence** - Triggers daily (configurable time)
2. **Create Job** - Starts the PowerShell runbook (50s timeout)
3. **Get Job Output** - Retrieves the JSON results (0.4s)
4. **Parse JSON** - Converts string output to objects (0s)
5. **Create HTML Table** - Formats data for email (0s)
6. **Send Email** - Delivers the report (0.8s)

### Execution Times:
- Total runtime: ~51 seconds
- Most time spent waiting for runbook execution
- Email delivery is fast and reliable

## 🎨 Sample Output

The email report includes an HTML table with:

| VM Name | Resource Group | Location | Status | Timestamp |
|---------|---------------|----------|---------|-----------|
| web-vm-01 | rg-production | East US | VM running | 2025-01-15 09:00:00 |
| db-vm-02 | rg-production | East US | VM deallocated | 2025-01-15 09:00:00 |
| test-vm-03 | rg-development | West US | VM stopped | 2025-01-15 09:00:00 |

## ⚙️ Configuration

### Subscription Settings
Update the subscription ID in the PowerShell script:
```powershell
$subscriptionId = "your-subscription-id-here"
```

### Schedule Settings
Modify the recurrence trigger in Logic App:
- **Frequency**: Daily
- **Time**: 9:00 AM (or preferred time)
- **Time Zone**: Your local timezone

## 🔍 Monitoring & Troubleshooting

### Common Issues:

1. **Authentication Errors**
   - Ensure Managed Identity has Reader permissions
   - Use OAuth as preferred authentication method when creating a job in Logic App
   - Check subscription ID is correct

2. **Empty Results**
   - Verify VMs exist in the subscription
   - Check PowerShell execution logs

3. **Email Delivery Issues**
   - Validate email connector configuration
   - Check spam/junk folders

See [`docs/troubleshooting.md`](docs/troubleshooting.md) for detailed solutions.

## 🛠️ Customization Options

- **Filter VMs**: Add resource group or tag filters
- **Additional Data**: Include VM size, OS type, or custom tags
- **Multiple Subscriptions**: Loop through multiple subscriptions
- **Different Formats**: Export to Excel, CSV, or other formats
- **Alerting**: Add conditions to alert on specific states

## 💰 Cost Considerations

- **Logic App**: ~$0.01 per execution (daily = ~$0.30/month)
- **Automation**: First 500 minutes free, then ~$0.002/minute
- **Storage**: Minimal for logs and job history

Total estimated cost: **<$1 per month** for daily execution.

## 🔐 Security Best Practices

- **Use Managed Identity**: No stored credentials required
- **Least Privilege**: Reader role only for VM status
- **Secure Email**: Use organizational email for reports
- **Resource Groups**: Separate automation resources

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -am 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Create a Pull Request

## 📚 Additional Resources

### Documentation
- [Azure Logic Apps Documentation](https://docs.microsoft.com/en-us/azure/logic-apps/)
- [Azure Automation Documentation](https://docs.microsoft.com/en-us/azure/automation/)
- [Azure PowerShell VM Management](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/ps-common-ref)


## 🏷️ Tags

`azure` `logic-apps` `automation` `powershell` `vm-monitoring` `devops` `cloud` `reporting`

---

**Author**: Harsh Jain
**Last Updated**: August 2025  
**Version**: 1.0.0  
**⭐ Star this repo if it helped you!**
