<script setup lang="ts">
import { ref } from "vue";
import FormBuilder from "@/components/FormBuilder.vue";
import Card from "primevue/card";
import { z } from "zod";

// 定义表单字段类型
export interface FormItem {
  name: string;
  type: string;
  label: string;
  layout?: string;
  placeholder?: string;
  required?: boolean;
  options?: Array<{ label: string; value: string | number }>;
  checkboxLabel?: string;
  maxSelectedLabels?: number; // For multiselection
  rows?: number; //For textarea
  hide?: boolean;
}

// 定义表单数据类型
export interface FormData {
  [key: string]: string | number | boolean | Date | null | string[];
  name: string;
  email: string;
  age: number | null;
  password?: string;
  gender?: string;
  state?: string;
  city?: string;
  region?: string;
  hobbies?: string[];
  birthday?: Date | null;
  bio?: string;
  agree?: boolean;
}

export interface EmitItem {
  name: string;
  value: any;
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
  },
  {
    name: "password",
    type: "password",
    label: "密码",
    placeholder: "请输入密码",
    layout: "half-width",
    required: true,
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
    name: "state",
    type: "select",
    label: "省份",
    placeholder: "请选择您所在的省份",
    layout: "third-width",
    required: true,
    options: [
      { label: "北京", value: "beijing" },
      { label: "上海", value: "shanghai" },
      { label: "广东", value: "guangdong" },
      { label: "湖北", value: "hubei" },
    ],
  },
  {
    name: "city",
    type: "select",
    label: "城市",
    placeholder: "请选择您所在的城市",
    layout: "third-width",
    required: true,
    options: [],
  },
  {
    name: "region",
    type: "select",
    label: "区域",
    placeholder: "请选择您所在的区域",
    layout: "third-width",
    required: true,
    options: [],
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
    name: "celebration",
    type: "date",
    label: "庆祝日",
    layout: "half-width",
    placeholder: "请选择您的庆祝日",
    required: false,
  },
  {
    name: "bio",
    type: "textarea",
    label: "个人简介",
    placeholder: "请简单介绍一下自己...",
    layout: "half-width",
    required: false,
  },
  {
    name: "agree",
    type: "checkbox",
    label: "同意条款",
    checkboxLabel: "我已阅读并同意用户协议和隐私政策",
    required: true,
  },
]);

const cityOfGuangdong = [
  { label: "广州", value: "guangzhou" },
  { label: "深圳", value: "shenzhen" },
  { label: "珠海", value: "zhuhai" },
  { label: "佛山", value: "foshan" },
  { label: "东莞", value: "dongguan" },
];

const regionOfGuangzhou = [
  { label: "天河区", value: "tianhe" },
  { label: "越秀区", value: "yuexiu" },
  { label: "海珠区", value: "haizhu" },
  { label: "荔湾区", value: "liwan" },
];

const cityOfShanghai = [{ label: "上海", value: "shanghai" }];

const getInitialFormData = (): FormData => ({
  name: "",
  email: "",
  age: 0,
  password: "",
  gender: "",
  state: "",
  city: "",
  region: "",
  hobbies: [],
  birthday: null,
  celebration: null,
  bio: "",
  agree: false,
});

// 表单数据
const formData = ref<FormData>(getInitialFormData());

// 提交状态
const isSubmitting = ref(false);

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
const handleReset = () => {
  if (confirm("确定要重置吗？未保存的数据将丢失。")) {
    formData.value = getInitialFormData();
  }
};

const handleValueChange = (emitItem: EmitItem) => {
  if (emitItem.name === "city") {
    formItems.value = formItems.value.map((item) => {
      if (item.name === "hobbies") {
        return {
          ...item,
          hide: emitItem.value === "shanghai",
        };
      }
      return item;
    });
  }

  if (emitItem.name === "state") {
    formItems.value = formItems.value.map((item) => {
      if (item.name === "city") {
        if (emitItem.value === "guangdong") {
          return {
            ...item,
            options: cityOfGuangdong,
          };
        }
        if (emitItem.value === "shanghai") {
          return {
            ...item,
            options: cityOfShanghai,
          };
        }
      }
      return item;
    });
  }

  if (emitItem.name === "city") {
    formItems.value = formItems.value.map((item) => {
      if (item.name === "region") {
        if (emitItem.value === "guangzhou") {
          return {
            ...item,
            options: regionOfGuangzhou,
          };
        }
      }
      return item;
    });
  }
};

const formSchema = z.object({
  name: z
    .string({ required_error: "姓名不能为空" })
    .min(2, { message: "姓名必须大于两个字符" }),
  email: z
    .string()
    .refine(
      (val) => /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(val),
      "请输入有效的邮箱（示例：user@example.com）"
    ),
  age: z.number().min(18),
});
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
          resetLabel="重置信息"
          :formSchema="formSchema"
          @submit="handleSubmit"
          @reset="handleReset"
          @value-change="handleValueChange"
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
