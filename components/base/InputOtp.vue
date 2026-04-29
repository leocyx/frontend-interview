<script setup lang="ts">
interface Props {
  length: number,
}

const props = withDefaults(defineProps<Props>(), {
  length: () => 6
})
const {length} = toRefs(props);

const safeLength = computed(() => Math.min(Math.max(length.value, 4), 8)) ;

const otpNumber = ref<(number | undefined)[]>([]);

const inputRefs = useTemplateRef<HTMLInputElement[]>('input-refs')

const onNumberInput = (e:KeyboardEvent, idx:number) => {
  e.preventDefault();
  
  if (/^\d$/.test(e.key)) {                                 
    otpNumber.value[idx] = +e.key;                            
    onInputJumpNext(idx);                                     
  } else if (e.key === 'Backspace') {
    if(otpNumber.value[idx]) {
      otpNumber.value[idx] = undefined
    }  else {
      otpNumber.value[idx-1] = undefined;                                                  
      onInputJumpBefore(idx);
    }                               
  } else if (e.key === 'Tab') {                               
    onInputJumpNext(idx);                                     
  }
};

const onInputJumpBefore = (idx:number) => {
  inputRefs.value?.[idx-1]?.focus();
}

const onInputJumpNext = (idx:number) => {
  inputRefs.value?.[idx+1]?.focus();
}
</script>
<template>
  <div
    class="min-h-screen bg-gray-100 flex items-center justify-center px-4">
    <div class="flex flex-col items-center gap-4 w-full max-w-xs">
      <div class="flex gap-2 w-full">
        <div
          v-for="(item, idx) in safeLength"
          class="flex-1 aspect-square"
        >
          <input
            :value="otpNumber[idx]"
            ref="input-refs"
            class="w-full h-full border-2 rounded-md text-center bg-white"
            name="otpNumber"
            type="text"
            @keydown="onNumberInput($event, idx)"
          >
        </div>
      </div>
      <span class="text-red-500 text-sm">Invalid code</span>
      <button class="w-full border-2 rounded-md bg-white py-2 ">Submit</button>
    </div>
  </div>
</template>
