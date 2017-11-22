![Final Form](banner.png)

✅ **Zero** dependencies

✅ Framework agnostic

✅ Opt-in subscriptions

✅ Tiny! **9.9k (3.5k gzipped)**

---

## Installation

```bash
npm install --save final-form
```

or

```bash
yarn add final-form
```

## Getting Started

🏁 Final Form works on subscriptions to updates based on the
[Observer pattern](https://en.wikipedia.org/wiki/Observer_pattern). Both form
and field subscribers must specify exactly which parts of the form state they
want to receive updates about.

```js
import { createForm } from 'final-form'
import type { FormState, FieldState } from 'final-form'

// Create Form
const form = createForm({
  initialValues,
  onSubmit, // required
  validate
})

// Subscribe to form state updates
const unsubscribe = form.subscribe(
  (state: FormState) => {
    // Update UI
  },
  { // FormSubscription: the list of values you want to be updated about
    dirty: true,
    valid: true,
    values: true
  }
})

// Subscribe to field state updates
const unregisterField = form.registerField(
  'username',
  (fieldState: FieldState) => {
    // Update field UI
    const { blur, change, focus, ...rest } = fieldState
    // In addition to the values you subscribe to, field state also
    // includes functions that your inputs need to update their state.
  },
  { // FieldSubscription: the list of values you want to be updated about
    active: true,
    dirty: true,
    touched: true,
    valid: true,
    value: true
  }
)

// Submit
form.submit() // only submits if all validation passes
```

## Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->

<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

* [API](#api)
  * [`createForm: (config: Config) => FormApi`](#createform-config-config--formapi)
  * [`fieldSubscriptionItems: string[]`](#fieldsubscriptionitems-string)
  * [`formSubscriptionItems: string[]`](#formsubscriptionitems-string)
  * [`FORM_ERROR: Symbol`](#form_error-symbol)
* [Types](#types)
  * [`Config`](#config)
    * [`initialValues?: Object`](#initialvalues-object)
    * [`onSubmit: (values: Object, callback: ?(errors: ?Object) => void) => ?Object | Promise<?Object>`](#onsubmit-values-object-callback-errors-object--void--object--promiseobject)
    * [`validate?: (values: Object, callback: ?(errors: Object) => void) => Object](#validate-values-object-callback-errors-object--void--object)
    * [`debug?: (state: FormState, fieldStates: { [string]: FieldState }) => void`](#debug-state-formstate-fieldstates--string-fieldstate---void)
  * [`FieldState`](#fieldstate)
    * [`active?: boolean`](#active-boolean)
    * [`blur: () => void`](#blur---void)
    * [`change: (value: any) => void`](#change-value-any--void)
    * [`dirty?: boolean`](#dirty-boolean)
    * [`error?: any`](#error-any)
    * [`focus: () => void`](#focus---void)
    * [`initial?: any`](#initial-any)
    * [`invalid?: boolean`](#invalid-boolean)
    * [`name: string`](#name-string)
    * [`pristine?: boolean`](#pristine-boolean)
    * [`submitError?: any`](#submiterror-any)
    * [`submitFailed?: boolean`](#submitfailed-boolean)
    * [`submitSucceeded?: boolean`](#submitsucceeded-boolean)
    * [`touched?: boolean`](#touched-boolean)
    * [`valid?: boolean`](#valid-boolean)
    * [`visited?: boolean`](#visited-boolean)
  * [`FieldSubscriber: (state: FieldState) => void`](#fieldsubscriber-state-fieldstate--void)
  * [`FieldSubscription: { [string]: boolean }`](#fieldsubscription--string-boolean-)
  * [`FormApi`](#formapi)
    * [`batch: (fn: () => void) => void)`](#batch-fn---void--void)
    * [`blur: (name: string) => void`](#blur-name-string--void)
    * [`change: (name: string, value: ?any) => void`](#change-name-string-value-any--void)
    * [`focus: (name: string) => void`](#focus-name-string--void)
    * [`initialize: (values: Object) => void`](#initialize-values-object--void)
    * [`getState: () => FormState`](#getstate---formstate)
    * [`submit: () => ?Promise<?Object>`](#submit---promiseobject)
    * [`subscribe: (subscriber: FormSubscriber, subscription: FormSubscription) =>](#subscribe-subscriber-formsubscriber-subscription-formsubscription-)
    * [`registerField: (name: string, subscriber: FieldSubscriber, subscription:](#registerfield-name-string-subscriber-fieldsubscriber-subscription)
    * [`reset: () => void`](#reset---void)
  * [`FormState`](#formstate)
    * [`active?: string`](#active-string)
    * [`dirty?: boolean`](#dirty-boolean-1)
    * [`error?: any`](#error-any-1)
    * [`invalid?: boolean`](#invalid-boolean-1)
    * [`initialValues?: Object`](#initialvalues-object-1)
    * [`pristine?: boolean`](#pristine-boolean-1)
    * [`submitting?: boolean`](#submitting-boolean)
    * [`submitFailed?: boolean`](#submitfailed-boolean-1)
    * [`submitError?: any`](#submiterror-any-1)
    * [`valid?: boolean`](#valid-boolean-1)
    * [`validating?: boolean`](#validating-boolean)
    * [`values?: Object`](#values-object)
  * [`FormSubscriber: (state: FieldState) => void`](#formsubscriber-state-fieldstate--void)
  * [`FormSubscription: { [string]: boolean }`](#formsubscription--string-boolean-)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## API

The following can be imported from `final-form`.

### `createForm: (config: Config) => FormApi`

Creates a form instance. It takes a `Config` and returns a `FormApi`.

### `fieldSubscriptionItems: string[]`

An _à la carte_ list of all the possible things you can subscribe to for a
field. Useful for subscribing to everything.

### `formSubscriptionItems: string[]`

An _à la carte_ list of all the possible things you can subscribe to for a form.
Useful for subscribing to everything.

### `FORM_ERROR: Symbol`

A special `Symbol` key used to return a whole-form error inside error objects
returned from validation or submission.

## Types

### `Config`

#### `initialValues?: Object`

The initial values of your form. These will be used to compare against the
current values to calculate `pristine` and `dirty`.

#### `onSubmit: (values: Object, callback: ?(errors: ?Object) => void) => ?Object | Promise<?Object>`

Function to call when the form is submitted. There are three possible ways to
write an `onSubmit` function:

* Synchronously: returns `undefined` on success, or an `Object` of submission
  errors on failure
* Asynchronously with a callback: returns `undefined`, calls `callback()` with
  no arguments on success, or with an `Object` of submission errors on failure.
* Asynchronously with a `Promise`: returns a `Promise<?Object>` that resolves
  with no value on success or _resolves_ with an `Object` of submission errors
  on failure. The reason it _resolves_ with errors is to leave _rejection_ for
  when there is a server or communications error.

Submission errors must be in the same shape as the values of the form. You may
return a generic error for the whole form (e.g. `'Login Failed'`) using the
special `FORM_ERROR` symbol key.

#### `validate?: (values: Object, callback: ?(errors: Object) => void) => Object

| void`

A whole-record validation function that takes all the values of the form and
returns any validation errors. There are three possible ways to write a
`validate` function:

* Synchronously: returns `{}` or `undefined` when the values are valid, or an
  `Object` of validation errors when the values are invalid.
* Asynchronously with a callback: returns `undefined`, calls `callback()` with
  no arguments on success, or with an `Object` of validation errors on failure.
* Asynchronously with a `Promise`: returns a `Promise<?Object>` that resolves
  with no value on success or _resolves_ with an `Object` of validation errors
  on failure. The reason it _resolves_ with errors is to leave _rejection_ for
  when there is a server or communications error.

Validation errors must be in the same shape as the values of the form. You may
return a generic error for the whole form using the special `FORM_ERROR` symbol
key.

#### `debug?: (state: FormState, fieldStates: { [string]: FieldState }) => void`

An optional callback for debugging that returns the form state and the states of
all the fields. It's called _on every state change_. A typical thing to pass in
might be `console.log`.

### `FieldState`

`FieldState` is an object containing:

#### `active?: boolean`

Whether or not the field currently has focus.

#### `blur: () => void`

A function to blur the field (mark it as inactive).

#### `change: (value: any) => void`

A function to change the value of the field.

#### `dirty?: boolean`

`true` when the value of the field is `!==` the initial value, `false` if the
values are `===`.

#### `error?: any`

The current validation error for this field.

#### `focus: () => void`

A function to focus the field (mark it as active).

#### `initial?: any`

The initial value of the field. `undefined` if it was never initialized.

#### `invalid?: boolean`

`true` if the field has a validation error or a submission error. `false`
otherwise.

#### `name: string`

The name of the field.

#### `pristine?: boolean`

`true` if the current value is `===` to the initial value, `false` if the values
are `!===`.

#### `submitError?: any`

The submission error for this field.

#### `submitFailed?: boolean`

`true` if a form submission has been tried and failed. `false` otherwise.

#### `submitSucceeded?: boolean`

`true` if the form has been successfully submitted. `false` otherwise.

#### `touched?: boolean`

`true` if this field has ever gained and lost focus. `false` otherwise. Useful
for knowing when to display error messages.

#### `valid?: boolean`

`true` if this field has no validation or submission errors. `false` otherwise.

#### `visited?: boolean`

`true` if this field has ever gained focus.

### `FieldSubscriber: (state: FieldState) => void`

### `FieldSubscription: { [string]: boolean }`

`FieldSubscription` is an object containing the following:

####`active?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the `active`
value in `FieldState`.

####`dirty?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the `dirty`
value in `FieldState`.

####`error?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the `error`
value in `FieldState`.

####`initialValues?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the
`initialValues` value in `FieldState`.

####`invalid?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the `invalid`
value in `FieldState`.

####`pristine?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the `pristine`
value in `FieldState`.

####`submitting?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the
`submitting` value in `FieldState`.

####`submitFailed?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the
`submitFailing` value in `FieldState`.

####`submitSucceeded?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the
`submitSucceeded` value in `FieldState`.

####`valid?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the `valid`
value in `FieldState`.

####`validating?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the
`validating` value in `FieldState`.

####`values?: boolean`

When `true` the `FieldSubscriber` will be notified of changes to the `values`
value in `FieldState`.

### `FormApi`

#### `batch: (fn: () => void) => void)`

Allows batch updates by silencing notifications while the `fn` is running.
Example:

```js
form.batch(() => {
  form.change('firstName', 'Erik') // listeners not notified
  form.change('lastName', 'Rasmussen') // listeners not notified
}) // NOW all listeners notified
```

#### `blur: (name: string) => void`

Blurs (marks inactive) the given field.

#### `change: (name: string, value: ?any) => void`

Changes the value of the given field.

#### `focus: (name: string) => void`

Focuses (marks active) the given field.

#### `initialize: (values: Object) => void`

Initializes the form to the values provided. All the values will be set to these
values, and `dirty` and `pristine` will be calculated by performing a
shallow-equals between the current values and the values last initialized with.
The form will be `pristine` after this call.

#### `getState: () => FormState`

A way to request the current state of the form without subscribing.

#### `submit: () => ?Promise<?Object>`

Submits the form if there are currently no validation errors. It may return
`undefined` or a `Promise` depending on the nature of the `onSubmit`
configuration value given to the form when it was created.

#### `subscribe: (subscriber: FormSubscriber, subscription: FormSubscription) =>

Unsubscribe`

Subscribes to changes to the form. **The `subscriber` will _only_ be called when
values specified in `subscription` change.** A form can have many subscribers.

#### `registerField: (name: string, subscriber: FieldSubscriber, subscription:

FieldSubscription, validate?: (value: ?any, allValues: Object)) => Unsubscribe`

Registers a new field and subscribes to changes to it. **The `subscriber` will
_only_ be called when the values specified in `subscription` change.** More than
one subscriber can subscribe to the same field.

#### `reset: () => void`

Resets the values back to the initial values the form was initialized with. Or
empties all the values if the form was not initialized.

### `FormState`

#### `active?: string`

The name of the currently active field. `undefined` if none are active.

#### `dirty?: boolean`

`true` if the form values are different from the values it was initialized with.
`false` otherwise. Comparison is done with shallow-equals.

#### `error?: any`

The whole-form error returned by a validation function under the `FORM_ERROR`
key.

#### `invalid?: boolean`

`true` if any of the fields or the form has a validation or submission error.
`false` otherwise.

#### `initialValues?: Object`

The values the form was initialized with. `undefined` if the form was never
initialized.

#### `pristine?: boolean`

`true` if the form values are the same as the initial values. `false` otherwise.
Comparison is done with shallow-equals.

#### `submitting?: boolean`

`true` if the form is currently being submitted asynchronously. `false`
otherwise.

#### `submitFailed?: boolean`

`true` if the form was submitted, but the submission failed with submission
errors. `false` otherwise.

####`submitSucceeded?: boolean`

`true` if the form was successfully submitted. `false` otherwise.

#### `submitError?: any`

The whole-form submission error returned by `onSubmit` under the `FORM_ERROR`
key.

#### `valid?: boolean`

`true` if neither the form nor any of its fields has a validation or submission
error. `false` otherwise.

#### `validating?: boolean`

`true` if the form is currently being validated asynchronously. `false`
otherwise.

#### `values?: Object`

The current values of the form.

### `FormSubscriber: (state: FieldState) => void`

### `FormSubscription: { [string]: boolean }`

`FormSubscription` is an object containing the following:

####`active`

When `true` the `FormSubscriber` will be notified of changes to the `active`
value in `FormState`.

####`dirty`

When `true` the `FormSubscriber` will be notified of changes to the `dirty`
value in `FormState`.

####`error`

When `true` the `FormSubscriber` will be notified of changes to the `error`
value in `FormState`.

####`initialValues`

When `true` the `FormSubscriber` will be notified of changes to the
`initialValues` value in `FormState`.

####`invalid`

When `true` the `FormSubscriber` will be notified of changes to the `invalid`
value in `FormState`.

####`pristine`

When `true` the `FormSubscriber` will be notified of changes to the `pristine`
value in `FormState`.

####`submitting`

When `true` the `FormSubscriber` will be notified of changes to the `submitting`
value in `FormState`.

####`submitFailed`

When `true` the `FormSubscriber` will be notified of changes to the
`submitFailed` value in `FormState`.

####`submitSucceeded`

When `true` the `FormSubscriber` will be notified of changes to the
`submitSucceeded` value in `FormState`.

####`valid`

When `true` the `FormSubscriber` will be notified of changes to the `valid`
value in `FormState`.

####`validating`

When `true` the `FormSubscriber` will be notified of changes to the `validating`
value in `FormState`.

####`values`

When `true` the `FormSubscriber` will be notified of changes to the `values`
value in `FormState`.