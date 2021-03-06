# Principle

Pledg generates an `eCard`, a debit bank card with the purchase amount, and submit your payment form with this eCard infos.

There is not need of API integration, you just have to send :

- The purchase infos (see the table `Merchant settings`, ex the amount)
- The function to submit your form.

# How to integrate it

## Getting Started

### Html
```html
<script src="https://s3-eu-west-1.amazonaws.com/pledg-assets/pledg-card-integration/latest/plugin.min.js"></script>

<form action="" method="post" id="payment-form"> <!-- form to perform payment -->
    <input type="text" id="card-number" name="card-number">
    <input type="text" id="card-expiry-month" name="card-expiry-month">
    <input type="text" id="card-expiry-year" name="card-expiry-year">
    <input type="text" id="card-cvc" name="card-cvc">
</form>

<!-- button to open pledg tunnel -->
<button id="pledg-button">Pay with friends</div>
```

### Javascript
```javascript
var button = document.querySelector('#pledg-button')

new Pledg(button, {
    // your pledg merchant id
    merchantId: 'mer_uid',
    // the amount **in cents** from your purchase
    amountCents: 6500,
    // the form element to submit to validate the payment
    formElement: document.querySelector('#payment-form')
    // the function that will prefill you card form
    onSuccess: function (eCard) {
        document.querySelector('#card-number').value = eCard.card_number
        document.querySelector('#card-expiry-month').value = eCard.expiry_month
        document.querySelector('#card-expiry-year').value = eCard.expiry_year
        document.querySelector('#card-cvc').value = eCard.cvc
    },
    // the function you can use to handle errors from the eCard
    onError: function (error) {
        // see the "Errors" section of the documentation for more a detailed explanation
    },
})
```

### How it works

The merchant sets a javascript listener to receive and handle the eCard generated by Pledg.

The eCard is generated in the Pledg iframe and sent to the merchant page using the standard API `Window.postMessage`.


## Merchant settings

Create the pledg button like this :
`new Pledg(button, settings)`

As you can see settings are passed as the second argument of the `Pledg` class.

Here is the full list of available settings:

### Required settings

| name | required | type | details |
|--|--|--|--|
| `amountCents` | **true** | `integer` | Amount *in cents* to be credited on the purchase eCard |
| `formElement` | **true** | `element` | Form to automatically submit at the end|
| `merchantUid` | **true** | `string` | Your merchant id |
| `onSuccess` | **true** | `function` | Function that should modify payment form with eCard, takes the generated eCard as its first argument |

### Optional settings

| name | required | type | details |
|--|--|--|--|
| `email` | no  | `string` | Email of the client |
| `onError` | no  | `function` | Function that should handle and display potential errors |
| `subtitle` | no  | `string` | Subtitle of the purchase |
| `title` | no  | `string` |  Title of the purchase |

## CSS

**Button :**
You can customize the Pledg button by
- Changing the text in the `button` element
- Applying style to the element `.pledg-pay-with-pledg`

**Overlay :**
You can customize the the black overlay by appliying style to the element `.pledg-iframe-overlay`
for example change the black background to a white background :
```css
.pledg-iframe-overlay {
    background: rgba(255, 255, 255, 0.3);
}
```

## Errors Handling

An error is an object containing two properties: `type` and `message`.
For example :

```javascript
{
    type: 'payment_refused',
    message: '3Ds refused'
}
```

Errors are all handled in the `onError` function like this :

```javascript
new Pledg(button, {
    // ... Other settings merchantId, amountCents, ...

    onError: function (error) {
        if (error.type === 'payment_refused') {
            // Handle payment_refused errors here
        } else if (error.type === 'invalid_request_error') {
            // Handle invalid_request_error errors here
        } else if (error.type === 'internal_error') {
            // Handle internal_error errors here
        }
    },
})
```

The error types (`error.type`) are the following:

| name | details |
|--|--|
| `internal_error` | Internal error in Pledg (rare) |
| `invalid_request_error` | When your settings are malformed, for example : invalid `merchandId`, invalid `amountCents`, ... |
| `payment_refused` | Pledg decided to not process the payment (ex 3Ds refused) |

# The User Workflow

1) Opening the Pledg payment funnel

Clicking on the Pledg payment button opens the iframe containing the payment funnel.

2) Filling out the emails

The user fills his email and those of his friends.

3) Pre-authorisation for the whole amount of the transaction

The user fills out his card details and Pledg pre-authorises (via Stripe) the total amount of the purchase.

For security purposes, Pledg only stores a reference on the card in order to make the payment later. Pledg does not store the user's card details.

4) Generating the eCard and sending emails

An eCard is generated and it is credited with the total amount of the purchase according to the embedded parameters in the URL.

Pledg sends a validation email to the user and invitation emails to his friends.

5) Directly filling out the eCard details on the merchant’s website

In order to provide a liquid and safe UX, the details of the generated eCard are not visible for the end-user and are directly transferred to the merchant.

The Javascript function retrieves the result of the journey by using the standard API Window.postMessage.

If successful, the function:
- Fills out the payment form
- Validates the payment form

If not :
- the iframe closes and the merchant function handles the error.

6) The merchant is paid

The end-used lands on the merchant's payment confirmation page without having to filling out the the payment form with the generated eCard details or validating the form. themselves.
