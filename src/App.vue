<template>
  <v-stage
    :config="stageConfig"
    @mousedown="handleOnStageClick"
    @mousemove="handleOnStageMove"
    ref="stageRef"
    :style="{ cursor: isDrawing ? 'crosshair' : 'default' }"
  >
    <v-layer></v-layer>
  </v-stage>
</template>

<script setup>
import { ref, reactive, computed } from "vue";

const points = ref([]);
const cursorMousePosition = ref([0, 0]);
const isMouseOverStartPoint = ref(false);
const isFinished = ref(false);
const isDrawing = ref(false);
const isEditing = ref(false);
const floors = ref([]);
const stageRef = ref();
const image = reactive({
  imageObj: new window.Image(),
  imageUrl: "../src/assets/apartment.png",
});

const stageConfig = computed(() => ({
  width: window.innerWidth,
  height: window.innerHeight,
}));

const handleOnStageClick = (event) => {
  const mousePosition = [event.evt.offsetX, event.evt.offsetY];

  if (isEditing.value) {
    return;
  }

  if (isMouseOverStartPoint.value && points.value.length >= 3) {
    isFinished.value = true;
  } else if (isDrawing.value) {
    points.value = [...points.value, mousePosition];
  }
};

const handleOnStageMove = (event) => {
  const mousePosition = [event.evt.offsetX, event.evt.offsetY];
  cursorMousePosition.value = mousePosition;
};
</script>
