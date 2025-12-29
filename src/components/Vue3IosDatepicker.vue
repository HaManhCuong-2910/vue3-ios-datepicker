<template>
  <div
    :class="['vue3-ios-datepicker-container v3-ios-datepicker', props.class]"
  >
    <img
      :src="props.icon"
      alt="icon date vue3-ios-datepicker"
      class="vue3-ios-datepicker-icon"
    />
    <DatePicker
      :key="key"
      :model-value="customValue"
      :placeholder="props.placeholder"
      :id="inputId"
      :name="props.name || inputId"
      :format="props.format"
      :class="props.inputClass"
      :title="props.title"
      :default-value="props.defaultValue"
      :confirm-label="props.confirmLabel"
      :options="props.options"
      @update:model-value="
        (val) => emits('update:modelValue', dayjs(val, props.format).toDate())
      "
      @on-change="(val) => emits('onChange', val)"
    />
  </div>
</template>

<script setup lang="ts">
import { useId } from "vue";
import { computed, onMounted, ref, watch } from "vue";
import DatePicker from "./DatePicker.vue";
import dayjs from "dayjs";
import type { IOptions } from "../models/types";
import CalendarIcon from "../assets/svg/calendar.svg";
import "../assets/scss/main.scss";

const props = withDefaults(
  defineProps<{
    modelValue?: Date;
    placeholder?: string;
    id?: string;
    name?: string;
    format?: string;
    iconClose?: string;
    class?: string;
    inputClass?: string;
    title?: string;
    defaultValue?: Date;
    confirmLabel?: string;
    options?: IOptions;
    icon?: string;
  }>(),
  {
    placeholder: "Select date",
    format: "YYYY-MM-DD",
    title: "Select Date",
    confirmLabel: "OK",
    class: "",
    inputClass: "",
    icon: CalendarIcon,
  }
);
const emits = defineEmits<{
  (e: "update:modelValue", value: Date): void;
  (e: "onChange", value: string): void;
}>();

const key = ref(0);
const inputId = computed(() => props.id ?? `vue3-ios-date-picker-${useId()}`);

const setIcon = () => {
  if (props.iconClose) {
    const el = document.querySelector(
      ".rolldate-container .icon-close-roll-date"
    ) as HTMLElement | null;
    if (el) {
      el.style.setProperty("--default-url-icon", `url(${props.iconClose})`);
    }
  }
};

const customValue = computed(() => {
  return props.modelValue ? dayjs(props.modelValue).format(props.format) : "";
});

watch(
  () => [
    props.id,
    props.format,
    props.title,
    props.defaultValue,
    props.confirmLabel,
    props.placeholder,
    props.name,
    props.class,
    props.inputClass,
    props.iconClose,
  ],
  () => {
    key.value++;
    setIcon();
  }
);

onMounted(() => {
  setIcon();
});
</script>

<style scoped lang="scss"></style>
