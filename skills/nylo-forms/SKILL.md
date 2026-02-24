---
name: nylo-forms
description: Use when building forms, adding form fields, validating user input, submitting form data, using NyFormWidget or FormValidator, creating picker/radio/checkbox/slider fields, or implementing form layouts in a Nylo v7 Flutter app.
---

# Nylo Forms

## Overview

Nylo v7 forms are built around `NyFormWidget`, which IS the widget itself (no separate wrapper needed). Forms define fields via a `fields()` method, support 22 field types, provide built-in validation through `FormValidator`, and return data as snake_case `Map<String, dynamic>` on submission.

## When to Use

- Building any form (login, registration, edit profile, settings)
- Adding validated input fields to a page
- Creating picker, radio, checkbox, slider, or date fields
- Validating user input with built-in or custom rules
- Submitting form data to an API
- Laying out fields side-by-side in rows
- When NOT to use: For standalone text input without form context, use `InputField` directly

## Quick Reference

| Action | Code |
|--------|------|
| Create form (CLI) | `metro make:form LoginForm` |
| Text field | `Field.text("Name")` |
| Email field | `Field.email("Email")` |
| Password field | `Field.password("Password", viewable: true)` |
| Number field | `Field.number("Age")` |
| Picker field | `Field.picker("Country", options: FormCollection.from([...]))` |
| Validate not empty | `validator: FormValidator.notEmpty()` |
| Validate email | `validator: FormValidator.email()` |
| Validate password | `validator: FormValidator.password(strength: 2)` |
| Chain validators | `FormValidator().notEmpty().minLength(3).maxLength(20)` |
| Custom validator | `FormValidator.custom(validate: (data) => ..., message: "...")` |
| Submit via actions | `MyForm.actions.submit(onSuccess: (data) { })` |
| Update field | `MyForm.actions.updateField("Name", "Jane")` |
| Clear all fields | `MyForm.actions.clear()` |

## Form Structure

Generate a form with CLI:

```bash
metro make:form LoginForm
```

Every form extends `NyFormWidget`:

```dart
class LoginForm extends NyFormWidget {
  LoginForm({super.key, super.submitButton, super.onSubmit, super.onFailure});

  @override
  fields() => [
    Field.email("Email", validator: FormValidator.email()),
    Field.password("Password", validator: FormValidator.password()),
  ];

  static NyFormActions get actions => const NyFormActions('LoginForm');
}
```

## Field Types

### Text-Based Fields

```dart
Field.text("Name")                                    // Standard text
Field.email("Email")                                  // Email keyboard + filtering
Field.password("Password", viewable: true)            // Obscured + toggle
Field.number("Age", decimal: true)                    // Numeric keyboard
Field.url("Website")                                  // URL keyboard
Field.textArea("Description")                         // Multi-line
Field.phoneNumber("Mobile")                           // Auto-formatted phone
Field.currency("Price", currency: "usd")              // Currency formatted
Field.mask("Phone", mask: "(###) ###-####")           // Pattern mask
Field.capitalizeWords("Full Name")                    // Title case
Field.capitalizeSentences("Bio")                      // Sentence case
```

### Selection Fields

```dart
// Picker (single selection from list)
Field.picker("Category",
  options: FormCollection.from(["Tech", "Science", "Art"]),
)

// Radio buttons
Field.radio("Newsletter",
  options: FormCollection.fromMap({"yes": "Yes", "no": "No"}),
)

// Checkbox (boolean toggle)
Field.checkbox("Accept Terms")

// Switch (boolean toggle)
Field.switchBox("Enable Notifications")

// Chips (multi-select)
Field.chips("Tags",
  options: FormCollection.from(["Flutter", "Dart", "Nylo"]),
)
```

### Range Fields

```dart
// Slider (single value)
Field.slider("Rating",
  style: FieldStyleSlider(min: 0, max: 10),
)

// Range slider (two values)
Field.rangeSlider("Price Range",
  style: FieldStyleRangeSlider(min: 0, max: 1000),
)
```

### Date/Time Fields

```dart
Field.date("Birthday")
Field.datetime("Appointment", firstDate: DateTime(2025))
```

### Special Fields

```dart
// Custom field (child must extend NyFieldStatefulWidget)
Field.custom("My Field", child: MyCustomFieldWidget())

// Embed any widget (no field functionality)
Field.widget(child: Divider())
```

## FormCollection

Required for picker, radio, and chips fields:

```dart
// From list (value and label are the same)
FormCollection.from(["Red", "Green", "Blue"])

// From map (key = value, value = label)
FormCollection.fromMap({"us": "United States", "ca": "Canada"})

// From structured data
FormCollection.fromKeyValue([
  {"value": "en", "label": "English"},
  {"value": "es", "label": "Spanish"},
])
```

Query methods: `getByValue()`, `getLabelByValue()`, `containsValue()`, `searchByLabel()`, `values`, `labels`.

## Validation

### Built-In Validators

```dart
FormValidator.email()                        // Email format
FormValidator.password(strength: 2)          // Strength 1: 8+ chars, upper, digit
                                             // Strength 2: adds special char
FormValidator.notEmpty()                     // Rejects empty
FormValidator.minLength(5)                   // Min string length
FormValidator.maxLength(100)                 // Max string length
FormValidator.minValue(18)                   // Min numeric value
FormValidator.maxValue(100)                  // Max numeric value
FormValidator.url()                          // URL format
FormValidator.numeric()                      // Numeric check
FormValidator.booleanTrue()                  // Must be true
FormValidator.phoneNumberUs()                // US phone format
FormValidator.phoneNumberUk()               // UK phone format
FormValidator.zipcodeUs()                    // US zipcode
FormValidator.date()                         // Valid date
FormValidator.dateInPast()                   // Date in past
FormValidator.dateInFuture()                 // Date in future
FormValidator.dateAgeIsOlder(18)             // Age >= 18
FormValidator.uppercase()                    // All uppercase
FormValidator.lowercase()                    // All lowercase
FormValidator.regex(r'^[A-Z]{3}\d{4}$')     // Regex pattern
```

### Chaining Validators

```dart
Field.text("Username",
  validator: FormValidator()
      .notEmpty(message: "Username is required")
      .minLength(3, message: "At least 3 characters")
      .maxLength(20, message: "At most 20 characters"),
)
```

### Custom Validator

```dart
Field.number("Age",
  validator: FormValidator.custom(
    message: "Age must be between 18 and 100",
    validate: (data) {
      int? age = int.tryParse(data.toString());
      return age != null && age >= 18 && age <= 100;
    },
  ),
)
```

### Reusable Custom Rule

```dart
class FormRuleUsername extends FormRule {
  @override
  String? rule = "username";

  @override
  String? message = "The {{attribute}} must be a valid username.";

  FormRuleUsername({String? message}) {
    if (message != null) this.message = message;
  }

  @override
  bool validate(data) {
    if (data is! String) return false;
    return RegExp(r'^[a-zA-Z0-9_]{3,20}$').hasMatch(data);
  }
}

// Apply with FormValidator.rule()
FormValidator validator = FormValidator.rule([
  FormRuleNotEmpty(),
  FormRuleUsername(),
]);
```

### Page-Level Validation with check()

Validate outside of forms using `check()` in NyPage:

```dart
check((validate) {
  validate.that(_emailController.text, label: "Email").email();
  validate.that(_passwordController.text, label: "Password")
      .notEmpty()
      .password(strength: 2);
}, onSuccess: () {
  _submitForm();
}, onValidationError: (FormValidationResponseBag bag) {
  print(bag.firstErrorMessage);
});
```

## Displaying Forms

The form IS the widget, use it directly in your build method:

```dart
@override
Widget view(BuildContext context) {
  return Scaffold(
    body: SafeArea(
      child: LoginForm(
        submitButton: Button.primary(text: "Login"),
        onSubmit: (data) {
          // data = {"email": "...", "password": "..."}
          print(data);
        },
        onFailure: (errors) {
          print(errors.first.rule.getMessage());
        },
      ),
    ),
  );
}
```

### Form Constructor Parameters

| Parameter | Type | Purpose |
|-----------|------|---------|
| `submitButton` | `Widget?` | Submit button widget |
| `onSubmit` | `Function(Map)?` | Success callback with form data |
| `onFailure` | `Function(List)?` | Validation failure callback |
| `initialData` | `Map<String, dynamic>?` | Pre-populate fields |
| `onChanged` | `Function(Field, dynamic)?` | Field change callback |
| `crossAxisSpacing` | `double` | Horizontal spacing between row fields |
| `mainAxisSpacing` | `double` | Vertical spacing between fields |
| `header` | `Widget?` | Widget above form |
| `footer` | `Widget?` | Widget below form |
| `locked` | `bool` | Make entire form read-only |
| `loadingStyle` | `LoadingStyle?` | Loading indicator style |

## Submission

### Method 1: onSubmit Callback

```dart
LoginForm(
  submitButton: Button.primary(text: "Login"),
  onSubmit: (data) {
    // {email: "user@email.com", password: "pass123"}
  },
)
```

### Method 2: NyFormActions

```dart
LoginForm.actions.submit(
  onSuccess: (data) { print(data); },
  onFailure: (errors) { print(errors.first.rule.getMessage()); },
  showToastError: true,
);
```

### Method 3: Static Submit

```dart
NyFormWidget.submit("LoginForm",
  onSuccess: (data) { print(data); },
);
```

## Initial Data and Dynamic Fields

### Using init Getter

```dart
class EditAccountForm extends NyFormWidget {
  @override
  Function()? get init => () async {
    final user = await api<ApiService>((r) => r.getUserData());
    final countries = await api<ApiService>((r) => r.getCountries());
    return {
      "First Name": user?.firstName,
      "Last Name": user?.lastName,
      "Country": define(value: user?.country, options: countries),
    };
  };
  // ...
}
```

Use `define()` for fields that need both a value and dynamic options (pickers, chips, radios).

### Using initialData Parameter

```dart
EditAccountForm(
  initialData: {
    "first_name": "John",
    "last_name": "Doe",
  },
)
```

## Form Actions (NyFormActions)

```dart
static NyFormActions get actions => const NyFormActions('MyForm');
```

| Method | Purpose |
|--------|---------|
| `updateField(key, value)` | Set a field's value |
| `clearField(key)` | Clear a specific field |
| `clear()` | Clear all fields |
| `refresh()` | Refresh form UI state |
| `refreshForm()` | Re-call `fields()` and rebuild entirely |
| `setOptions(key, options)` | Update picker/chip/radio options |
| `submit(onSuccess:, onFailure:)` | Submit with validation |

## Form Layout

Place fields in a list to render them side-by-side in a row:

```dart
@override
fields() => [
  Field.text("Title"),

  // Two fields in one row
  [
    Field.text("First Name"),
    Field.text("Last Name"),
  ],

  Field.textArea("Bio"),
  Field.widget(child: Divider()),
  Field.email("Email"),
];
```

## Field Styling

Each field type has a corresponding style class:

```dart
Field.text("Name",
  style: FieldStyleTextField(
    filled: true,
    fillColor: Colors.grey.shade100,
    border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
    contentPadding: EdgeInsets.symmetric(horizontal: 16, vertical: 12),
    prefixIcon: Icon(Icons.person),
  ),
)
```

### Picker Styles

```dart
Field.picker("Country",
  options: FormCollection.from(["US", "CA", "UK"]),
  style: FieldStylePicker(
    listTileStyle: PickerListTileStyle.radio(activeColor: Colors.blue),
    // Or: PickerListTileStyle.checkmark(activeColor: Colors.green)
    // Or: PickerListTileStyle.custom(builder: (option, isSelected, onTap) => ...)
  ),
)
```

## Complete Example

```dart
class RegisterForm extends NyFormWidget {
  RegisterForm({super.key, super.submitButton, super.onSubmit, super.onFailure});

  @override
  fields() => [
    [
      Field.text("First Name", validator: FormValidator.notEmpty()),
      Field.text("Last Name", validator: FormValidator.notEmpty()),
    ],
    Field.email("Email", validator: FormValidator.email()),
    Field.password("Password",
      viewable: true,
      validator: FormValidator.password(strength: 2),
    ),
    Field.phoneNumber("Phone",
      validator: FormValidator.phoneNumberUs(),
    ),
    Field.picker("Country",
      options: FormCollection.from(["US", "CA", "UK", "AU"]),
      validator: FormValidator.notEmpty(),
    ),
    Field.checkbox("Accept Terms",
      validator: FormValidator.booleanTrue(
        message: "You must accept the terms"),
    ),
  ];

  static NyFormActions get actions => const NyFormActions('RegisterForm');
}

// Usage in page
@override
Widget view(BuildContext context) {
  return Scaffold(
    body: SafeArea(
      child: RegisterForm(
        submitButton: Button.primary(text: "Register"),
        onSubmit: (data) async {
          // data = {first_name, last_name, email, password, phone, country, accept_terms}
          await api<AuthApiService>((r) => r.register(data));
          routeTo(HomePage.path);
        },
        onFailure: (errors) {
          showToastDanger(description: errors.first.rule.getMessage());
        },
      ),
    ),
  );
}
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using raw lists for picker/radio/chips options | Must use `FormCollection.from()`, `.fromMap()`, or `.fromKeyValue()` |
| Field key mismatch with initialData | Field keys are title-cased ("First Name"), but initialData uses snake_case ("first_name"); both formats work for initialData |
| Not defining `static NyFormActions get actions` | Required for using `MyForm.actions.submit()` and other programmatic interactions |
| Forgetting `define()` for dynamic picker options in init | When setting both value and options in `init`, wrap with `define(value: ..., options: ...)` |
| Expecting non-null from onSubmit data | Form data values can be null if fields are left empty; always handle nulls |
| Using `FormValidator.password()` without specifying strength | Defaults to strength 1 (8+ chars, 1 upper, 1 digit); use `strength: 2` for special char requirement |
| Placing `Field.widget()` and expecting form data | `Field.widget()` is for embedding non-field widgets like dividers; it does not participate in form data |
| Not calling `super.onTap(index)` equivalent for custom submit | When using a custom footer button, call `MyForm.actions.submit()` explicitly |
