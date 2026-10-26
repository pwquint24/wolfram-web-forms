# Instant Web Forms with Semantic Interpreters

This notebook demonstrates how to create and deploy an instant web form to the Wolfram Cloud using ![1h095lyghj2iw](img/1h095lyghj2iw.png) and ![0770wxpj0kevj](img/0770wxpj0kevj.png). 

|  |
| - |
| -Spacer- |
| -Spacer- |

### 1. Concepts Demonstrated

- `FormFunction[formspec, func, fmt]`: Creates a submittable web form that evaluates an action function when submitted.

- The ![02yl5wbpv7apd](img/02yl5wbpv7apd.png) Framework: Wolfram Web Forms parse and validate user inputs into rich symbolic Wolfram expressions automatically:

    - `"Country"`: Recognizes country names via the Wolfram Knowledgebase and returns `Entity["Country", ...]`.

    - `"Color"`: Provides an HTML5 color picker and returns a native ![0qa7bk0yt9sp5](img/0qa7bk0yt9sp5.png).

    - `Restricted["Integer", {min, max}]`: Validates that input is an integer within the specified range.

- **Defaults and Labels**: Using {"key", "Display Label"} -> type -> defaultValue syntax.

- ![0770wxpj0kevj](img/0770wxpj0kevj.png): Deploys the form as an active web application in the Wolfram Cloud.

- ![0od20of8xqqi6](img/0od20of8xqqi6.png): Removes the deployed cloud object when it is no longer needed.

|  |
| - |
| -Spacer- |
| -Spacer- |

### 2. Define the FormFunction

The form below asks for a country, a highlight color, and a zoom radius in miles. When submitted, it applies the action function to generate a centered ![1peon2ywgb4iy](img/1peon2ywgb4iy.png) image. Here we are using the form {name, label}->type. 
You can either define your form as a separate formObject, and then insert it into the FormFunction, or simply include it in the form function. 

```wl
In[]:= formObject = FormObject[{
         {"country", "Select a Country:"} -> "Country" -> "Japan", 
         {"color", "Highlight Color:"} -> "Color" -> RGBColor[0.85, 0.2, 0.2], 
         {"zoom", "Zoom Radius (miles):"} -> Restricted["Integer", {100, 2000}] -> 500 
       }, AppearanceRules -> <|
          "Title" -> "Country Geo-Focus Form", 
          "Description" -> "Demonstrating semantic Interpreter types (Country, Color, Restricted).", 
          "SubmitLabel" -> "Generate Map" 
        |>];
```

```wl
In[]:= form = FormFunction[
      formObject, 
      Function[params, 
         GeoGraphics[
            {params["color"], GeoDisk[params["country"], Quantity[params["zoom"], "Miles"]]}, 
            GeoRange -> Quantity[params["zoom"]*1.5, "Miles"], 
            GeoCenter -> params["country"], 
            ImageSize -> 500 
          ] 
       ], 
      "PNG" 
      
    ];
```

|  |
| - |
| -Spacer- |
| -Spacer- |

### 3. Deploy to the Wolfram Cloud

Deploy the form to your Wolfram Cloud account. The returned ![1hu2bedxpqmop](img/1hu2bedxpqmop.png) contains the live public URL. 

```wl
In[]:= co = CloudDeploy[form, Permissions -> "Public"]
```

![1knnsu6bo8slg](img/1knnsu6bo8slg.png)

You can view the URL directly: 

```wl
In[]:= co["URL"]
```

Click the resulting link or copy the URL into your web browser to test the form. Notice:

1. The country field supports natural language and knowledgebase entities.

2. The color field opens a browser-native color picker.

3. The zoom field enforces bounds between 100 and 2000. 

|  |
| - |
| -Spacer- |
| -Spacer- |

### 4. Delete the Cloud Object

Evaluate the following cell to un-deploy and delete the cloud object from your account: 

```wl
In[]:= DeleteObject[co]
```