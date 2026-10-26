# Custom Layouts and Controls with FormObject and FormLayoutFunction

This notebook demonstrates how to build professional web forms with custom layouts using FormObject and FormLayoutFunction, and how to assign concrete UI controls like sliders, radio buttons, popup menus, and checkboxes. 
The Slider control here doesn’t work.

---

## 1. Concepts Demonstrated

* FormObject: A symbolic object that defines field metadata (interpreters, controls, labels, hints, initial inputs, and requirements).

* **Explicit UI Controls**: Override default text fields with rich HTML5 controls:

	* Slider: Renders as HTML5 ``<input type="range">``. Pair with ``Restricted["Integer", {min, max}]`` so the compiler automatically sets ``min``, ``max``, and ``step``.

	* RadioButtonBar: Renders as ``<input type="radio">``.

	* PopupMenu: Renders as ``<select><option>...</option></select>``.

	* Checkbox: Renders as ``<input type="checkbox">``.

* How Slider Works in Web Forms:

	* **Syntax**: Use ``"Control" -> Slider`` (not a function). The range and step are inferred directly from the Restricted interpreter.

	* **Initial Value**: Use ``"Input" -> initialValue`` (e.g. ``"Input" -> 10``). In FormObject, the ``"Input"`` key sets the HTML ``value`` attribute for the slider thumb.

	* **Interactivity Model**: Unlike desktop notebook sliders which use client-side Dynamic, an HTML5 web form slider conforms to HTTP request/response: moving the slider changes the control's value, which is sent to the server upon clicking Submit.

* FormLayoutFunction: A pure function applied to the FormObject that arranges controls using Wolfram layout constructs (Grid, Column, Row, Panel):

	* ``\#["key", "Label"]``: Retrieves the field's formatted label.

	* ``\#["key", "Control"]``: Retrieves the bare input control.

	* ``\#["key", "Help"]``: Retrieves help or tooltip text.

	* ``\#["key", "Required"]``: Checks if the field is required.

* **Cloud Deployment and Cleanup**: Using CloudDeploy and DeleteObject.

---

## 2. Define the FormObject and FormFunction

Here we construct an Enterprise Subscription Pricing Calculator with a custom two-column grid layout, styled dividers, and explicit controls.

---

```wl
In[56]:=
fieldList = {
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

In[57]:=
processFunc = Function[data, 
	    Module[{baseRate, userCost, discount, supportCost, totalMonthly, totalAnnual}, 
	      baseRate = data["tier"];
	      userCost = data["users"] * 15;
	      discount = data["billingCycle"];
	      supportCost = If[TrueQ[data["supportAddon"]], 50, 0];
	      totalMonthly = Round[(baseRate + userCost + supportCost) * discount, 0.01];
	      totalAnnual = Round[totalMonthly * 12, 0.01];
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

In[59]:=
form2 = FormFunction[
	  fieldList,    processFunc, 
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

## 3. Deploy to the Wolfram Cloud

Deploy the form to the cloud:

```wl
In[60]:= co = CloudDeploy[form2, Permissions -> "Public"]

Out[60]= CloudObject["https://www.wolframcloud.com/obj/c501b438-2087-4360-a05a-8044661158d4"]
```

Retrieve the URL:

```wl
First[co]
```

Open the link in your browser to verify:

* Clean 2-column grid layout with horizontal dividers.

* Radio buttons for tier selection.

* Range slider for user seats starting at 10.

* Popup dropdown menu for billing cycle.

* Checkbox for the support add-on.

---

## 4. Delete the Cloud Object

Evaluate the following to delete the cloud object:

```wl
In[58]:= DeleteObject[co]

In[27]:= DeleteObject[CloudObject["https://www.wolframcloud.com/obj/4f8c3bb7-1550-4fc1-8ee4-2dd85a8a8b76"]]
```
