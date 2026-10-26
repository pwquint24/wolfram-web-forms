# Multi-Page Dynamic Form Wizards

This notebook demonstrates how to construct multi-page dynamic form workflows (wizards) where subsequent pages are generated programmatically based on user inputs from earlier pages. 

|  |
| - |
| -Spacer- |
| -Spacer- |

### 1. Concepts Demonstrated

- **Multi-Page Form Specification**: Providing a list of page specifications to ![1h095lyghj2iw](img/1h095lyghj2iw.png):

  `FormFunction[{page1, page2, ...}, actionFunction]`

- **Dynamic Page Generation via Pure Functions**:

    - The second and subsequent pages can be pure functions of all inputs gathered so far: `Function[prevInputs, ...]`.

    - For example, if page 1 asks for a count $![0zr45l23q95au](img/0zr45l23q95au.png)$, page 2 can generate exactly $![0zr45l23q95au](img/0zr45l23q95au.png)$ custom input fields using ![1lsgc0fze0dfv](img/1lsgc0fze0dfv.png).

- **Automatic Wizard Navigation**:

    - The Wolfram Cloud form engine automatically provides **"Next"** and **"Back"** navigation between intermediate pages, and a **"Submit"** button on the final page.

    - State is preserved across wizard steps.

- **Aggregated Submission**: The final action function receives an ![0om0dxcw9a4rv](img/0om0dxcw9a4rv.png) containing values from all pages combined.

- **Cloud Deployment and Cleanup**: Using ![0770wxpj0kevj](img/0770wxpj0kevj.png) and ![0od20of8xqqi6](img/0od20of8xqqi6.png).

|  |
| - |
| -Spacer- |
| -Spacer- |

### 2. Define the Multi-Page Wizard

In this example: 

- **Page 1**: Requests the project name and the number of team members (1 to 4).

- **Page 2**: A pure function that inspects #teamSize and generates a dedicated text input for each member.

- **Action Function**: Compiles all responses into a formatted project manifest and team roster table.

```wl
In[]:= form = FormFunction[
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

|  |
| - |
| -Spacer- |
| -Spacer- |

### 3. Deploy to the Wolfram Cloud

Deploy the multi-page wizard to the Wolfram Cloud: 

```wl
In[]:= co = CloudDeploy[form, Permissions -> "Public"]
```

![0z8o6f5181pfs](img/0z8o6f5181pfs.png)

Retrieve the URL: 

```wl
In[]:= co["URL"]
```

#### Try It in the Browser:

1. Open the URL. Page 1 asks for the **Project Name** and **Number of Team Members** (default: 2).

2. Enter a project name, change the count to **3**, and click **Next**.

3. Page 2 dynamically presents exactly **3 input fields** for Member 1, Member 2, and Member 3.

4. Fill in the member details and submit to see the compiled project roster. 

|  |
| - |
| -Spacer- |
| -Spacer- |

### 4. Delete the Cloud Object

When testing is complete, delete the cloud deployment: 

```wl
In[]:= DeleteObject[co]
```