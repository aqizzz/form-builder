<template>
  <label :for="item.name"
    >{{ item.label }} <span v-if="item.required">*</span></label
  >
  <InputText
    v-if="['text', 'email', 'password'].includes(item.type)"
    :id="item.name"
    :type="item.type"
    :name="item.name"
    v-model="formData[item.name] as string"
    @value-change="handleValueChange"
    :placeholder="item.placeholder"
  />

  <InputNumber
    v-else-if="item.type === 'number'"
    :id="item.name"
    :name="item.name"
    :type="item.type"
    v-model="formData[item.name] as number"
    @value-change="handleValueChange"
    :placeholder="item.placeholder"
  />

  <div v-else-if="item.type === 'radio'" :id="item.name">
    <label v-for="opt in item.options" :key="opt.value">
      <RadioButton
        v-model="formData[item.name]"
        @value-change="handleValueChange"
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
    @value-change="handleValueChange"
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
    @value-change="handleValueChange"
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
    @value-change="handleValueChange"
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
      @value-change="handleValueChange"
    />
  </template>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { FormItem, FormData } from "@/App.vue";
import {
  MultiSelect,
  InputText,
  InputNumber,
  RadioButton,
  Select,
  Textarea,
  Checkbox,
} from "primevue";
import InputDateMask from "./InputDateMask.vue";
import { type ZodSchema } from "zod";
import { EmitItem } from "@/App.vue";

const props = defineProps<{
  formSchema?: ZodSchema;
  item: FormItem;
}>();

const formData = defineModel<FormData>();

const emitItem = ref<EmitItem>({
  name: props.item.name,
  value: formData[props.item.name],
});

const emit = defineEmits<{
  (e: "value-change", ItemData: EmitItem): void;
}>();

const handleValueChange = (event: any) => {
  emit("value-change", {
    name: props.item.name,
    value: event.value ?? formData.value[props.item.name],
  });
};
</script>
