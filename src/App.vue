<template>
  <button @click="toggleDrawingMode">
    {{ isDrawing ? "Stop Drawing" : "Start Drawing" }}
  </button>
  <button v-if="isDrawing" @click="toggleEditingMode">
    {{ isEditing ? "Stop Editing" : "Start Editing" }}
  </button>
  <v-stage
    :config="stageConfig"
    @mousedown="handleOnStageClick"
    @mousemove="handleOnStageMove"
    ref="stageRef"
    :style="{ cursor: isDrawing ? 'crosshair' : 'default' }"
  >
    <v-layer>
      <v-image
        :config="{
          image: image,
        }"
        :height="windowHeight"
        :width="windowWidth"
      />
    </v-layer>
  </v-stage>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from "vue";

const points = ref([]);
const cursorMousePosition = ref([0, 0]);
const isMouseOverStartPoint = ref(false);
const isFinished = ref(false);
const isDrawing = ref(false);
const isEditing = ref(false);
const floors = ref([]);
const stageRef = ref();
const image = ref(null);
const windowWidth = ref(0);
const windowHeight = ref(0);

onMounted(() => {
  // display image
  const newImage = new Image();
  newImage.src = "../src/assets/apartment.png";
  newImage.onload = () => {
    image.value = newImage;
  };

  // Get window coordinants for image
  windowHeight.value = window.innerHeight;
  windowWidth.value = window.innerWidth;
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

const toggleDrawingMode = () => {
  isDrawing.value = !isDrawing.value;
  isEditing.value = false;
  if (points.value.length > 0) {
    floors.value = [...floors.value, points.value];
  }
  points.value = [];
};

const toggleEditingMode = () => {
  isEditing.value = !isEditing.value;

  if (!isEditing.value && points.value.length > 0) {
    floors.value = [...floors.value, points.value];
  }
};
</script>
