<script setup lang="ts">
interface Props {
  modelValue: (number | undefined)[],
  length: number,
  error: boolean,
  errorMessage: string,
  disabled: boolean,
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: () => [],
  length: () => 6,
  error: false,
  errorMessage: '',
  disabled: false,
})
const {disabled, length, modelValue} = toRefs(props);

const emit = defineEmits<{
    'update:modelValue': [val: (number | undefined)[]]
    complete: [otp: string]
  }>();

const safeLength = computed(() => Math.min(Math.max(length.value, 4), 8)) ;

const n = ref(3);
const perRow = computed(() => n.value > 0 ? n.value : safeLength.value);

const inputRefs = useTemplateRef<HTMLInputElement[]>('input-refs')

const onNumberInput = (e:KeyboardEvent, idx:number) => {
  if (e.ctrlKey || e.metaKey) return

  e.preventDefault();
  const newArr = [...modelValue.value];

  if (/^\d$/.test(e.key)) {                                 
    newArr[idx] = +e.key;

    emit('update:modelValue', newArr);
                         
    onInputJumpNext(idx);                                     
  } else if (e.key === 'Backspace') {
    if(modelValue.value[idx]) {
      newArr[idx] = undefined
      emit('update:modelValue', newArr);
    }  else {
      newArr[idx] = undefined;   
      emit('update:modelValue', newArr);                                       
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

const onPaste = (e:ClipboardEvent) => {
  e.preventDefault();
  
  const value =  e.clipboardData?.getData('text') ?? ''; 
  const digits = value.replace(/\D/g, '').slice(0, safeLength.value)  

  modelValue.value = [...digits].map(item => +item);
}

const disabledBtn = computed(()=>{
   return modelValue.value.length !== safeLength.value ||
    modelValue.value.some(item => item === undefined) ||
    disabled.value
})

const onComplete = () => {
  if(disabledBtn.value) return;

  emit('complete', modelValue.value.join(''));
}

</script>
<template>
  <div
    class="min-h-screen bg-gray-100 flex items-center justify-center px-4">
    <div class="flex flex-col items-center gap-4 w-full max-w-xs">
      <div class="grid gap-2 w-full" :style="{ gridTemplateColumns: `repeat(${perRow}, 1fr)` }">
        <div
          v-for="(item, idx) in safeLength"
          class="aspect-square"
        >
          <input
            :value="modelValue[idx]"
            :disabled="disabled"
            ref="input-refs"
            class="w-full h-full border-2 rounded-md text-center bg-white"
            name="otpNumber"
            type="text"
            @keydown="onNumberInput($event, idx)"
            @paste="onPaste"   
          >
        </div>
      </div>
      <span 
        v-if="error"
        class="text-red-500 text-sm"
        v-text="errorMessage" />
      <button
        :disabled="disabledBtn"
        :class="{'cursor-not-allowed': disabledBtn}"
        class="w-full border-2 rounded-md bg-white py-2"
        @click="onComplete"
       >
       Submit
      </button>
    </div>
  </div>
</template>
