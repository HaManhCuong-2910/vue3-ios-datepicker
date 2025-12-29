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
  </div>
</template>

<script setup lang="ts">
import { computed, nextTick, onMounted, onUnmounted, ref } from "vue";
// @ts-ignore
import { customRollDate } from "../assets/js/rolldate.min.js";
import { nanoid } from "nanoid";
import type { IOptions } from "../models/types.js";

const EDatePicker = ref<HTMLInputElement | null>(null);
let rollDate = null;
let timer: NodeJS.Timeout | null = null;
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
}>();
const emits = defineEmits<{
  (e: "update:modelValue", value: string): void;
  (e: "onChange", value: string): void;
}>();

const ariaControls = computed(() => {
  return `${props.id}-${nanoid()}`;
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

const onInitElement = async () => {
  await nextTick();
  timer = setTimeout(() => {
    if (!EDatePicker.value) return;
    customRollDate(
      '[aria-controls="' + ariaControls.value + '"]',
      normalizeDateFormat(props.format),
      props.title,
      props.modelValue,
      props.options,
      props.defaultValue,
      props.confirmLabel,
      props.iconClose,
      (value: string) => {
        emits("update:modelValue", value);
        emits("onChange", value);
      }
    );
  }, 300);
};

onMounted(async () => {
  await onInitElement();
});

onUnmounted(() => {
  if (timer) clearTimeout(timer);
  rollDate = null;
});
</script>

<style scoped lang="scss"></style>
