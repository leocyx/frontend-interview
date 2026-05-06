<script setup lang="ts">
const disabled = ref<boolean>(false);
const errorMessage = ref<string>('');
const isErrorShow = ref<boolean>(false);

const otpNumber = ref<(number | undefined)[]>([]);

interface FetchResult {
  success: boolean,
  data: {
    verified: boolean,
    timestamp: string,
  }
}
const onComplete = async (otp:string) => {
  disabled.value = true;
  errorMessage.value = '';
  isErrorShow.value = false;

  try { 
    const result = await $fetch<FetchResult>('/api/examples/verify-otp-simple',
    {                                                            
      method: 'POST',                                         
      body: { otp }                                   
    });
    
    if(!result.data.verified){
      isErrorShow.value = true;
      errorMessage.value = "Invalid code";
    } else alert("驗證成功");
  } catch (error){                                                     
    throw error;
  } finally {                                                   
    disabled.value = false                                      
  } 
}
</script>

<template>
  <BaseInputOtp
    v-model="otpNumber"
    :disabled="disabled"
    :error="isErrorShow"
    :errorMessage
    :length="6" 
    @complete="onComplete" />
</template>
