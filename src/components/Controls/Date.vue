<template>
  <div>
    <div v-if="showLabel" :class="labelClasses">
      {{ df.label }}
    </div>
    <input
      v-show="showInput"
      :id="componentID"
      ref="input"
      :class="[inputClasses, containerClasses]"
      type="text"
      :value="inputValue"
      :placeholder="inputPlaceholder"
      :readonly="isReadOnly"
      :tabindex="isReadOnly ? '-1' : '0'"
    />

    <date-picker 
      v-show="showInput"
      ref="persianDatePicker"
      @close="showInput=false"
      @change="onChange"
      v-model="selectedDate"
      format="YYYY-MM-DD"
      :show = "showInput"
      :custom-input="datePickerID"
      locale="fa,en"
    />
    
    <div
      v-show="!showInput"
      class="flex"
      :class="[containerClasses, sizeClasses]"
      tabindex="0"
      @click="activateInput"
      @focus="activateInput"
    >
      <p
        v-if="!isEmpty"
        :class="[baseInputClasses]"
        class="overflow-auto no-scrollbar whitespace-nowrap"
      >
        {{ formattedValue }}
      </p>
      <p v-else-if="inputPlaceholder" class="text-base text-gray-500 w-full">
        {{ inputPlaceholder }}
      </p>

      <button v-if="!isReadOnly" class="-me-0.5 ms-1">
        <FeatherIcon
          name="calendar"
          class="w-4 h-4"
          :class="showMandatory ? 'text-red-600' : 'text-gray-600'"
        />
      </button>
    </div>
  </div>
</template>
<script lang="ts">
import { DateTime } from 'luxon';
import { fyo } from 'src/initFyo';
import { defineComponent, nextTick} from 'vue';
import Base from './Base.vue';
import DatePicker from 'vue3-persian-datetime-picker';

let uidCounter = 0;

export default defineComponent({
  extends: Base,
  name: "Date",
  emits: ['input', 'focus'],
  data() {
    return {
      showInput: false,
      selectedDate: this.value,
      componentID: '',
      datePickerID: '',
    };
  },
  mounted (){
    this.componentID = this.$options.name + '-' + uidCounter++;
    this.datePickerID = '#' + this.componentID;
  },
  components: { DatePicker },
  computed: {
    inputValue(): string {
      let value = this.value;
      if (typeof value === 'string') {
        value = new Date(value);
      }

      if (value instanceof Date && !Number.isNaN(value.valueOf())) {
        return DateTime.fromJSDate(value).toISODate();
      }

      return '';
    },
    inputType() {
      return 'date';
    },
    formattedValue() {
      const value = this.parse(this.value);
      //return fyo.format(value, this.df, this.doc);
      return new Date(fyo.format(value, this.df, this.doc)).toLocaleDateString('fa-IR');
    },
    borderClasses(): string {
      if (!this.border) {
        return '';
      }

      const border = 'border border-gray-200';
      let background = 'bg-gray-25';
      if (this.isReadOnly) {
        background = 'bg-gray-50';
      }

      if (this.showInput) {
        return background;
      }

      return border + ' ' + background;
    },
  },
  methods: {
    onChange(e: Event){
      const target = e.target;
      this.showInput = false;
      let value: Date | null = DateTime.fromISO(e.toISOString()).toJSDate();
      if (Number.isNaN(value.valueOf())) {
        value = null;
      }
      this.triggerChange(value);      
    },
    onFocus(e: FocusEvent) {
      const target = e.target;
      if (!(target instanceof HTMLInputElement)) {
        return;
      }

      target.select();
      this.showInput = true;
      this.$emit('focus', e);
    },
    onBlur(e: FocusEvent) {
      const target = e.target;
      if (!(target instanceof HTMLInputElement)) {
        return;
      }
      this.showInput = false;

      let value: Date | null = DateTime.fromISO(target.value).toJSDate();
      if (Number.isNaN(value.valueOf())) {
        value = null;
      }

      this.triggerChange(value);
    },
    activateInput() {
      if (this.isReadOnly) {
        return;
      }

      this.showInput = true;
      nextTick(() => {
        this.focus();

        // @ts-ignore
        //this.$refs.input.showPicker();
      });
    },
  },
});
</script>
