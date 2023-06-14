# mobx_form_builder

[![pub package](https://img.shields.io/pub/v/mobx_form_builder?style=plastic&logo=flutter)](https://pub.dev/packages/mobx_form_builder)

A Flutter package to make it easy to build, react to and validate forms using [MobX](https://pub.dev/packages/mobx).

## Features

![Working form gif](https://github.com/revelojobs/flutter_form_builder/assets/20102814/888295d6-d45a-44db-a467-02e742845d7d)

- Responsive state and form answer caching on current instance.
- Form validation.
- Abstract ValidationResult and Validator classes to help you integrate with form builder and make your own validations.

## Requirements

This library is designed to work with [MobX](https://pub.dev/packages/mobx) and [MobX Code generation](https://pub.dev/packages/mobx_codegen) as its state management provider. It does not support BLoC, Provider or GetX as of yet.

Make sure to include both dependencies on your project and run build runner once everything is set:

```yml
dependencies:
  mobx: <version>
  mobx_codegen: <version>

```

Running build runner:
```
flutter pub run build_runner build
```

## Usage

1. Add the `mobx_form_builder` package to your [pubspec dependencies](https://pub.dev/packages/flutter_form_builder/install).

2. Import `mobx_form_builder`.
    ```dart
    import 'package:mobx_form_builder/mobx_form_builder.dart';
    ```

3. Apply the mixin to your mobx store, specifying the type of the keys that will be used to retrieve each form field.
    ```dart
    class ExamplePageViewModel extends _ExamplePageViewModelBase with _$ExamplePageViewModel {
      ExamplePageViewModel();
    }

    abstract class _ExamplePageViewModelBase with Store, FormBuilder<String> {
      _ExamplePageViewModelBase();
    }
    ```


3. As soon as the view is ready, make sure to call `setupForm` with a map of FormItems (an entry for each of the inputs):
- The keys of this map will be used to access each specific field and must be of the same type used on `FormBuilder<Type>` such as String, enum, int etc.
- Create FormItems with the type of the input inside the `<>` and use the `FormItem.from` constructor.
- When creating FormItems you should pass its initial value, its validators and `onValidationError` (if needed) to log any errors when validating.

  Example:
  ```dart
  setupForm({
    'firstName': FormItem<String?>.from(
      value: null,
      validators: [
        RequiredFieldValidator(...),
      ],
      onValidationError: _logValidationError,
    ),
    'lastName': FormItem<String?>.from(
      value: null,
      validators: [
        RequiredFieldValidator(...),
      ],
      onValidationError: _logValidationError,
    ),
    'email': FormItem<String?>.from(
      value: null,
      validators: const [],
    ),
  });
  ```

4. Access the fields values and errors in the UI using `getFieldValue<T>(key)` and `getFieldErrorMessage<T>(key)`, either with computed mobx getters or using the FormBuilder's getters directly in the UI.

    ```dart
    /// using computed mobx getters on the store
    @computed
    String? get firstName => getFieldValue<String?>('firstName');

    @computed
    String? get firstNameError => getFieldErrorMessage('firstName');

    /// using directly in your Widget (make sure to wrap it in an Observer if you want to observe to the changes)
    Text(
      'First Name: ${getFieldValue<String?>('email')}',
    ),
    ```

5. Update any field in the form using the inbuilt `updateAndValidateField` and `updateField` methods when the input is updated on your Widget.
    ```dart
    Future<void> updateFirstName(String? newValue) async {
      await updateAndValidateField(newValue, 'firstName');
    }
    ```

6. Quick tip: always validate your entire form before submitting information to the server.
    ```dart
    @action
    Future<void> submitForm() async {
      if (await validateForm()) {
        // submit form
      }
    }
    ```

## Validators
You can create any kind of validator needed specifically for your needs and according to the field type you have. We've included the `RequiredFieldValidator`, but feel free to create more in your project as you need.

### Create your own validators
You can do that by creating a class that extends the `Validator` class. See example below:

Example:

```dart
class EmailValidator extends Validator<String?> {
  final String errorMessage;

  EmailValidator(
    this.errorMessage,
  );

  @override
  Future<ValidatorResult> validate(value) {
    if (value == null || value.isEmpty) {
      return result(isValid: true);
    }

    final isEmailValid = _validateEmail(value);

    return result(
      isValid: isEmailValid,
      errorMessage: errorMessage,
    );
  }

  bool _validateEmail(String email) {
    final regex = RegExp(r'^[^@,\s]+@[^@,\s]+\.[^@,.\s]+$');

    return regex.hasMatch(email);
  }
}

```

> **Note**<br/>
> We recommend avoiding implementing more than one validation in each validator. If the field must be required and a valid email, add two validators, such as [RequiredValidator(), EmailValidator()]. This way you can reuse the email validator if this field ever becomes optional.

## Testing

See [example/test](https://github.com/revelojobs/flutter_form_builder/tree/main/test/form) for testing examples.
