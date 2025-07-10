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
      <FormField :item="item" v-model="formData" />
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
import { Button, Message } from "primevue";
import FormField from "./FormField.vue";
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
