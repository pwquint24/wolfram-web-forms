# Embedding Images in Web Forms

This notebook builds on the earlier examples to show how to embed images into Wolfram Language web forms. Images can appear in three distinct places: (1) the form chrome (title, description, and submit button), (2) inside a custom FormLayoutFunction layout, and (3) as user-uploaded content via the ``"Image"`` interpreter that is then composited into an image result.

---

## 1. Concepts Demonstrated

* Images as non-textual AppearanceRules values: The ``"Title"``, ``"Description"``, and ``"SubmitLabel"`` appearance rules do not have to be plain strings. They accept any renderable Wolfram expression such as Image, Graphics, or Style. The form engine rasterizes these values and embeds them directly in the generated HTML, which makes them ideal for logos, banners, and branded forms.

* Images and graphics inside FormLayoutFunction: Because a FormLayoutFunction simply returns a Wolfram layout expression, you can mix arbitrary graphics and images with form controls using Row, Column, Grid, and Delimiter.

* The ``"Image"`` interpreter: A field declared with interpreter ``"Image"`` renders a file-upload control. On submission, the uploaded file is delivered to the action function as a native Image expression that can be processed, transformed, or composited.

* **Image output**: The third argument of FormFunction or FormPage can be an image format such as ``"PNG"``, ``"JPEG"``, or ``"GIF"``, so the result is returned as an image.

* **Cloud deployment and cleanup**: Using CloudDeploy and DeleteObject.

---

## 2. Example 1 — Branding with a Banner Title and an Inline Logo

First, build two image assets: a rasterized banner for the form title and a small triangular logo for the inline layout.

```wl
In[4]:=
(* A rasterized banner image used as the form's title *)
	banner = Rasterize[
	  Graphics[{
	    {Darker[Blue], Rectangle[{-300, -40}, {300, 40}]}, 
	    Text[Style["ACME CONFERENCE 2024", White, Bold, 26, FontFamily -> "Arial"], {0, 0}], 
	    {Orange, Rectangle[{-300, 40}, {300, 46}]}
	  }, ImageSize -> {600, 92}, PlotRangePadding -> None], 
	  ImageSize -> {600, 92}
	];
	
	(* A small inline logo for the layout *)
	logo = Rasterize[
	  Graphics[{
	    EdgeForm[White], FaceForm[Orange], 
	    RegularPolygon[3], 
	    Text[Style["A", White, Bold, 22], {0, -0.15}]
	  }, ImageSize -> 90, Background -> Darker[Blue]], 
	  ImageSize -> 90
	];
```

Now define a registration form. The banner is passed to ``"Title"`` in AppearanceRules, and the logo is placed in the FormLayoutFunction beside a heading:

```wl
In[27]:=
form = FormFunction[
	  {
	    {"name", "Full Name:"} -> "String", 
	    {"role", "Role:"} -> {"Attendee", "Speaker", "Sponsor", "Staff"}, 
	    {"color", "Badge Color:"} -> "Color" -> RGBColor[0.2, 0.4, 0.8]
	  }, 
	  Function[params, 
	    Panel[
	      Column[{
	        Style["Registration Confirmed", Bold, 18, Darker[Blue]], 
	        Spacer[5], 
	        Row[{Style["Name: ", Bold], params["name"]}], 
	        Row[{Style["Role: ", Bold], params["role"]}], 
	        Row[{Style["Badge Color: ", Bold], 
	          Graphics[{params["color"], Rectangle[]}, ImageSize -> {18, 18}]}]
	      }, Alignment -> Left, Spacings -> 1], 
	      ImageSize -> 360
	    ]
	  ], 
	  AppearanceRules -> <|
	    "Title" -> banner, 
	    "Description" -> Style["Register for the conference by filling out the form below.", Italic, Gray], 
	    "SubmitLabel" -> "Register"
	  |>, 
	  FormLayoutFunction -> Function[
	    Column[{
	      Row[{logo, Spacer[15], Style["Attendee Registration", Bold, 18, Darker[Blue]]}], 
	      Delimiter, 
	      Grid[{
	        {#["name", "Label"], #["name", "Control"]}, 
	        {#["role", "Label"], #["role", "Control"]}, 
	        {#["color", "Label"], #["color", "Control"]}
	      }, Alignment -> Left, Spacings -> {2, 1}]
	    }]
	  ], 
	  PageTheme -> "White"
	];
```

---

## 3. Example 2 — Uploading an Image to Generate an ID Badge

The ``"Image"`` interpreter adds an upload field. The action function receives the uploaded photo as an Image, composites it into a badge graphic, and returns it as a PNG:

```wl
In[1]:=
badgeForm = FormFunction[
	  {
	    {{"photo", "Upload Your Photo:"} -> "Image"}, 
	    {{"name", "Full Name:"} -> "String" -> "Ada Lovelace"}, 
	    {{"role", "Job Title:"} -> "String" -> "Engineer"}, 
	    {{"accent", "Accent Color:"} -> "Color" -> Darker[Blue]}
	  }, 
	  Function[params, 
	    Graphics[{
	      {White, Rectangle[{0, 0}, {4, 5}, RoundingRadius -> 0.25]}, 
	      {params["accent"], Rectangle[{0, 4.2}, {4, 5}, RoundingRadius -> {0.25, 0.25, 0, 0}]}, 
	      Inset[params["photo"], {2, 2.85}, Center, {3.2, 2.7}], 
	      Text[Style[params["name"], Bold, 22, Darker[Blue]], {2, 1.15}], 
	      Text[Style[params["role"], 14, Gray], {2, 0.62}]
	    }, ImageSize -> 420, AspectRatio -> 5 / 4, PlotRangePadding -> None]
	  ], 
	  "PNG", 
	  AppearanceRules -> <|
	    "Title" -> "ID Badge Generator", 
	    "Description" -> "Upload a photo to have it composited into a personalized ID badge.", 
	    "SubmitLabel" -> "Generate Badge"
	  |>, 
	  PageTheme -> "White"
	];
```

---

## 4. Deploy to the Wolfram Cloud

Deploy both forms to the Wolfram Cloud with public permissions:

```wl
In[30]:=
co1 = CloudDeploy[form, Permissions -> "Public"]
	
co2 = CloudDeploy[badgeForm, Permissions -> "Public"]

Out[30]= CloudObject["https://www.wolframcloud.com/obj/4d7be62f-ab9c-4a78-a653-b1edecbe7ae8"]

Out[31]= CloudObject["https://www.wolframcloud.com/obj/daec1b69-388d-4eec-8947-b9e704eeaace"]
```

Retrieve the URLs:

```wl
In[16]:=
co1["URL"]
	
co2["URL"]

Out[16]= CloudObject["https://www.wolframcloud.com/obj/d978423a-ba46-4ed8-be10-bca37378ae31"]["URL"]

Out[17]= CloudObject["https://www.wolframcloud.com/obj/55b74896-c52b-4fb9-85cb-4b7719cb489b"]["URL"]
```

### Try It in the Browser

1. Open ``co1``. Notice the banner image in the title, the styled description, and the logo placed beside the heading in the layout.

2. Open ``co2``. The photo field renders an upload control. Choose an image file, enter a name and role, and submit. The result returns a PNG of the composited ID badge.

---

## 5. Delete the Cloud Objects

When testing is complete, delete both deployments:

```wl
In[28]:=
DeleteObject[co1]
	
DeleteObject[co2]
```
