# Multi-Page Dynamic Form Wizards

This notebook demonstrates how to construct multi-page dynamic form workflows (wizards) where subsequent pages are generated programmatically based on user inputs from earlier pages.

---

## 1. Concepts Demonstrated

* **Multi-Page Form Specification**: Providing a list of page specifications to FormFunction :

``FormFunction[{page1, page2, ...}, actionFunction]``

* **Dynamic Page Generation via Pure Functions** :

	* The second and subsequent pages can be pure functions of all inputs gathered so far: ``Function[prevInputs, ...]``.

	* For example, if page 1 asks for a count N, page 2 can generate exactly N custom input fields using Table.

* **Automatic Wizard Navigation** :

	* The Wolfram Cloud form engine automatically provides **"Next"** and **"Back"** navigation between intermediate pages, and a **"Submit"** button on the final page.

	* State is preserved across wizard steps.

* **Aggregated Submission**: The final action function receives an Association containing values from all pages combined.

* **Cloud Deployment and Cleanup**: Using CloudDeploy and DeleteObject.

---

## 2. Define the Multi-Page Wizard

In this example:

* **Page 1**: Requests the project name and the number of team members (1 to 4).

* **Page 2**: A pure function that inspects ``\#teamSize`` and generates a dedicated text input for each member.

* **Action Function**: Compiles all responses into a formatted project manifest and team roster table.

```wl
In[77]:=
form = FormFunction[
	  {
	    (* Page 1: Project Metadata *)
	    {
	      {"projectName", "Project Name:"} -> "String", 
	      {"teamSize", "Number of Team Members (1-4):"} -> Restricted["Integer", {1, 4}] -> 2
	    }, 
	    
	    (* Page 2: Dynamically Generated Fields Based on Page 1 *)
	    Function[prev, 
	      Table[
	        {"member_" <> ToString[i], "Member " <> ToString[i] <> " (Name and Role):"} -> "String", 
	        {i, prev["teamSize"]}
	      ]
	    ]
	  }, 
	  
	  (* Final Action Function: Processes Data From All Steps *)
	  Function[allData, 
	    Module[{members}, 
	      members = Table[allData["member_" <> ToString[i]], {i, allData["teamSize"]}];
	      Panel[
	        Column[{
	          Style["Project Roster: " <> allData["projectName"], Bold, 16], 
	          Spacer[5], 
	          Grid[
	            Prepend[
	              MapIndexed[{First[#2], #1} &, members], 
	              {"#", "Team Member"}
	            ], 
	            Dividers -> All, 
	            FrameStyle -> GrayLevel[0.8], 
	            ItemStyle -> {Automatic, {1 -> Bold}}, 
	            Spacings -> {2, 1}
	          ]
	        }], 
	        ImageSize -> 420
	      ]
	    ]
	  ], 
	  AppearanceRules -> <|
	    "Title" -> "Dynamic Multi-Page Project Setup Wizard", 
	    "Description" -> "Page 2 dynamically generates member fields matching the count chosen on Page 1.", 
	    "SubmitLabel" -> "Generate Project Manifest"
	  |>, 
	  PageTheme -> "White"
	];
```

---

## 3. Deploy to the Wolfram Cloud

Deploy the multi-page wizard to the Wolfram Cloud:

```wl
In[80]:= co = CloudDeploy[form, Permissions -> "Public"]

Out[80]= CloudObject["https://www.wolframcloud.com/obj/61a6eb1f-5d85-4068-89f7-6190f562ceea"]
```

Retrieve the URL:

```wl
co["URL"]
```

### Try It in the Browser:

1. Open the URL. Page 1 asks for the **Project Name** and **Number of Team Members** (default: 2).

2. Enter a project name, change the count to **3**, and click **Next**.

3. Page 2 dynamically presents exactly **3 input fields** for Member 1, Member 2, and Member 3.

4. Fill in the member details and submit to see the compiled project roster.

---

## 4. Delete the Cloud Object

When testing is complete, delete the cloud deployment:

```wl
In[78]:= DeleteObject[co]
```
