# Instant Web Forms with Semantic Interpreters

This notebook demonstrates how to create and deploy an instant web form to the Wolfram Cloud using FormFunction and CloudDeploy.

---

## 1. Concepts Demonstrated

* **``FormFunction[formspec, func, fmt]``**: Creates a submittable web form that evaluates an action function when submitted.

* The Interpreter Framework: Wolfram Web Forms parse and validate user inputs into rich symbolic Wolfram expressions automatically:

	* ``"Country"``: Recognizes country names via the Wolfram Knowledgebase and returns ``Entity["Country", ...]``.

	* ``"Color"``: Provides an HTML5 color picker and returns a native RGBColor.

	* ``Restricted["Integer", {min, max}]``: Validates that input is an integer within the specified range.

* **Defaults and Labels**: Using ``{"key", "Display Label"} -> type -> defaultValue`` syntax.

* CloudDeploy: Deploys the form as an active web application in the Wolfram Cloud.

* DeleteObject: Removes the deployed cloud object when it is no longer needed.

---

## 2. Define the FormFunction

The form below asks for a country, a highlight color, and a zoom radius in miles. When submitted, it applies the action function to generate a centered GeoGraphics image. Here we are using the form {name, label}->type. 
You can either define your form as a separate formObject, and then insert it into the FormFunction, or simply include it in the form function.

```wl
In[83]:=
formObject = FormObject[{
	    {"country", "Select a Country:"} -> "Country" -> "Japan", 
	    {"color", "Highlight Color:"} -> "Color" -> RGBColor[0.85, 0.2, 0.2], 
	    {"zoom", "Zoom Radius (miles):"} -> Restricted["Integer", {100, 2000}] -> 500
	  }, AppearanceRules -> <|
	    "Title" -> "Country Geo-Focus Form", 
	    "Description" -> "Demonstrating semantic Interpreter types (Country, Color, Restricted).", 
	    "SubmitLabel" -> "Generate Map"
	  |>];

In[84]:=
form = FormFunction[
	  formObject, 
	  Function[params, 
	    GeoGraphics[
	      {params["color"], GeoDisk[params["country"], Quantity[params["zoom"], "Miles"]]}, 
	      GeoRange -> Quantity[params["zoom"] * 1.5, "Miles"], 
	      GeoCenter -> params["country"], 
	      ImageSize -> 500
	    ]
	  ], 
	  "PNG"
	  
	];
```

---

## 3. Deploy to the Wolfram Cloud

Deploy the form to your Wolfram Cloud account. The returned CloudObject contains the live public URL.

```wl
In[85]:= co = CloudDeploy[form, Permissions -> "Public"]

Out[85]= CloudObject["https://www.wolframcloud.com/obj/833140ae-982e-48de-9e51-cc3578dfec24"]
```

You can view the URL directly:

```wl
co["URL"]
```

Click the resulting link or copy the URL into your web browser to test the form. Notice:

1. The country field supports natural language and knowledgebase entities.

2. The color field opens a browser-native color picker.

3. The zoom field enforces bounds between 100 and 2000.

---

## 4. Delete the Cloud Object

Evaluate the following cell to un-deploy and delete the cloud object from your account:

```wl
In[64]:= DeleteObject[co]
```
