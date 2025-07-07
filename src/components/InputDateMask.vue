<template>
  <div class="date-input-container">
    <InputMask
      v-model="formattedDate"
      :mask="mask"
      :placeholder="placeholder"
      @value-change="onInputChange"
      @blur="notifyValidity"
    />

    <Button
      @click="toggle"
      icon="pi pi-calendar"
      severity="secondary"
      outlined
    />

    <Popover ref="op">
      <DatePicker v-model="selectedDate" inline @date-select="onDateSelect" />
    </Popover>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import Popover from "primevue/popover";
import Button from "primevue/button";
import DatePicker from "primevue/datepicker";
import InputMask from "primevue/inputmask";
import { parse, format, isValid } from "date-fns";

const props = defineProps({
  mask: {
    type: String,
    default: "99/99/9999",
  },
  pattern: {
    type: String,
    default: "MM/dd/yyyy",
  },
  placeholder: {
    type: String,
    default: "MM/DD/YYYY",
  },
  dateFormat: {
    type: String,
    default: "yyyy-MM-dd",
  },
});

const inputModel = defineModel<Date | null>({ required: true });
const formattedDate = ref<string>("");
const selectedDate = ref<Date | null>(null);
const inputValue = ref<string>("");
const isDateValid = ref<boolean>(true);
const dateString = ref<string>("");

const emit = defineEmits<{
  (
    e: "valueChange",
    value: Date | null,
    formattedString: string,
    isValid: boolean,
    dateString: string
  ): void;
}>();

const op = ref();

const toggle = (event: MouseEvent) => {
  op.value.toggle(event);
};

// 从父组件传入日期或日历转string
const formatDateToString = (date: Date | null): string => {
  if (!date || !isValid(date)) return "";

  return format(date, props.pattern);
};

// 从输入string值转日历
const parseStringToDate = (dateString: string): Date | null => {
  if (!dateString) {
    return null;
  }
  try {
    const parsedDate = parse(dateString, props.pattern, new Date());

    if (!isValid(parsedDate)) {
      return null;
    }

    return parsedDate;
  } catch (e) {
    console.error("Parse error:", e);
    return null;
  }
};

const onDateSelect = (newDate: Date) => {
  formattedDate.value = formatDateToString(newDate);
  inputModel.value = newDate;
  dateString.value = format(newDate, props.dateFormat);
  emit("valueChange", newDate, formattedDate.value, true, dateString.value);
  op.value.hide();
};

const onInputChange = (newVal: string) => {
  const newDate = parseStringToDate(newVal);
  inputValue.value = newVal;
  if (!newDate) {
    selectedDate.value = new Date();
    isDateValid.value = false;
    inputModel.value = null;
    dateString.value = "";
  } else {
    selectedDate.value = newDate;
    inputModel.value = newDate;
    isDateValid.value = true;
    dateString.value = format(newDate, props.dateFormat);
  }
  notifyValidity();
};

const notifyValidity = () => {
  emit(
    "valueChange",
    selectedDate.value,
    inputValue.value,
    isDateValid.value,
    dateString.value
  );
};

onMounted(() => {
  selectedDate.value = inputModel.value ?? new Date();
  formattedDate.value = formatDateToString(inputModel.value);
});
</script>
