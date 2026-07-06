<template>
  <div>
    <input
      ref="EDatePicker"
      autocorrect="off"
      autocapitalize="off"
      spellcheck="false"
      type="text"
      readonly
      :name="props.name || props.id"
      :value="props.modelValue"
      :placeholder="props.placeholder"
      :class="['vue3-ios-datepicker-input', props.class]"
      :id="props.id"
      :aria-controls="ariaControls"
    />

    <IosDatePickerModal
      :is-open="isOpen"
      :model-value="props.modelValue"
      :format="normalizedFormat"
      :title="props.title"
      :default-value="props.defaultValue"
      :confirm-label="props.confirmLabel"
      :icon-close="props.iconClose"
      :disabled-date="props.disabledDate"
      :lang="props.lang"
      :lang-object-custom="props.langObjectCustom"
      :options="props.options"
      @confirm="handleConfirm"
      @cancel="handleCancel"
    />
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from "vue";
import { nanoid } from "nanoid";
import type { IOptions } from "../models/types.js";
import IosDatePickerModal from "./IosDatePickerModal.vue";

const EDatePicker = ref<HTMLInputElement | null>(null);
const isOpen = ref(false);
let containerElement: HTMLElement | null = null;

const props = defineProps<{
  modelValue: string;
  placeholder: string;
  id: string;
  name: string;
  class: string;
  format: string;
  title: string;
  iconClose?: string;
  defaultValue?: Date;
  confirmLabel: string;
  options?: IOptions;
  lang: string;
  langObjectCustom?: object;
  disabledDate?: (date: Date) => boolean;
}>();

const emits = defineEmits<{
  (e: "update:modelValue", value: string): void;
  (e: "onChange", value: string): void;
}>();

const stableId = nanoid();
const ariaControls = computed(() => {
  return `${props.id}-${stableId}`;
});

const normalizeDateFormat = (format: string) => {
  const map: any = {
    yyyy: "YYYY",
    yy: "YY",
    dd: "DD",
    d: "D",
    hh: "HH",
    h: "H",
    mm: "mm", // minute
    m: "M", // fallback
    MM: "MM", // month
  };

  return format.replace(/yyyy|yy|dd|d|HH|H|mm|MM|m/gi, (token) => {
    const lower = token.toLowerCase();

    if (lower === "mm" && token === "MM") return "MM"; // month
    if (lower === "mm") return "mm"; // minute

    return map[lower] || token;
  });
};

const normalizedFormat = computed(() => {
  return normalizeDateFormat(props.format);
});

const openModal = (e: Event) => {
  e.preventDefault();
  e.stopPropagation();
  isOpen.value = true;
};

const handleConfirm = (value: string) => {
  emits("update:modelValue", value);
  emits("onChange", value);
  isOpen.value = false;
};

const handleCancel = () => {
  isOpen.value = false;
};

onMounted(() => {
  containerElement = EDatePicker.value?.closest(".vue3-ios-datepicker-container") as HTMLElement || null;
  if (containerElement) {
    containerElement.addEventListener("click", openModal);
  }
});

onUnmounted(() => {
  if (containerElement) {
    containerElement.removeEventListener("click", openModal);
  }
});
</script>

<style scoped lang="scss"></style>
