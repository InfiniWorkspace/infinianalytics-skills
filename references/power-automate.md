# Power Automate (Desktop and Cloud)

Both use official Infini templates: no code. The user imports a package and calls prebuilt subflows or child flows. Your job is telling the user where each call goes in their flow and which variables to set.

## Power Automate Desktop (PAD)

Template download (zip: `.robin` flow definition + setup guide PDF):
`https://api.analytics.infini.es/media/documentation/Instalaci%C3%B3n%20Infini%20Analytics%20PAD.zip`

After importing into PAD, set two variables at the top of the flow:

| Variable | Value |
|---|---|
| `InfiniToken` | Organization token |
| `AutomationId` | Automation UUID |

Call the template's subflows at these points:

| Subflow | Where |
|---|---|
| `InfiniStart` | Very beginning of the flow |
| `InfiniEvent` | Each meaningful step (set the Description input) |
| `InfiniEnd` | End of the flow, success path |
| `InfiniError` | Inside the error handler block |

## Power Automate Cloud (PAC)

Template download (Power Platform solution zip, imported from the Power Automate portal under **Solutions**):
`https://api.analytics.infini.es/media/documentation/Instalaci%C3%B3n%20Infini%20Analytics%20PAC.zip`

The solution wraps the API calls as reusable **child flows**. After importing, configure the solution's environment variables (`InfiniToken`, `AutomationId`) in the solution settings, then call the child flows from any cloud flow:

| Child flow | Where |
|---|---|
| `Infini - Start` | First action |
| `Infini - Event` | After each significant action, passing a description string |
| `Infini - End` | Last action on success |
| `Infini - Error` | Inside the error scope (configure "run after" on failure) |

A cloud flow that calls another technology (e.g. a Python script) can pass its execution ID along so both report into the same execution record.
