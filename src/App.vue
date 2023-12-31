<script setup>
import { ref, computed, onMounted } from "vue";

const points = ref([]);
const cursorMousePosition = ref([0, 0]);
const isMouseOverStartPoint = ref(false);
const isFinished = ref(false);
const isDrawing = ref(false);
const isEditing = ref(false);
const shapes = ref([]);
const stageRef = ref();
const image = ref(new Image());
const imageHeight = ref(0);
const imageWidth = ref(0);
const windowWidth = ref(0);
const windowHeight = ref(0);

onMounted(() => {
  image.value.src = "../src/assets/apartment.png";
  image.value.addEventListener("load", () => {
    correctImageDimension();
  });

  windowHeight.value = window.innerHeight;
  windowWidth.value = window.innerWidth;

  window.addEventListener("resize", () => {
    windowWidth.value = window.innerWidth;
    windowHeight.value = window.innerHeight;
  });
});

const correctImageDimension = () => {
  imageWidth.value = window.innerWidth;
  imageHeight.value = window.innerHeight;
};

const lineConfig = computed(() => ({
  stroke: "black",
  lineJoin: "round",
  strokeWidth: 5,
  closed: isFinished.value,
}));

const rectangleConfig = (point, index) => ({
  x: point[0] - 6 / 2.5,
  y: point[1] - 6 / 2.5,
  width: 6,
  height: 6,
  fill: "red",
  stroke: "red",
  strokeWidth: 3,
  draggable: isEditing.value,
  onDragMove: handleDragMovePoint,
  ...getRectangleAttributes(index),
});

const stageConfig = ref({
  width: window.innerWidth,
  height: window.innerHeight,
});

// [ [a, b], [c, d], ... ] to [ a, b, c, d, ...]
const flattenedPoints = computed(() => {
  return points.value
    .concat(isFinished.value ? [] : cursorMousePosition.value)
    .reduce((a, b) => a.concat(b), []);
});

const getRectangleAttributes = (index) =>
  index === 0
    ? {
        hitStrokeWidth: 12,
        onMouseOver: handleMouseOverStartPoint,
        onMouseOut: handleMouseOutStartPoint,
      }
    : {};

const handleMouseOverStartPoint = (event) => {
  if (isFinished.value || points.value.length < 3) return;
  event.target.scale({ x: 2, y: 2 });
  isMouseOverStartPoint.value = true;
};

const handleMouseOutStartPoint = (event) => {
  event.target.scale({ x: 1, y: 1 });
  isMouseOverStartPoint.value = false;
};

const updatePointPositionInEditMode = (index, position) => {
  points.value[index] = position;
};

const handleDragMovePoint = (event) => {
  const index = event.target.index - 1;
  const position = [event.target.attrs.x, event.target.attrs.y];
  updatePointPositionInEditMode(index, position);
};

const handleOnStageClick = (event) => {
  const mousePosition = [event.evt.offsetX, event.evt.offsetY];

  if (isEditing.value) {
    return;
  }

  if (isMouseOverStartPoint.value) {
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
  isFinished.value = false;
  if (points.value.length > 0) {
    shapes.value = [...shapes.value, points.value];
  }
  points.value = [];
};

const toggleEditingMode = () => {
  isEditing.value = !isEditing.value;
};

const updateShapeCoordinatesBasedNewImageDimensions = (shapePoints) => {
  const scaleX = windowWidth.value / imageWidth.value;
  const scaleY = windowHeight.value / imageHeight.value;

  return {
    points: shapePoints
      .map((point) => [point[0] * scaleX, point[1] * scaleY])
      .reduce((a, b) => a.concat(b), []),
    fill: "blue",
    lineJoin: "round",
    opacity: 0.5,
    closed: true,
  };
};
</script>

<template>
  <button @click="toggleDrawingMode">
    {{ isDrawing ? "Stop Drawing" : "Start Drawing" }}
  </button>
  <button v-if="isDrawing" @click="toggleEditingMode">
    {{ isEditing ? "Stop Editing" : "Start Editing" }}
  </button>

  <v-stage
    @mousedown="handleOnStageClick"
    @mousemove="handleOnStageMove"
    ref="stageRef"
    :config="stageConfig"
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
      <v-group>
        <v-line :config="lineConfig" :points="flattenedPoints" />
        <v-rect
          v-for="(point, index) in points"
          :key="index"
          :config="rectangleConfig(point, index)"
        />
        <v-line
          v-for="(shapePoints, index) in shapes"
          :key="`shape-${index}`"
          :config="updateShapeCoordinatesBasedNewImageDimensions(shapePoints)"
        />
      </v-group>
    </v-layer>
  </v-stage>
</template>
