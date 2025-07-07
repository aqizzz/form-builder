<script setup lang="ts">
import { ref } from "vue";
import FormBuilder from "@/components/FormBuilder.vue";
import Card from "primevue/card";

// 定义表单字段类型
export interface FormItem {
  name: string;
  type: string;
  label: string;
  layout?: string;
  placeholder?: string;
  required?: boolean;
  validation?: {
    min?: number;
    max?: number;
  };
  options?: Array<{ label: string; value: string | number }>;
  checkboxLabel?: string;
  maxSelectedLabels?: number;
  rows?: number;
}

// 定义表单数据类型
export interface FormData {
  [key: string]: string | number | boolean | Date | null | string[];
  name: string;
  email: string;
  age: number | null;
  password?: string;
  gender?: string;
  city?: string;
  hobbies?: string[];
  birthday?: Date | null;
  bio?: string;
  agree?: boolean;
}

// 表单配置数据
const formItems = ref<FormItem[]>([
  {
    name: "name",
    type: "text",
    label: "姓名",
    layout: "full-width",
    placeholder: "请输入您的姓名",
    required: true,
    validation: {
      min: 2,
      max: 50,
    },
  },
  {
    name: "email",
    type: "email",
    label: "邮箱",
    layout: "full-width",
    placeholder: "请输入您的邮箱地址",
    required: true,
  },
  {
    name: "age",
    type: "number",
    label: "年龄",
    placeholder: "请输入您的年龄",
    layout: "half-width",
    required: true,
    validation: {
      min: 1,
      max: 120,
    },
  },
  {
    name: "password",
    type: "password",
    label: "密码",
    placeholder: "请输入密码",
    layout: "half-width",
    required: true,
    validation: {
      min: 6,
    },
  },
  {
    name: "gender",
    type: "radio",
    label: "性别",
    required: true,
    options: [
      { label: "男", value: "male" },
      { label: "女", value: "female" },
      { label: "其他", value: "other" },
    ],
  },
  {
    name: "city",
    type: "select",
    label: "城市",
    placeholder: "请选择您所在的城市",
    layout: "third-width",
    required: true,
    options: [
      { label: "北京", value: "beijing" },
      { label: "上海", value: "shanghai" },
      { label: "广州", value: "guangzhou" },
      { label: "深圳", value: "shenzhen" },
      { label: "杭州", value: "hangzhou" },
    ],
  },
  {
    name: "hobbies",
    type: "multiselect",
    label: "兴趣爱好",
    placeholder: "请选择您的兴趣爱好",
    required: false,
    layout: "third-width",
    rows: 5,
    options: [
      { label: "阅读", value: "reading" },
      { label: "运动", value: "sports" },
      { label: "音乐", value: "music" },
      { label: "旅行", value: "travel" },
      { label: "编程", value: "coding" },
      { label: "摄影", value: "photography" },
    ],
  },
  {
    name: "birthday",
    type: "date",
    label: "生日",
    layout: "third-width",
    placeholder: "请选择您的生日",
    required: false,
  },
  {
    name: "bio",
    type: "textarea",
    label: "个人简介",
    placeholder: "请简单介绍一下自己...",
    required: false,
    validation: {
      max: 500,
    },
  },
  {
    name: "agree",
    type: "checkbox",
    label: "同意条款",
    checkboxLabel: "我已阅读并同意用户协议和隐私政策",
    required: true,
  },
]);

// 表单数据
const formData = ref<FormData>({
  name: "",
  email: "",
  age: null,
  password: "",
  gender: "",
  city: "",
  hobbies: [],
  birthday: null,
  bio: "",
  agree: false,
});

// 提交状态
const isSubmitting = ref(false);

const getInitialFormData = (): FormData => ({
  name: "",
  email: "",
  age: null,
  password: "",
  gender: "",
  city: "",
  hobbies: [],
  birthday: null,
  bio: "",
  agree: false,
});

// 处理表单提交
const handleSubmit = async (data: FormData) => {
  isSubmitting.value = true;

  try {
    // 模拟API调用
    await new Promise((resolve) => setTimeout(resolve, 2000));

    console.log("提交的表单数据:", data);
    alert("表单提交成功！");
  } catch (error) {
    console.error("提交失败:", error);
    alert("提交失败，请重试");
  } finally {
    isSubmitting.value = false;
  }
};

// 处理取消
const handleCancel = () => {
  if (confirm("确定要取消吗？未保存的数据将丢失。")) {
    formData.value = getInitialFormData();
  }
};
</script>

<template>
  <div class="w-full flex flex-col items-stretch">
    <Card class="flex-1">
      <template #title>
        <div>
          <i class="pi pi-user"></i>
          用户信息表单
        </div>
      </template>

      <template #content>
        <FormBuilder
          :formItems="formItems"
          v-model="formData"
          :isSubmitting="isSubmitting"
          submitLabel="保存信息"
          @submit="handleSubmit"
          @cancel="handleCancel"
        />
      </template>
    </Card>

    <!-- 调试信息 -->
    <Card
      v-if="Object.keys(formData).length > 0"
      class="mt-4 w-full flex flex-col"
    >
      <template #title>
        <div>
          <i class="pi pi-code"></i>
          当前表单数据 (调试信息)
        </div>
      </template>

      <template #content>
        <div class="px-4">
          <pre class="w-full">{{ JSON.stringify(formData, null, 2) }}</pre>
        </div>
      </template>
    </Card>
  </div>
</template>
