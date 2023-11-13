<script setup lang="ts">
import * as PDFJS from "pdfjs-dist/legacy/build/pdf.js";
import * as PdfWorker from "pdfjs-dist/build/pdf.worker.js";
import { fabric } from "fabric-with-erasing";
import { nextTick, ref, Ref, watch } from "vue";
// import { Log } from '@/utils'
import { isEmpty, debounce } from "lodash-es";

const props: any = defineProps({
  pdf: {
    type: String,
    required: true,
  },
});

window.pdfjsWorker = PdfWorker;
let pdfDoc: any = null;
const pdfPages: Ref = ref(0);
const pdfScale: Ref = ref(1.3);
const currentFarbicCanvas = ref();
const currentTool = ref("pencil");
const mouseFrom = ref({ x: 0, y: 0 });
const mouseTo = ref({ x: 0, y: 0 });
const moveCount = ref(1);
const doDrawing = ref(false);
const drawingObject = ref(null);
const drawColor = ref("#E34F51");
const drawWidth = ref(2);
const zoom = ref(1);

const loadFile = async (url: any) => {
  // 设定pdfjs的 workerSrc 参数
  PDFJS.GlobalWorkerOptions.workerSrc = PdfWorker;
  const loadingTask = PDFJS.getDocument(url);
  loadingTask.promise
    .then(async (pdf: any) => {
      pdfDoc = pdf; // 保存加载的pdf文件流
      pdfPages.value = pdfDoc.numPages; // 获取pdf文件的总页数
      await nextTick(() => {
        renderPage(1); // 将pdf文件内容渲染到canvas
      });
    })
    .catch((error: any) => {
      console.log(error);
    });
};

const renderPage = (num: any) => {
  pdfDoc.getPage(num).then((page: any) => {
    page.cleanup();
    const canvasContainer: any = document.getElementById(`pdf-canvas-${num}`)!;
    if (canvasContainer) {
      const ctx = canvasContainer.getContext("2d");
      let farbicCanvas = new fabric.Canvas(`pdf-canvas-${num}`, {
        isDrawingMode: true,
        selectable: false,
        selection: false,
        devicePixelRatio: true,
      });
      const viewport = page.getViewport({ scale: pdfScale.value });
      // console.log("viewport", viewport.width, viewport.height);
      farbicCanvas.setWidth(window.outerWidth);
      farbicCanvas.setHeight(window.outerHeight);
      farbicCanvas.isDrawingMode = true;
      farbicCanvas.freeDrawingBrush.color = "#000";
      farbicCanvas.freeDrawingBrush.width = 5;
      farbicCanvas.on({
        "mouse:down": (o: { pointer: { x: number; y: number } }) => {
          console.log("11111");
          mouseFrom.value.x = o.pointer.x;
          mouseFrom.value.y = o.pointer.y;
          currentFarbicCanvas.value = farbicCanvas;
          if (currentTool.value === "text") {
            // drawText();
          }
        },
        "mouse:move": (o: { pointer: { x: any; y: any } }) => {
          if (moveCount.value % 2 && !doDrawing.value) {
            //减少绘制频率
            return;
          }
          moveCount.value++;
          mouseTo.value.x = o.pointer.x;
          mouseTo.value.y = o.pointer.y;
          drawing();
        },
      });

      canvasContainer.width = viewport.width * 1;
      // // canvas.width = screen.availWidth;
      // // canvas.height = screen.availHeight;
      canvasContainer.height = viewport.height * 1;
      // canvasContainer.style.width = viewport.width + "px";
      canvasContainer.style.width = "100vw";
      // canvas.style.height = viewport.height + "px";
      canvasContainer.style.height = "100vh";
      ctx.setTransform(1, 0, 0, 1, 0, 0);
      const renderContext = {
        canvasContext: ctx,
        viewport: viewport,
      };
      page.render(renderContext);

      if (num < pdfPages.value) {
        renderPage(num + 1);
      }
    }
  });
};

const drawing = () => {
  if (drawingObject) {
    currentFarbicCanvas.value.remove(drawingObject);
  }
  let fabricObject: any = null;
  switch (currentTool.value) {
    case "pencil":
      console.log("pencil");
      currentFarbicCanvas.value.isDrawingMode = true;
      break;
    case "remove":
      break;
    default:
      // statements_def'
      break;
  }
  if (fabricObject) {
    nextTick(() => {
      currentFarbicCanvas.value.add(fabricObject);
      drawingObject.value = fabricObject;
    });
  }
};

// const
const debouncedLoadFile = debounce((pdf: any) => loadFile(pdf), 1000);
watch(
  () => props.pdf,
  (newValue: any) => {
    !isEmpty(newValue) && debouncedLoadFile(newValue);
  },
  {
    immediate: true,
  }
);
</script>
<template>
  <div>
    <div class="pdf-container">
      <canvas
        v-for="pageIndex in pdfPages"
        :id="`pdf-canvas-${pageIndex}`"
        :key="pageIndex"
      />
    </div>
    <div class="func-btn">
      <div>画笔</div>
    </div>
  </div>
</template>
<style>
.pdf-container {
  z-index: -1;
  margin-bottom: 20px;
  overflow-y: scroll;
  /* overflow: hidden; */
}
.pdf-container canvas {
  z-index: -1;
}
.pdf-container .upper-canvas {
  z-index: 1;
  background-color: transparent;
}
.func-btn {
  position: fixed;
  bottom: 0;
  z-index: 99;
  font-size: 80px;
  color: black;
  left: 50%;
  transform: translate(-50%, 0);
}
</style>
