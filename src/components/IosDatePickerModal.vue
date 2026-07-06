<template>
  <Teleport to="body">
    <Transition name="v3-fade" @after-enter="onAfterEnter">
      <div
        v-if="props.isOpen"
        class="rolldate-container"
        :style="{ zIndex: containerZIndex }"
        @click="handleCancel"
      >
        <div class="rolldate-mask"></div>
        <Transition name="v3-slide-up">
          <div v-if="props.isOpen" class="rolldate-panel" @click.stop>
            <header>
              <span
                class="rolldate-cancel icon-close-roll-date"
                :style="
                  props.iconClose
                    ? { '--default-url-icon': `url(${props.iconClose})` }
                    : {}
                "
                @click="handleCancel"
              ></span>
              <span class="rolldate-title">{{ props.title }}</span>
            </header>
            <section class="rolldate-content">
              <div class="rolldate-dim mask-top"></div>
              <div class="rolldate-dim mask-bottom"></div>
              <div class="rolldate-wrapper">
                <div
                  v-for="col in columns"
                  :key="col"
                  :id="`rolldate-${col}`"
                  class="wheel-col"
                >
                  <ul
                    :ref="
                      (el) => {
                        if (el) scrollRefs[col] = el as HTMLElement;
                      }
                    "
                    class="wheel-scroll"
                    :style="{
                      scrollSnapType: isReady ? 'y mandatory' : 'none',
                    }"
                    @scroll="(e) => handleScroll(e, col)"
                    @touchstart="activeTouches[col] = true"
                    @touchend="activeTouches[col] = false"
                    @touchcancel="activeTouches[col] = false"
                  >
                    <li
                      v-for="(item, idx) in getItems(col)"
                      :key="item.value"
                      :class="[
                        'wheel-item',
                        `wheel-item-${col.toLowerCase()}`,
                        { active: selectedValue[col] === item.value },
                      ]"
                      :data-value="item.value"
                    >
                      {{ item.label }}
                    </li>
                  </ul>
                </div>
              </div>
            </section>
            <span
              :class="[
                'rolldate-btn',
                'rolldate-confirm',
                { disabled: !isValid },
              ]"
              @click="handleConfirm"
            >
              {{ props.confirmLabel }}
            </span>
          </div>
        </Transition>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup lang="ts">
import { computed, nextTick, ref, watch, onMounted } from "vue";
import dayjs from "dayjs";
import customParseFormat from "dayjs/plugin/customParseFormat";
dayjs.extend(customParseFormat);
import type { IOptions } from "../models/types";
import enMonth from "../assets/lang/month/en.json";
import viMonth from "../assets/lang/month/vi.json";

const props = defineProps<{
  isOpen: boolean;
  modelValue: string;
  format: string;
  title: string;
  defaultValue?: Date;
  confirmLabel: string;
  iconClose?: string;
  disabledDate?: (date: Date) => boolean;
  lang: string;
  langObjectCustom?: object;
  options?: IOptions;
}>();

const emits = defineEmits<{
  (e: "confirm", value: string): void;
  (e: "cancel"): void;
}>();

// Map format tokens to standard columns
const COLUMNS_MAP: Record<string, string> = {
  YYYY: "YYYY",
  MM: "MM",
  DD: "DD",
  HH: "hh",
  hh: "hh",
  mm: "mm",
  ss: "ss",
};

const columns = computed(() => {
  const matches = props.format.match(/YYYY|MM|DD|HH|hh|mm|ss/g) || [];
  // Ensure unique columns in correct order, mapped to internal keys
  const list: string[] = [];
  matches.forEach((token) => {
    const key = COLUMNS_MAP[token];
    if (key && !list.includes(key)) {
      list.push(key);
    }
  });
  return list;
});

const selectedValue = ref<Record<string, string>>({
  YYYY: "",
  MM: "",
  DD: "",
  hh: "",
  mm: "",
  ss: "",
});

const activeTouches = ref<Record<string, boolean>>({
  YYYY: false,
  MM: false,
  DD: false,
  hh: false,
  mm: false,
  ss: false,
});

const scrollRefs = ref<Record<string, HTMLElement>>({});
const isReady = ref(false);
const containerZIndex = ref(1000);
const beginYear = ref(2000);
const endYear = ref(2100);

const getMonthLabel = (m: number) => {
  if (props.langObjectCustom && (props.langObjectCustom as any)[m]) {
    return (props.langObjectCustom as any)[m];
  }
  const lang = props.lang || "en";
  if (lang === "vi") {
    return (viMonth as any)[m] || `Tháng ${m}`;
  }
  return (enMonth as any)[m] || m.toString();
};

const getDaysInMonth = (year: number, month: number) => {
  return new Date(year, month, 0).getDate();
};

const getItems = (col: string) => {
  if (col === "YYYY") {
    const list = [];
    for (let y = beginYear.value; y <= endYear.value; y++) {
      list.push({ value: String(y), label: String(y) });
    }
    return list;
  }
  if (col === "MM") {
    const list = [];
    for (let m = 1; m <= 12; m++) {
      const val = m < 10 ? `0${m}` : String(m);
      list.push({ value: val, label: getMonthLabel(m) });
    }
    return list;
  }
  if (col === "DD") {
    const list = [];
    const year = Number(
      selectedValue.value["YYYY"] || new Date().getFullYear(),
    );
    const month = Number(selectedValue.value["MM"] || 1);
    const daysInMonth = getDaysInMonth(year, month);
    for (let d = 1; d <= daysInMonth; d++) {
      const val = d < 10 ? `0${d}` : String(d);
      list.push({ value: val, label: val });
    }
    return list;
  }
  if (col === "hh") {
    const list = [];
    for (let h = 0; h < 24; h++) {
      const val = h < 10 ? `0${h}` : String(h);
      list.push({ value: val, label: val });
    }
    return list;
  }
  if (col === "mm") {
    const list = [];
    for (let m = 0; m < 60; m++) {
      const val = m < 10 ? `0${m}` : String(m);
      list.push({ value: val, label: val });
    }
    return list;
  }
  if (col === "ss") {
    const list = [];
    for (let s = 0; s < 60; s++) {
      const val = s < 10 ? `0${s}` : String(s);
      list.push({ value: val, label: val });
    }
    return list;
  }
  return [];
};

const initYears = () => {
  const dateVal = props.modelValue || props.defaultValue || new Date();
  let baseDate = new Date();
  if (dateVal instanceof Date) {
    baseDate = dateVal;
  } else if (typeof dateVal === "string" && dateVal) {
    const parsed = dayjs(dateVal, props.format);
    if (parsed.isValid()) {
      baseDate = parsed.toDate();
    }
  }
  const year = baseDate.getFullYear();
  beginYear.value = year - 100;
  endYear.value = year + 100;
};

const calculateMaxZIndex = () => {
  if (typeof document === "undefined") return;
  const elements = document.getElementsByTagName("*");
  let maxZ = 1000;
  for (let i = 0; i < elements.length; i++) {
    const el = elements[i] as HTMLElement;
    if (el.classList.contains("rolldate-container")) continue;
    const style = window.getComputedStyle(el);
    const zIndex = style.zIndex;
    if (zIndex && zIndex !== "auto") {
      const z = Number(zIndex);
      if (!isNaN(z) && z > maxZ) {
        maxZ = z;
      }
    }
  }
  containerZIndex.value = maxZ + 1;
};

const parseInitialValue = () => {
  let baseDate = new Date();
  const dateVal = props.modelValue || props.defaultValue;
  if (dateVal instanceof Date) {
    baseDate = dateVal;
  } else if (typeof dateVal === "string" && dateVal) {
    const parsed = dayjs(dateVal, props.format);
    if (parsed.isValid()) {
      baseDate = parsed.toDate();
    }
  }
  selectedValue.value = {
    YYYY: String(baseDate.getFullYear()),
    MM: dayjs(baseDate).format("MM"),
    DD: dayjs(baseDate).format("DD"),
    hh: dayjs(baseDate).format("HH"),
    mm: dayjs(baseDate).format("mm"),
    ss: dayjs(baseDate).format("ss"),
  };
};

const getSelectedDate = () => {
  const now = new Date();
  const year = selectedValue.value["YYYY"]
    ? Number(selectedValue.value["YYYY"])
    : now.getFullYear();
  const month = selectedValue.value["MM"]
    ? Number(selectedValue.value["MM"]) - 1
    : now.getMonth();
  const day = selectedValue.value["DD"]
    ? Number(selectedValue.value["DD"])
    : now.getDate();
  const hour = selectedValue.value["hh"]
    ? Number(selectedValue.value["hh"])
    : 0;
  const minute = selectedValue.value["mm"]
    ? Number(selectedValue.value["mm"])
    : 0;
  const second = selectedValue.value["ss"]
    ? Number(selectedValue.value["ss"])
    : 0;

  return new Date(year, month, day, hour, minute, second);
};

const isValid = computed(() => {
  const date = getSelectedDate();

  if (props.options?.minDate) {
    if (date.getTime() < props.options.minDate.getTime()) {
      return false;
    }
  }
  if (props.options?.maxDate) {
    if (date.getTime() > props.options.maxDate.getTime()) {
      return false;
    }
  }
  if (props.disabledDate) {
    if (props.disabledDate(date)) {
      return false;
    }
  }
  return true;
});

const handleScroll = (event: Event, type: string) => {
  const el = event.target as HTMLElement;

  if (!isReady.value) return;

  const index = Math.round(el.scrollTop / 36);
  const items = getItems(type);
  const maxIndex = items.length - 1;
  const clampedIndex = Math.max(0, Math.min(index, maxIndex));

  const val = items[clampedIndex]?.value;
  if (val && selectedValue.value[type] !== val) {
    selectedValue.value[type] = val;
  }
};

// Removed getItemStyle to optimize scroll performance

const syncScrollPositions = (behavior: ScrollBehavior = "auto") => {
  columns.value.forEach((col) => {
    // If the user is actively touching/dragging this column, do not programmatically scroll it.
    if (activeTouches.value[col]) return;

    const el = scrollRefs.value[col];
    if (!el) return;
    const items = getItems(col);
    const val = selectedValue.value[col];
    const index = items.findIndex((item) => item.value === val);
    if (index !== -1) {
      const targetScrollTop = index * 36;
      const currentScrollTop = el.scrollTop;
      // Scroll if the scroll position is not within snapping tolerance (18px)
      if (Math.abs(currentScrollTop - targetScrollTop) >= 18) {
        el.scrollTo({ top: targetScrollTop, behavior });
      }
    }
  });
};

watch(
  () => props.isOpen,
  async (newVal) => {
    if (newVal) {
      calculateMaxZIndex();
      isReady.value = false;
      parseInitialValue();
      initYears();
      await nextTick();
      syncScrollPositions("auto");

      // Fallback in case transition hooks are missed by Vue
      setTimeout(() => {
        if (!isReady.value) {
          isReady.value = true;
          syncScrollPositions("auto");
        }
      }, 350);
    }
  },
);

watch(
  () => ({ ...selectedValue.value }),
  (newVal, oldVal) => {
    let dayClamped = false;
    if (newVal.YYYY !== oldVal.YYYY || newVal.MM !== oldVal.MM) {
      const year = Number(newVal.YYYY || new Date().getFullYear());
      const month = Number(newVal.MM || 1);
      const maxDay = getDaysInMonth(year, month);
      const currentDay = Number(newVal.DD || 1);
      if (currentDay > maxDay) {
        selectedValue.value.DD = maxDay < 10 ? `0${maxDay}` : String(maxDay);
        dayClamped = true;
      }
    }

    nextTick(() => {
      syncScrollPositions(dayClamped ? "smooth" : "auto");
    });
  },
  { deep: true },
);

watch(
  () => selectedValue.value.YYYY,
  (newYearVal) => {
    if (!newYearVal || !isReady.value) return;
    const year = Number(newYearVal);

    // Expand top boundary, shrink bottom boundary
    if (year <= beginYear.value + 5) {
      beginYear.value -= 50;
      endYear.value -= 50;
      nextTick(() => {
        const el = scrollRefs.value["YYYY"];
        if (el) {
          const wasReady = isReady.value;
          isReady.value = false;
          el.scrollTop += 50 * 36;
          nextTick(() => {
            isReady.value = wasReady;
          });
        }
      });
    }

    // Expand bottom boundary, shrink top boundary
    if (year >= endYear.value - 5) {
      beginYear.value += 50;
      endYear.value += 50;
      nextTick(() => {
        const el = scrollRefs.value["YYYY"];
        if (el) {
          const wasReady = isReady.value;
          isReady.value = false;
          el.scrollTop -= 50 * 36;
          nextTick(() => {
            isReady.value = wasReady;
          });
        }
      });
    }
  },
);

onMounted(() => {
  if (props.isOpen) {
    calculateMaxZIndex();
    isReady.value = false;
    parseInitialValue();
    initYears();
    nextTick(() => {
      syncScrollPositions("auto");

      // Fallback in case transition hooks are missed by Vue
      setTimeout(() => {
        if (!isReady.value) {
          isReady.value = true;
          syncScrollPositions("auto");
        }
      }, 350);
    });
  }
});

const onAfterEnter = () => {
  isReady.value = true;
  // Sync positions again just in case the scroll heights changed after animation finished
  syncScrollPositions("auto");
};

const handleCancel = () => {
  emits("cancel");
};

const handleConfirm = () => {
  if (!isValid.value) return;
  const formatted = dayjs(getSelectedDate()).format(props.format);
  emits("confirm", formatted);
};
</script>

<style scoped lang="scss">
.rolldate-container {
  position: fixed;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
  z-index: 999999;
  display: flex;
  align-items: flex-end;

  .rolldate-mask {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    background-color: rgba(37, 38, 45, 0.4);
    z-index: 1001;
  }

  .rolldate-panel {
    position: absolute;
    border-top-left-radius: 12px;
    border-top-right-radius: 12px;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 340px;
    z-index: 1002;
    background: #fff;
    box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.1);
    display: flex;
    flex-direction: column;
  }

  header {
    position: relative;
    height: 50px;
    border-bottom: 1px solid #e0e0e0;
    display: flex;
    align-items: center;
    justify-content: center;

    .rolldate-title {
      font-size: 15px;
      font-weight: 500;
      color: #333;
    }

    .rolldate-cancel.icon-close-roll-date {
      --default-url-icon: url("../assets/svg/close.svg");
      position: absolute;
      display: block;
      top: 17px;
      right: 16px;
      background-image: var(--default-url-icon);
      background-repeat: no-repeat;
      background-position: center;
      z-index: 5;
      width: 16px;
      height: 16px;
      cursor: pointer;
    }
  }

  .rolldate-content {
    position: relative;
    height: 173px;
    margin-top: 20px;
    overflow: hidden;
  }

  .rolldate-dim {
    position: absolute;
    left: 0;
    width: 100%;
    height: 68px;
    pointer-events: none;
    z-index: 10;
    transform: translateZ(0);

    &.mask-top {
      top: 0;
      border-bottom: 1px solid #ebebeb;
      background: linear-gradient(
        180deg,
        rgba(255, 255, 255, 0.9) 0%,
        rgba(255, 255, 255, 0.4) 100%
      );
    }

    &.mask-bottom {
      bottom: 0;
      border-top: 1px solid #ebebeb;
      background: linear-gradient(
        0deg,
        rgba(255, 255, 255, 0.9) 0%,
        rgba(255, 255, 255, 0.4) 100%
      );
    }
  }

  .rolldate-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
    padding: 0 10px;
    perspective: 1000px;
  }

  .wheel-col {
    flex: 1;
    height: 100%;
    overflow: hidden;
    position: relative;
    transform-style: preserve-3d;
  }

  .wheel-scroll {
    height: 173px;
    overflow-y: scroll;
    scroll-snap-type: y mandatory;
    padding: 68px 0;
    margin: 0;
    box-sizing: border-box;
    scrollbar-width: none;
    -ms-overflow-style: none;
    transform-style: preserve-3d;

    &::-webkit-scrollbar {
      display: none;
    }
  }

  .wheel-item {
    height: 36px;
    line-height: 36px;
    text-align: center;
    font-size: 18px;
    color: #333;
    scroll-snap-align: center;
    list-style: none;
    backface-visibility: hidden;
    transform-style: preserve-3d;
    font-weight: 500;

    &.active {
      color: #217b56;
      font-weight: 500;
    }
  }

  .rolldate-btn.rolldate-confirm {
    display: block;
    background-color: #217b56;
    color: #fff;
    text-align: center;
    padding: 12px 0px;
    margin: 25px 16px 0px 16px;
    border-radius: 6px;
    font-size: 16px;
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.2s ease;
    user-select: none;

    &.disabled {
      background-color: #bababa;
      color: #fff;
      pointer-events: none;
      cursor: not-allowed;
    }
  }
}

/* Transitions */
.v3-fade-enter-active,
.v3-fade-leave-active {
  transition: opacity 0.3s ease;

  .rolldate-panel {
    transition: transform 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  }
}

.v3-fade-enter-from,
.v3-fade-leave-to {
  opacity: 0;

  .rolldate-panel {
    transform: translateY(100%);
  }
}

.v3-slide-up-enter-active,
.v3-slide-up-leave-active {
  transition: transform 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

.v3-slide-up-enter-from,
.v3-slide-up-leave-to {
  transform: translateY(100%);
}
</style>
