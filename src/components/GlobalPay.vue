<template>
  <button
    @click="generatePaymentLink"
    :style="props.buttonStyle"
    :class="{ 'fade': loading }"
    class="pay-button"
  >
    {{ props.buttonText }}
    &nbsp;
    <span v-if="loading" class="loading-animation"></span>
  </button>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const loading = ref(false);

const props = withDefaults(defineProps<{
  isLive: boolean,
  buttonText?: string,
  buttonStyle?: Record<string, any>,
  apiKey: string,
  payload: GeneratePaymentLinkPayload,
  onError: (error: GeneratePaymentLinkError) => void,
}>(), {isLive: true, buttonText: 'Pay'});

export interface GeneratePaymentLinkPayload {
  amount: number,
  secretKey: string,
  merchantTransactionReference: string,
  redirectUrl: string,
  customer: {
    lastName: string,
    firstName: string,
    currency: string,
    phoneNumber: string,
    address: string,
    emailAddress: string
  }
}

export interface GeneratePaymentLinkError {
  message: string,
  details: any
}

interface GeneratePaymentLinkResponse {
  data: {
    checkoutUrl: string,
    accessCode: string,
    redirectURL: string,
    transactionReference: string,
    merchantMode: string,
    merchantCurrencies: string[]
  },
  successMessage: string,
  responseCode: string,
  isSuccessful: boolean,
  error: any
}

const liveUrl = 'https://paygw.globalpay.com.ng/globalpay-paymentgateway/api/paymentgateway/generate-payment-link';
const testUrl = 'https://newwebservicetest.zenithbank.com:8443/new-globalpay-paymentgateway-external/api/paymentgateway/generate-payment-link';

const generatePaymentLink = () => {

  const emptyKeys = findEmptyKeys(props.payload);

  if(!props.apiKey){
    return props.onError({message: 'API Key is required', details:null});
  }else if(!props.payload){
    return props.onError({message: 'Payload is required', details:null});
  }else if(emptyKeys.length > 0){
    const requiredFields = emptyKeys.join(', ').replace(/, (?!.*, )/, ' and ')
    return props.onError({message: `Payload is invalid. ${requiredFields} required`, details: null});
  }

  loading.value = true;

  fetch(
    props.isLive? liveUrl : testUrl,
    {
        method: 'POST',
        headers: {
          'Language': 'en',
          'Content-Type': 'application/json',
          'apiKey': props.apiKey,
        },
        body: JSON.stringify(props.payload),
    }
  )
  .then(
    async response => {
      if (!response.ok) {
        const errorResponse = await response.clone().json();
        throw JSON.stringify(errorResponse);
      }
      return response.json();
    }
  )
  .then((res: GeneratePaymentLinkResponse) => {
    const redirectURL = res?.data?.checkoutUrl;
    if(redirectURL) window.location.href = redirectURL;
  })
  .catch(
    error => {
      console.error(JSON.parse(error));
      props.onError(
        {
          message: 'error occurred generating paymentlink',
          details: JSON.parse(error)
        }
      );
    })
  .finally(() => loading.value = false);
};

const findEmptyKeys = (payload: any): string[] => {

const emptyKeys: string[] = [];

for (const key in payload) {

  if(key == 'address') continue;

  if (payload.hasOwnProperty(key)) {
    if (payload[key] === null || payload[key] === undefined || payload[key] === '') {
      emptyKeys.push(key);
    } else if (typeof payload[key] === 'object') {
      const subEmptyKeys = findEmptyKeys(payload[key]);
      emptyKeys.push(...subEmptyKeys);
    }
  }
}
return emptyKeys;
}
</script>

<style>
.pay-button {
  background-color: #007bff;
  display: flex;
  align-items: center;
  color: #fff;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 4px;
  font-weight: bold;
}

.pay-button.fade {
  opacity: 0.5;
}

.loading-animation {
  border: 2px solid rgba(0, 0, 0, 0.1);
  border-left-color: #fff;
  border-radius: 50%;
  width: 12px;
  height: 12px;
  animation: spin 1s linear infinite;
  display: inline-block;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>