# Persistent Forms with FormPage

This notebook demonstrates `FormPage`, which displays both the input form and the computational result on the same web page.

---

## 1. Concepts Demonstrated

- **`FormPage` vs. `FormFunction`**:
  - `FormFunction` displays a form, and upon submission replaces the page entirely with the output.
  - `FormPage` keeps both the input controls and the resulting output visible on the same page. After submission, users can adjust parameters and resubmit immediately.
- **Layout Specification**: The third argument can be `"ResultBelow"` or `"ResultAbove"` to control where the output appears relative to the form.
- **Enumeration and Function Mapping**: Form choices can map human-readable labels to symbolic functions (e.g., `"Sine Wave" -> Sin`).
- **Styling and Theming**: Using `PageTheme -> "Navy"` and `AppearanceRules` to create a polished dashboard look.
- **Cloud Deployment and Cleanup**: Using `CloudDeploy` and `DeleteObject`.

---

## 2. Define the FormPage

This form implements an interactive mathematical waveform visualizer. Users choose a waveform type, frequency multiplier, range, and plot color.

```wl
page = FormPage[
  {
    {"function", "Waveform Type:"} -> {
      "Sine Wave" -> Sin,
      "Damped Sine" -> (Exp[-# / 5] * Sin[#] &),
      "Cosine Wave" -> Cos,
      "Sinc Wave" -> Sinc
    } -> Sin,
    {"frequency", "Frequency Multiplier:"} -> Restricted["Number", {0.1, 10.}] -> 1.0,
    {"range", "Plot Range (0 to X):"} -> Restricted["Number", {2*Pi, 10*Pi}] -> 4*Pi,
    {"lineColor", "Waveform Color:"} -> "Color" -> Blue
  },
  Function[params,
    Plot[
      params["function"][params["frequency"] * x],
      {x, 0, params["range"]},
      PlotStyle -> {Thick, params["lineColor"]},
      PlotRange -> All,
      GridLines -> Automatic,
      Frame -> True,
      FrameLabel -> {"x", "f(x)"},
      PlotLabel -> Style["Waveform Analysis", Bold, 14],
      ImageSize -> 550
    ]
  ],
  "ResultBelow",
  AppearanceRules -> <|
    "Title" -> "Interactive Waveform Visualizer",
    "Description" -> "Adjust parameters below. The plot updates on the same page upon submission.",
    "SubmitLabel" -> "Update Plot"
  |>,
  PageTheme -> "Navy"
];
```

---

## 3. Deploy to the Wolfram Cloud

Deploy the `FormPage` to the Wolfram Cloud with public permissions:

```wl
co = CloudDeploy[page, Permissions -> "Public"]
```

Retrieve and visit the web address:

```wl
co["URL"]
```

Notice when you open the URL:
1. An initial form is displayed.
2. After submitting, your plot appears beneath the form, while the form fields retain your values so you can tweak them and re-submit immediately.

---

## 4. Delete the Cloud Object

To remove the deployed web form from the cloud:

```wl
DeleteObject[co]
```
