<script setup lang="ts">
import { onBeforeMount, ref, watch } from "vue";
import BaiduMap from "@/components/baiduMap.vue";
import { ElMessage } from "element-plus";
import { getDataApi } from "@/api/markApi";
import { useRoute } from "vue-router";

// 子组件实例
interface MapRef {
  addMarkOnMap: (
    url: string,
    size: number[],
    position: number[],
    offset?: number[],
  ) => void;
  getDistance: (start: number[], end: number[]) => number;
  clear: () => void;
  fit: (points: object[]) => void;
}

const mapRef2 = ref<MapRef | null>(null);

// 变量
const allSensors = ref([] as any); //存放地图上所有标记的传感器节点
const allGateways = ref([] as any); //存放地图上所有标记的网关节点
const allCrossings = ref([] as any); //存放地图上所有标记的路口节点

const route = useRoute();

async function getData() {
  try {
    const { data } = await getDataApi(route.query.dataset);
    if (data.success === true) {
      allSensors.value = data.data.sensors;
      allGateways.value = data.data.gateways;
      allCrossings.value = data.data.crossings;
      // console.log(allSensors.value)
    } else {
      ElMessage({
        type: "error",
        message: data.msg,
      });
    }
  } catch (error: any) {
    ElMessage({
      type: "error",
      message: error.message,
    });
  }
}

onBeforeMount(() => {
  getData();
});

watch(
  () => route.query.dataset,
  async (d) => {
    try {
      const { data } = await getDataApi(route.query.dataset);
      if (data.success === true) {
        allSensors.value = data.data.sensors;
        allGateways.value = data.data.gateways;
        allCrossings.value = data.data.crossings;
        showMarks();
      } else {
        ElMessage({
          type: "error",
          message: data.msg,
        });
      }
    } catch (error: any) {
      ElMessage({
        type: "error",
        message: error.message,
      });
    }
  },
);

function mapReady() {
  showMarks();
}

// 传感器图标
const SENSOR = "src/components/icons/point.png";
// 车位图标
// const PARK = 'src/components/icons/parkingspace.png'
// const SIZE2 = [32, 48]
// 网关图标
const GATEWAY = "src/components/icons/jian.png";
// 路口图标
const CROSSING = "src/components/icons/green.png";

// 子组件点击地图时触发标注事件
function showMarks() {
  // 清除图上标记
  mapRef2.value?.clear();
  // 在地图上标注这些点
  // 传感器
  for (let j = 0; j < allSensors.value.length; j++) {
    mapRef2.value?.addMarkOnMap(
      SENSOR,
      [16, 16],
      [allSensors.value[j].lng, allSensors.value[j].lat],
    );
  }
  // 网关
  for (let j = 0; j < allGateways.value.length; j++) {
    mapRef2.value?.addMarkOnMap(
      GATEWAY,
      [32, 32],
      [allGateways.value[j].lng, allGateways.value[j].lat],
      [0, -16],
    );
  }
  // 路口
  for (let j = 0; j < allCrossings.value.length; j++) {
    mapRef2.value?.addMarkOnMap(
      CROSSING,
      [32, 32],
      [allCrossings.value[j].lng, allCrossings.value[j].lat],
      [0, -16],
    );
  }
  mapRef2.value?.fit(allSensors.value);
}

// 格式化标记点
function format() {
  mapRef2.value?.clear();
}
</script>

<template>
  <baidu-map ref="mapRef2" @map-ready="mapReady"></baidu-map>
  <div class="func-area">
    车位:
    {{ allSensors.length }}个 灯杆: {{ allGateways.length }}个 路口:
    {{ allCrossings.length }}个
  </div>
</template>

<style scoped>
.func-area {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 10px;
  font-size: 30px;
  font-weight: bold;
}

.func-area > * {
  margin-right: 10px;
}

.func-area .typeS {
  display: flex;
  align-items: center;
  justify-content: center;
}

.notification {
  position: absolute;
  top: 10px; /* 调整位置 */
  left: 100px; /* 调整位置 */
  background-color: rgba(182, 182, 182, 0.8);
  padding: 5px;
  border-radius: 20px;
  font-size: 30px;
  z-index: 50;
  font-weight: bold;
}
</style>
