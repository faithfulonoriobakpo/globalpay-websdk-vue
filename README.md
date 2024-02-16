# GlobalPay Web SDK for VueJs

This is a VueJs library for integrating GlobalPay Payment Gateway into VueJs applications.

## Installation

You can install this library via npm:

```bash
npm install globalpay-websdk-vue
```

## Example Usage
### Import the GlobalPay Component (and Payload Type if you're using Typescript)

```vue
<script setup lang="ts">
import { GlobalPay } from 'globalpay-websdk-vue';
import { GeneratePaymentLinkPayload, GeneratePaymentLinkError } from 'globalpay-websdk-vue/components/GlobalPay.vue';

const payload: GeneratePaymentLinkPayload = {
    amount: 200,
    secretKey: "your-secret-key",
    merchantTransactionReference: "yfrf534534gce",
    redirectUrl: "www.google.com",
    customer: {
        lastName: "bn",
        firstName: "John",
        currency: "NGN",
        phoneNumber: "081000000000",
        address: "",
        emailAddress: "customer email address"
    }
};

const style = {
    background: 'green',
    color: 'yellow'
};

const erroHandler = (error: GeneratePaymentLinkError) => {console.log(error)}

</script>

<template>

<GlobalPay 
    :isLive="false"
    :apiKey="'TJQO0B60D17MDQD9HK2A'"
    :buttonText="'Pay'"
    :buttonStyle="style"
    :payload="payload"
    :onError="erroHandler"
/>

</template>
```

> buttonText and buttonStyle inputs are optional. buttonText is defaulted to Pay unless specified from prop and buttonStyle is defaulted to a blue background with white text.

> isLive is defaulted to true

> This library is only compatible with Vue3