# Advanced Formatting and Layout in Web Forms

This notebook extends the basic FormLayoutFunction examples to produce richer, sectioned, and more visually structured web forms. The layout machinery is the same Wolfram language primitives used in notebooks --- Column, Row, Grid, Panel, and Delimiter --- so complex, multi-panel layouts can be built entirely symbolically. 

|  |
| - |
| -Spacer- |
| -Spacer- |

### 1. Concepts Demonstrated

- **Sectioned layouts**: A FormLayoutFunction can group fields into logical sections separated by Delimiter, each headed by a custom header built from Style and small Graphics.

- **Grid formatting options**: Alignment, Spacings, Dividers, Background, Frame, and ItemSize give fine-grained control over multi-column field layouts.

- Non-textual AppearanceRules: `"Title"` and `"Description"` accept styled or graphical expressions, not just plain strings.

- `"ItemLayout"`: The AppearanceRules key `"ItemLayout"` controls how each field's label is positioned relative to its control: `"Vertical"`, `"Horizontal"`, or `"Inline"`.

- **Theming**: PageTheme sets the overall look, and style keys such as `"FormTitleStyle"` and `"FormDescriptionStyle"` can further customize headings.

|  |
| - |
| -Spacer- |
| -Spacer- |

### 2. Define a Reusable Section Header

A small helper builds a colored section header out of a thin orange bar and bold text: 

```wl
In[]:= sectionHeader[text_] := Row[{
       Graphics[{Orange, Rectangle[{0, 0}, {1, 1}]}, ImageSize -> {5, 16}], 
       Spacer[5], 
       Style[text, Bold, 14, Darker[Blue]] 
     }];
```

|  |
| - |
| -Spacer- |
| -Spacer- |

### 3. Define the Multi-Section Estimator

This form organizes six fields into three sections --- **Project Details**, **Budget**, and **Timeline** --- each rendered by a dedicated Grid with right-aligned labels, separated by Delimiter. 

```wl
In[]:= estimator = FormFunction[
      {
         "projectName" -> <|
             "Interpreter" -> "String", 
             "Label" -> "Project Name", 
             "Hint" -> "e.g. Q3 Launch", 
             "Required" -> True, 
             "Input" -> "Q3 Launch" 
           |>, 
         "category" -> <|
             "Interpreter" -> {"Web App", "Data Analysis", "Automation"},
             "Label" -> "Category", 
             "Control" -> PopupMenu, 
             "Input" -> "Web App" 
           |>, 
         "budget" -> <|
             "Interpreter" -> Restricted["Integer", {1000, 100000}], 
             "Label" -> "Budget", 
             "Input" -> 25000 
           |>, 
         "currency" -> <|
             "Interpreter" -> {"USD ($)" -> "$", "EUR (Euro)" -> "€", "GBP (Pound)" -> "£"}, 
             "Label" -> "Currency", 
             "Control" -> PopupMenu, 
             "Input" -> "USD ($)" 
           |>, 
         "startDate" -> <|
             "Interpreter" -> "Date", 
             "Label" -> "Start Date", 
             "Input" -> DateString[ Today] 
           |>, 
         "durationWeeks" -> <|
             "Interpreter" -> Restricted["Integer", {1, 52}], 
             "Label" -> "Duration (weeks)", 
             "Input" -> 8 
           |> 
       }, 
      Function[params, 
         Module[{weeklyCost}, 
            weeklyCost = Round[params["budget"]/params["durationWeeks"], 0.01]; 
            Panel[
               Column[{
                   Style["Cost Estimate: " <> params["projectName"], Bold, 18, Darker[Blue]], 
                   Spacer[5], 
                   Grid[{
                       {"Category:", params["category"]}, 
                       {"Total Budget:", params["currency"] <> ToString[NumberForm[params["budget"], {10, 2}]]}, 
                       {"Duration:", ToString[params["durationWeeks"]] <> " weeks"}, 
                       {"Weekly Run Rate:", Style[params["currency"] <> ToString[NumberForm[weeklyCost, {10, 2}]], Bold, Darker[Green]]}, 
                       {"Start Date:", DateString[params["startDate"], {"Month", "/", "Day", "/", "Year"}]} 
                     }, Alignment -> {{Right, Left}, Center}, Spacings -> {2, 1}] 
                 }], 
               ImageSize -> 400 
             ] 
          ] 
       ], 
      AppearanceRules -> <|
          "Title" -> "Project Cost Estimator", 
          "Description" -> Style["A multi-section form with advanced formatting and custom layout.", Italic, Gray], 
          "SubmitLabel" -> "Calculate Estimate" 
        |>, 
      FormLayoutFunction -> Function[
          Column[{
              sectionHeader["PROJECT DETAILS"], 
              Grid[{
                  {#["projectName", "Label"], #["projectName", "Control"]}, 
                  {#["category", "Label"], #["category", "Control"]} 
                }, Alignment -> {{Right, Left}, Center}, Spacings -> {2, 1}], 
              Delimiter, 
              sectionHeader["BUDGET"], 
              Grid[{
                  {#["budget", "Label"], #["budget", "Control"]}, 
                  {#["currency", "Label"], #["currency", "Control"]} 
                }, Alignment -> {{Right, Left}, Center}, Spacings -> {2, 1}], 
              Delimiter, 
              sectionHeader["TIMELINE"], 
              Grid[{
                  {#["startDate", "Label"], #["startDate", "Control"]},
                  {#["durationWeeks", "Label"], #["durationWeeks", "Control"]} 
                }, Alignment -> {{Right, Left}, Center}, Spacings -> {2, 1}] 
            }] 
        ], 
      PageTheme -> "White" 
    ];
```

#### Notes on the Layout

- Each section is a Grid with `Alignment -> {{Right, Left}, Center}` so labels sit flush right and controls flush left.

- Delimiter inserts a horizontal rule between sections.

- #["key", "Label"] and #["key", "Control"] extract the label and bare input element for each field.

- The `"Currency"` interpreter maps human-readable popup labels (`"USD ($)"`) to symbols (`"$"`) that are used in the result calculation.

|  |
| - |
| -Spacer- |
| -Spacer- |

### 4. Deploy to the Wolfram Cloud

Deploy the estimator: 

```wl
In[]:= co = CloudDeploy[estimator, Permissions -> "Public"]
```

[//]: # ({String, Cell, List})

Retrieve the URL: 

```wl
In[]:= co["URL"]
```

#### Try It in the Browser

1. Notice the three labeled sections and the horizontal rules between them.

2. Change the budget, currency, and duration, then submit to see the formatted cost breakdown. 

|  |
| - |
| -Spacer- |
| -Spacer- |

### 5. Delete the Cloud Object

```wl
In[]:= DeleteObject[co]
```