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
      v-show="!item.hide"
    >
      <FormField
        :item="item"
        v-model="formData"
        @value-change="handleValueChange"
      />
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
import { EmitItem } from "@/App.vue";

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
  (e: "value-change", ItemData: EmitItem): void;
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

const handleValueChange = (emitItem: EmitItem) => {
  emit("value-change", emitItem);
};
</script>
