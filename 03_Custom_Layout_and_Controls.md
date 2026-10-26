# Custom Layouts and Controls with FormObject and FormLayoutFunction

This notebook demonstrates how to build professional web forms with custom layouts using ![1oid1cte7yoxb](img/1oid1cte7yoxb.png) and ![1e3rd0iilrlvu](img/1e3rd0iilrlvu.png), and how to assign concrete UI controls like sliders, radio buttons, popup menus, and checkboxes. 
The Slider control here doesn't work. 

|  |
| - |
| -Spacer- |
| -Spacer- |

### 1. Concepts Demonstrated

- ![1oid1cte7yoxb](img/1oid1cte7yoxb.png): A symbolic object that defines field metadata (interpreters, controls, labels, hints, initial inputs, and requirements).

- **Explicit UI Controls**: Override default text fields with rich HTML5 controls:

    - ![1qnif62a3axs6](img/1qnif62a3axs6.png): Renders as HTML5 <input type="range">. Pair with `Restricted["Integer", {min, max}]` so the compiler automatically sets min, max, and step.

    - ![04xc7g8oqj2tv](img/04xc7g8oqj2tv.png): Renders as <input type="radio">.

    - ![01up4xdjpvpdf](img/01up4xdjpvpdf.png): Renders as <select><option>...</option></select>.

    - ![1beiblkz20lpc](img/1beiblkz20lpc.png): Renders as <input type="checkbox">.

- How ![1qnif62a3axs6](img/1qnif62a3axs6.png) Works in Web Forms:

    - **Syntax**: Use "Control" -> Slider (not a function). The range and step are inferred directly from the ![13k9np9hby57j](img/13k9np9hby57j.png) interpreter.

    - **Initial Value**: Use "Input" -> initialValue (e.g. "Input" -> 10). In ![1oid1cte7yoxb](img/1oid1cte7yoxb.png), the `"Input"` key sets the HTML value attribute for the slider thumb.

    - **Interactivity Model**: Unlike desktop notebook sliders which use client-side ![0bglc9zml28v0](img/0bglc9zml28v0.png), an HTML5 web form slider conforms to HTTP request/response: moving the slider changes the control's value, which is sent to the server upon clicking Submit.

- ![1e3rd0iilrlvu](img/1e3rd0iilrlvu.png): A pure function applied to the ![1oid1cte7yoxb](img/1oid1cte7yoxb.png) that arranges controls using Wolfram layout constructs (![0xu7k615i6kp5](img/0xu7k615i6kp5.png), ![1ejx23ioivqha](img/1ejx23ioivqha.png), ![1r8fzt3meab20](img/1r8fzt3meab20.png), ![0gqxhnt4krjf1](img/0gqxhnt4krjf1.png)):

    - #["key", "Label"]: Retrieves the field's formatted label.

    - #["key", "Control"]: Retrieves the bare input control.

    - #["key", "Help"]: Retrieves help or tooltip text.

    - #["key", "Required"]: Checks if the field is required.

- **Cloud Deployment and Cleanup**: Using ![0770wxpj0kevj](img/0770wxpj0kevj.png) and ![0od20of8xqqi6](img/0od20of8xqqi6.png).

|  |
| - |
| -Spacer- |
| -Spacer- |

### 2. Define the FormObject and FormFunction

Here we construct an Enterprise Subscription Pricing Calculator with a custom two-column grid layout, styled dividers, and explicit controls. 

|  |
| - |
| -Spacer- |
| -Spacer- |

```wl
In[]:= fieldList = {
        "companyName" -> <|
            "Interpreter" -> "String", 
            "Label" -> "Company Name", 
            "Hint" -> "Acme Corporation", 
            "Required" -> True, 
            "Input" -> "Acme Corporation" 
          |>, 
        "tier" -> <|
            "Interpreter" -> {"Starter" -> 29, "Professional" -> 99, "Enterprise" -> 299}, 
            "Label" -> "Subscription Tier", 
            "Control" -> RadioButtonBar, 
            "Input" -> "Professional" 
          |>, 
     "users" -> <|
            "Interpreter" -> Restricted["Integer", {1, 100}], 
            "Label" -> "Seat Count (1 - 100)", 
            "Control" -> Slider, 
            "Input" -> 10 
          |>, 
        "billingCycle" -> <|
            "Interpreter" -> {"Monthly" -> 1.0, "Annual (15% discount)" -> 0.85}, 
            "Label" -> "Billing Cycle", 
            "Control" -> PopupMenu, 
            "Input" -> "Monthly" 
          |>, 
        "supportAddon" -> <|
            "Interpreter" -> "Boolean", 
            "Label" -> "24/7 Dedicated Support (+$50/mo)", 
            "Control" -> Checkbox, 
            "Input" -> False 
          |> 
      };
```

```wl
In[]:= processFunc = Function[data, 
        Module[{baseRate, userCost, discount, supportCost, totalMonthly, totalAnnual}, 
           baseRate = data["tier"]; 
           userCost = data["users"]*15; 
           discount = data["billingCycle"]; 
           supportCost = If[TrueQ[data["supportAddon"]], 50, 0]; 
           totalMonthly = Round[(baseRate + userCost + supportCost)*discount, 0.01]; 
           totalAnnual = Round[totalMonthly*12, 0.01]; 
           Panel[
              Column[{
                  Style["Estimate for " <> data["companyName"], Bold, 18], 
                  Spacer[10], 
                  Grid[{
                      {"Base Tier Rate:", "$" <> ToString[baseRate] <> "/mo"}, 
                      {"Seat Add-on (" <> ToString[data["users"]] <> " users):", "$" <> ToString[userCost] <> "/mo"}, 
                      {"Support Add-on:", "$" <> ToString[supportCost] <> "/mo"}, 
                      {"Effective Monthly Cost:", Style["$" <> ToString[NumberForm[totalMonthly, {6, 2}]], Bold, 16, Darker[Green]]}, 
                      {"Projected Annual Cost:", Style["$" <> ToString[NumberForm[totalAnnual, {6, 2}]], Bold, 16]} 
                    }, Alignment -> {{Left, Right}, Center}, Spacings -> {3, 1}] 
                }], 
              ImageSize -> 450 
            ] 
         ] 
      ];
```

```wl
In[]:= form2 = FormFunction[
       fieldList,   processFunc, 
      FormLayoutFunction -> Function[
           Grid[
              {
                 {Style[#["companyName", "Label"], Bold], #["companyName", "Control"]}, 
                 {Style[#["tier", "Label"], Bold], #["tier", "Control"]},
                 {Style[#["users", "Label"], Bold], #["users", "Control"]}, 
                 {Style[#["billingCycle", "Label"], Bold], #["billingCycle", "Control"]}, 
                 {Style[#["supportAddon", "Label"], Bold], #["supportAddon", "Control"]} 
               }, 
              Alignment -> {{Right, Left}, Center}, 
              Dividers -> {None, {2 -> GrayLevel[0.85], 3 -> GrayLevel[0.85], 4 -> GrayLevel[0.85], 5 -> GrayLevel[0.85]}}, 
              Spacings -> {2, 1.5} 
            ] 
         ], 
      AppearanceRules -> <|
           "Title" -> "Enterprise Pricing Calculator", 
           "Description" -> "Custom layout using FormObject and FormLayoutFunction with HTML5 controls.", 
           "SubmitLabel" -> "Calculate Estimate" 
         |> 
     ]; 
  
 
```

### 3. Deploy to the Wolfram Cloud

Deploy the form to the cloud: 

```wl
In[]:= co = CloudDeploy[form2, Permissions -> "Public"]
```

![049qygjl93v5h](img/049qygjl93v5h.png)

Retrieve the URL: 

```wl
In[]:= First[co]
```

Open the link in your browser to verify: 

- Clean 2-column grid layout with horizontal dividers.

- Radio buttons for tier selection.

- Range slider for user seats starting at 10.

- Popup dropdown menu for billing cycle.

- Checkbox for the support add-on.

|  |
| - |
| -Spacer- |
| -Spacer- |

### 4. Delete the Cloud Object

Evaluate the following to delete the cloud object: 

```wl
In[]:= DeleteObject[co]
```

![15ea3t986qwe6](img/15ea3t986qwe6.png)