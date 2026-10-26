# Cascading Dependent Fields with TemplateSlot and Autosubmit

This notebook demonstrates how to create dynamic web forms with interdependent fields using the Wolfram Templating Engine (![1ikpkshtlojge](img/1ikpkshtlojge.png) and ![1obymmrbftm83](img/1obymmrbftm83.png)). 

|  |
| - |
| -Spacer- |
| -Spacer- |

### 1. Concepts Demonstrated

#### Why `Dynamic[ ...]` Does Not Work in Web Forms

In desktop Mathematica notebooks, ![0bglc9zml28v0](img/0bglc9zml28v0.png) creates a live client-side reactive binding within the front-end preemptive evaluation loop.

However, **web forms adhere to HTTP request/response semantics**. When you place `Dynamic[x]` inside a web form: 

- It is **not** converted into client-side reactive JavaScript.

- It is treated as a held expression that renders literally into the HTML (e.g. value="Dynamic[0]").

#### How Web Forms Achieve Interdependent Fields: The Template Slot Engine

Wolfram Web Forms achieve field-to-field dynamic interdependence through server-side cascading: 

- `TemplateSlot["fieldName"]`: References the current value of an earlier field.

- `TemplateExpression[ ...]`: Dynamically computes choices, ranges, or controls based on earlier field values.

#### Under the Hood: The Autosubmit Mechanism

1. **Initial Page Load**: When the Wolfram Engine sees that field `"topic"` depends on `TemplateSlot["category"]`, it **withholds** `"topic"` from the initial HTML.

2. **Autosubmit Tagging**: The prerequisite `"category"` input is tagged with data-autosubmit="autosubmit".

3. **Triggered Submission**: As soon as the user selects a category, the browser automatically submits an HTTP request to the Wolfram Cloud.

4. **Server Re-evaluation**: The engine evaluates the ![1obymmrbftm83](img/1obymmrbftm83.png) with the chosen category and returns the updated form, now unlocking the populated `"topic"` field. 

|  |
| - |
| -Spacer- |
| -Spacer- |

### 2. Define the Form with Cascading Fields

In this example, choosing a category ("Science", "Arts", or "Sports") automatically triggers a server-side round trip to populate the corresponding list of topics. 

```wl
In[]:= form = FormFunction[
      {
         "category" -> {"Science", "Arts", "Sports"}, 
         "topic" -> TemplateExpression[
             Switch[TemplateSlot["category"], 
                "Science", {"Physics", "Chemistry", "Biology", "Astronomy"}, 
                "Arts", {"Literature", "Music", "Painting", "Cinema"}, 
                "Sports", {"Football", "Basketball", "Tennis", "Swimming"}, 
                _, {} 
              ] 
           ], 
         "notes" -> "String" 
       }, 
      Function[params, 
         Panel[
            Column[{
                Style["Selection Summary", Bold, 16], 
                Spacer[5], 
                Grid[{
                    {"Selected Category:", Style[params["category"], Bold]}, 
                    {"Selected Topic:", Style[params["topic"], Bold, Darker[Blue]]}, 
                    {"User Notes:", params["notes"]} 
                  }, Alignment -> Left, Spacings -> {2, 1}] 
              }], 
            ImageSize -> 400 
          ] 
       ], 
      AppearanceRules -> <|
          "Title" -> "Dynamic Dependent Category Selector", 
          "Description" -> "Selecting a Category automatically submits and unlocks the corresponding Topic dropdown via TemplateSlot.", 
          "SubmitLabel" -> "Confirm Selection" 
        |>, 
      PageTheme -> "Light" 
    ];
```

```wl
In[]:= form = FormFunction[
      {
         "category" -> {"Science", "Arts", "Sports"}, 
         "topic" -> TemplateExpression[
             Switch[TemplateSlot["category"], 
                "Science", {"Physics", "Chemistry", "Biology", "Astronomy"}, 
                "Arts", {"Literature", "Music", "Painting", "Cinema"}, 
                "Sports", {"Football", "Basketball", "Tennis", "Swimming"}, 
                _, {} 
              ] 
           ], 
         "notes" -> "String" 
       }, 
      Function[params, 
         Panel[
            Column[{
                Style["Selection Summary", Bold, 16], 
                Spacer[5], 
                Grid[{
                    {"Selected Category:", Style[params["category"], Bold]}, 
                    {"Selected Topic:", Style[params["topic"], Bold, Darker[Blue]]}, 
                    {"User Notes:", params["notes"]} 
                  }, Alignment -> Left, Spacings -> {2, 1}] 
              }], 
            ImageSize -> 400 
          ] 
       ], 
      AppearanceRules -> <|
          "Title" -> "Dynamic Dependent Category Selector", 
          "Description" -> "Selecting a Category automatically submits and unlocks the corresponding Topic dropdown via TemplateSlot.", 
          "SubmitLabel" -> "Confirm Selection" 
        |>, 
     PageTheme -> "Blue" 
    ];
```

|  |
| - |
| -Spacer- |
| -Spacer- |

### 3. Deploy to the Wolfram Cloud

Deploy the form: 

```wl
In[]:= co = CloudDeploy[form, Permissions -> "Public"]
```

![1dd3m66mxbsuk](img/1dd3m66mxbsuk.png)

Retrieve the URL: 

```wl
In[]:= co["URL"]
```

#### Try It in the Browser:

1. Open the URL. Note that initially, only the **Category** dropdown and **Notes** field are displayed.

2. Select **"Science"**.

3. Notice the form briefly reloads automatically: the **Topic** dropdown appears populated with *Physics*, *Chemistry*, *Biology*, and *Astronomy*.

4. Change the category to **"Arts"**---the topic dropdown immediately updates to *Literature*, *Music*, *Painting*, and *Cinema*. 

|  |
| - |
| -Spacer- |
| -Spacer- |

### 4. Delete the Cloud Object

When you are done testing, delete the cloud deployment: 

```wl
In[]:= DeleteObject[co]
```