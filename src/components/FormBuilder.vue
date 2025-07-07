<template>
  <Form @submit.prevent="emit('submit', formData)" class="dynamic-form">
    <div
      v-for="item in formItems"
      :key="item.name"
      :class="['form-item', item.layout || 'full-width']"
    >
      <label :for="item.name"
        >{{ item.label }} <span v-if="item.required">*</span></label
      >
      <InputText
        v-if="['text', 'email', 'password'].includes(item.type)"
        :id="item.name"
        :type="item.type"
        :name="item.name"
        v-model="formData[item.name] as string"
        :placeholder="item.placeholder"
      />

      <InputNumber
        v-else-if="item.type === 'number'"
        :id="item.name"
        :name="item.name"
        :type="item.type"
        v-model="formData[item.name] as number"
        :placeholder="item.placeholder"
      />

      <div v-else-if="item.type === 'radio'" :id="item.name">
        <label v-for="opt in item.options" :key="opt.value">
          <RadioButton
            v-model="formData[item.name]"
            :value="opt.value"
            :name="item.name"
          />
          {{ opt.label }}
        </label>
      </div>

      <Select
        v-else-if="item.type === 'select'"
        :id="item.name"
        :name="item.name"
        v-model="formData[item.name]"
        :options="item.options"
        optionLabel="label"
        optionValue="value"
        :placeholder="item.placeholder"
      />

      <MultiSelect
        v-else-if="item.type === 'multiselect'"
        :id="item.name"
        :name="item.name"
        v-model="formData[item.name]"
        :options="item.options"
        optionLabel="label"
        ptionValue="value"
        filter
        :placeholder="item.placeholder"
        :maxSelectedLabels="item.maxSelectedLabels || 99"
      />

      <InputDateMask
        v-else-if="item.type === 'date'"
        :id="item.name"
        :name="item.name"
        v-model="formData[item.name] as Date"
      />

      <Textarea
        v-else-if="item.type === 'textarea'"
        :id="item.name"
        :name="item.name"
        v-model="formData[item.name] as string"
        :rows="item.rows || 3"
        variant="filled"
        :placeholder="item.placeholder"
      />

      <Checkbox
        v-else-if="item.type === 'checkbox'"
        :id="item.name"
        :name="item.name"
        :class="['form-item', item.layout || 'full-width']"
        v-model="formData[item.name]"
        binary
      />
    </div>

    <Button type="submit" class="half-width">{{ submitLabel }}</Button>
  </Form>
</template>

<script setup lang="ts">
import { FormItem, FormData } from "@/App.vue";
import { Form } from "@primevue/forms";
import {
  MultiSelect,
  Button,
  InputText,
  InputNumber,
  RadioButton,
  Select,
  Textarea,
  Checkbox,
} from "primevue";
import InputDateMask from "./InputDateMask.vue";

defineProps<{
  formItems: FormItem[];
  isSubmitting: boolean;
  submitLabel?: string;
}>();

const emit = defineEmits<{
  (e: "submit", formData: FormData): void;
  (e: "cancel"): void;
}>();

const formData = defineModel<FormData>();
</script>
<style scoped>
.dynamic-form {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 0.5rem;
  align-items: start;
}

.form-item {
  min-width: 0; /* 防止内容溢出 */
}

/* 通过layout直接控制 */
.full-width {
  grid-column: span 12;
}

input,
.p-inputtext,  /* PrimeVue输入框类名 */
.p-dropdown,
.p-inputnumber,
.p-textarea,
.p-multiselect,
.p-select {
  /* PrimeVue下拉框类名 */
  width: 100%;
  box-sizing: border-box; /* 包含padding和border */
}

/* 深度选择器强制覆盖 */
:deep(.date-input-container) {
  display: flex !important;
  width: 100% !important;
}

:deep(.date-input-container .p-inputtext) {
  flex: 1 !important;
  min-width: 0 !important;
  width: 100% !important;
}

:deep(.date-input-container .p-button) {
  flex: none !important;
  margin-left: 0.2rem !important;
}

.half-width {
  grid-column: span 6;
}

.third-width {
  grid-column: span 4;
}

/* 响应式适配 */
@media (max-width: 768px) {
  .dynamic-form {
    grid-template-columns: 1fr; /* 单列布局 */
  }

  .full-width,
  .half-width,
  .third-width {
    grid-column: span 1;
  }
}
</style>
