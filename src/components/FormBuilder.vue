<template>
  <Form
    @submit="onFormSubmit"
    class="dynamic-form"
    :resolver
    :validate-on-input="true"
    :validate-on-blur="true"
    :validate-on-submit="true"
    :initial-values="formData"
    v-slot="$form"
  >
    <div
      v-for="item in formItems"
      :key="item.name"
      :class="['form-item', item.layout || 'full-width']"
      v-show="shouldShowItem(item, formData)"
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

      <template v-else-if="item.type === 'checkbox'">
        <div v-if="item.checkboxLabel">{{ item.checkboxLabel }}</div>
        <Checkbox
          :id="item.name"
          :name="item.name"
          :class="['form-item', item.layout || 'full-width']"
          v-model="formData[item.name]"
          binary
        />
      </template>

      <Message
        v-if="item.name in $form && $form[item.name].invalid"
        severity="error"
        size="small"
        variant="simple"
        >{{ $form[item.name]?.error?.message }}</Message
      >
    </div>

    <Button type="submit" class="half-width">{{
      submitLabel || "Submit"
    }}</Button>
    <Button
      type="reset"
      class="half-width"
      @click="handleReset"
      severity="warn"
      >{{ resetLabel || "Reset" }}</Button
    >
  </Form>
</template>

<script setup lang="ts">
import { FormItem, FormData } from "@/App.vue";
import { ref, computed, watch } from "vue";
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
  Message,
} from "primevue";
import InputDateMask from "./InputDateMask.vue";
import { zodResolver } from "@primevue/forms/resolvers/zod";
import { type ZodSchema } from "zod";
import type { Ref } from "vue";

const props = defineProps<{
  formItems: FormItem[];
  isSubmitting: boolean;
  submitLabel?: string;
  resetLabel?: string;
  formSchema?: ZodSchema;
}>();

const resolver = ref(zodResolver(props.formSchema));

const emit = defineEmits<{
  (e: "submit", formData: FormData): void;
  (e: "reset"): void;
}>();

const handleReset = () => {
  emit("reset");
};

const onFormSubmit = (e: { valid: any }) => {
  if (e.valid) {
    emit("submit", formData.value);
  }
};

const formData = defineModel<FormData>();

const shouldShowItem = (item: FormItem, data: FormData): boolean => {
  if (!item.visibleCondition) return true;

  const { sourceField, operator, referenceValue } = item.visibleCondition;
  const targetValue = data[sourceField];

  switch (operator) {
    case "===":
      return targetValue === referenceValue;
    case "!==":
      return targetValue !== referenceValue;
    case ">":
      return typeof targetValue === "number" && targetValue > referenceValue;
    case "<":
      return typeof targetValue === "number" && targetValue < referenceValue;
    case "includes":
      return (
        typeof targetValue === "string" && targetValue.includes(referenceValue)
      );
    case "not-includes":
      return (
        typeof targetValue === "string" && !targetValue.includes(referenceValue)
      );
    default:
      return true;
  }
};

const useFieldCalculation = (
  formItems: FormItem[],
  formData: Ref<FormData>
) => {
  const caculatedItems = formItems
    .filter((item) => item.calculationCondition)
    .map((item) => ({
      source: computed(() => formData.value[item.name]),
      target: item.calculationCondition!.targetField,
      operator: item.calculationCondition!.operator,
      operand: item.calculationCondition!.operand,
    }));

  caculatedItems.forEach(({ source, target, operator, operand }) => {
    watch(
      source,
      (newVal) => {
        if (newVal == null) return;

        const newValue = calculateValue(newVal, operator, operand);
        if (shouldUpdate(formData.value[target], newValue)) {
          formData.value[target] = newValue;
        }
      },
      { immediate: true }
    );
  });
};

const calculateValue = (value: any, operator: string, operand: number) => {
  let newValue: number;
  if (value instanceof Date) {
    const newDate = new Date(value);
    newDate.setDate(newDate.getDate() + operand);
    return newDate;
  }
  if (typeof value === "number") {
    switch (operator) {
      case "+":
        newValue = value + operand;
        break;
      case "-":
        newValue = value - operand;
        break;
      case "*":
        newValue = value * operand;
        break;
      case "/":
        newValue = operand !== 0 ? value / operand : value;
        break;
    }
    return newValue;
  }
  return value;
};

const shouldUpdate = (current: any, newVal: any) => {
  return current instanceof Date && newVal instanceof Date
    ? current.getTime() !== newVal.getTime()
    : current !== newVal;
};

useFieldCalculation(props.formItems, formData);
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
