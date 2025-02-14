<script setup lang="ts">
import { computed, inject } from "vue";
import { useCameraSettingsStore } from "@/stores/settings/CameraSettingsStore";
import { useStateStore } from "@/stores/StateStore";
import loadingImage from "@/assets/images/loading.svg";
import type { StyleValue } from "vue/types/jsx";

const props = defineProps<{
  streamType: "Raw" | "Processed";
  id?: string;
}>();

const src = computed<string>(() => {
  const port =
    useCameraSettingsStore().currentCameraSettings.stream[props.streamType === "Raw" ? "inputPort" : "outputPort"];

  if (!useStateStore().backendConnected || port === 0) {
    return loadingImage;
  }

  return `http://${inject("backendHostname")}:${port}/stream.mjpg`;
});
const alt = computed<string>(() => `${props.streamType} Stream View`);

const style = computed<StyleValue>(() => {
  if (useStateStore().colorPickingMode) {
    return { cursor: "crosshair" };
  } else if (src.value !== loadingImage) {
    return { cursor: "pointer" };
  }

  return {};
});

const handleClick = () => {
  if (!useStateStore().colorPickingMode && src.value !== loadingImage) {
    window.open(src.value);
  }
};
</script>

<template>
  <img :id="id" crossorigin="anonymous" :src="src" :alt="alt" :style="style" @click="handleClick" />
</template>
