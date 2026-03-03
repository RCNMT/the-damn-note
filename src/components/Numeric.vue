<template>
  <div class="numeric-input">
    <button 
      type="button"
      @click="decrement" 
      :disabled="modelValue <= min"
      class="stepper-btn"
    >
      −
    </button>
    
    <input
      type="number"
      :value="modelValue"
      @input="updateValue($event.target.value)"
      :min="min"
      :max="max"
      class="numeric-field"
    />
    
    <button 
      type="button"
      @click="increment" 
      :disabled="modelValue >= max"
      class="stepper-btn"
    >
      +
    </button>
  </div>
</template>

<script>
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'NumericInput',
  props: {
    modelValue: {
      type: Number,
      default: 0
    },
    min: {
      type: Number,
      default: 0
    },
    max: {
      type: Number,
      default: 100
    },
    step: {
      type: Number,
      default: 1
    }
  },
  emits: ['update:modelValue'],
  setup(props, { emit }) {
    const increment = () => {
      if (props.modelValue < props.max) {
        emit('update:modelValue', props.modelValue + props.step);
      }
    };

    const decrement = () => {
      if (props.modelValue > props.min) {
        emit('update:modelValue', props.modelValue - props.step);
      }
    };

    const updateValue = (value) => {
      const num = parseInt(value, 10);
      if (!isNaN(num)) {
        const clamped = Math.max(props.min, Math.min(props.max, num));
        emit('update:modelValue', clamped);
      }
    };

    return {
      increment,
      decrement,
      updateValue
    };
  }
});
</script>

<style scoped>
.numeric-input {
  display: flex;
  align-items: center;
  gap: 0;
  background: #333;
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid #555;
  width: fit-content;
}

.stepper-btn {
  width: 40px;
  height: 43px;
  background: #444;
  border: none;
  color: #fff;
  font-size: 1.2rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.stepper-btn:hover:not(:disabled) {
  background: #555;
}

.stepper-btn:active:not(:disabled) {
  background: #666;
}

.stepper-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.numeric-field {
  width: 60px;
  height: 40px;
  background: #333;
  border: none;
  border-left: 1px solid #555;
  border-right: 1px solid #555;
  color: #fff;
  font-size: 1rem;
  font-weight: 600;
  text-align: center;
}

.numeric-field::-webkit-outer-spin-button,
.numeric-field::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.numeric-field:focus {
  outline: none;
  background: #444;
}

/* responsive adjustments */
@media (max-width: 640px) {
  .numeric-input {
    width: 100%;
  }

  .numeric-field {
    width: 50px;
  }

  .stepper-btn {
    width: 32px;
    height: 36px;
    font-size: 1rem;
  }
}
</style>