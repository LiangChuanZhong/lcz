---
title: vue3中以高德地图为基础实现可视化仓库网络
date: 2024-02-27 21:48:26
tags: [Vue]
categories: Vue
---

vue3 中以高德地图为基础实现可视化仓库网络，实现仓库在地图上标记定位，自定义信息窗体，非单个区域框选（单个或多个区域），带 1 个孔区域框选，带多个孔区域框选，选中标记点呼吸动效

<!-- more -->

## 实现效果：

![img](./assets/img/map/2.png)
![img](./assets/img/map/3.png)
![img](./assets/img/map/4.png)

高德开发平台 : [高德开放平台 | 高德地图 API (amap.com)](https://console.amap.com/dev/key/app)

## 使用：

1. 首先你要注册好账号登录,并在应用管理中创建应用，然后得到 key 和安全密钥
   ![img](./assets/img/map/0.png)
2. 按 NPM 方式安装使用 Loader :

```JS
npm i @amap/amap-jsapi-loader --save
```

3. 引入使用：

```JS
<div id="container"></div>
const initMap = async () => {
  window._AMapSecurityConfig = {
    securityJsCode: "7788cd228050738a8d7dcb1978b2fb19", // 安全密钥
  };
  await AMapLoader.load({
    key: "700ae1a3c31530a5bfa73b5aa8b0b862", // 申请好的Web端开发者Key，首次调用 load 时必填
    version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
    plugins: [""], // 需要使用的的插件列表，如比例尺'AMap.Scale'等
  });
  map.value = new AMap.Map("container", {
    //设置地图容器id
    viewMode: "3D", //是否为3D地图模式
    zoom: 5, //初始化地图级别
    center: centerPoint, //初始化地图中心点位置
  });

  // 自定义 官方默认自定义样式
  var styleName = "amap://styles/normal";
  map.value.setMapStyle(styleName);

  // 添加插件
  map.value.plugin(
    [
      "AMap.ToolBar",
      "AMap.Scale",
      "AMap.HawkEye",
      "AMap.Geolocation",
      "AMap.MapType",
      "AMap.MouseTool",
    ],
    function () {
      //异步同时加载多个插件
      // 添加地图插件
      map.value.addControl(new AMap.ToolBar()); // 工具条控件;范围选择控件
      map.value.addControl(new AMap.Scale()); // 显示当前地图中心的比例尺
      map.value.addControl(new AMap.HawkEye()); // 显示缩略图
      // map.value.addControl(new AMap.Geolocation()) // 定位当前位置
      map.value.addControl(new AMap.MapType()); // 实现默认图层与卫星图,实时交通图层之间切换
    }
  );
}
```

## 核心代码：

绘制标记点（根据仓库所在经纬度定位）：

```JS
  warehouseList.value.forEach((value, index) => {
    if (value.longitude !== null && value.latitude !== null) {
      var markerPoint = new AMap.Marker({
        position: [value.longitude, value.latitude],
        offset: new AMap.Pixel(0, -3), //点偏移量
      });
      // 将标记点绘制到地图
      map.value.add(markerPoint);
    }
  });
```

绘制区域：

```JS
const handlePolygon = (paths) => {
    let polygon = new AMap.Polygon({
      path: paths,
      fillColor: "#256edc",
      strokeOpacity: 1,
      fillOpacity: 0.5,
      strokeColor: "#256edc",
      strokeWeight: 1,
      strokeStyle: "dashed",
      strokeDasharray: [5, 5],
    });
    map.value.add(polygon);
  };
```

注：非带孔区域（单个或多个）、带 1 个孔区域框选、带多个孔区域框选主要是数据格式差异，其中：
非带孔区域（单个或多个）：[[[[121.7789, 31.3102],...]],[[[121.627, 31.445],...]],[[[121.8018, 31.357],...]]]
带 1 个孔区域：[[[113.321171, 23.119323],...],[[113.326809, 23.118568],...]]
带多个孔区域：[[[113.288443, 23.112144],...],[[113.289258, 23.110644],...],[[113.321058, 23.111394],...],[[113.328526, 23.109776],...]]
把对应数据传入 handlePolygon 函数 paths 参数中执行，即可生成区域

完整代码:

```JS
<template>
  <div class="page-container">
    <div class="relative">
      <div class="side-container">
        <div class="w-full flex flex-col justify-center items-center py-8 px-2">
          <el-icon :size="24" @click="isCollapse = true"><Operation /></el-icon>
        </div>
        <div class="border-t border-gray-300 m-auto" style="width: calc(100% - 24px)"></div>
        <div class="side-warehouse-container flex-1 flex overflow-y-auto py-4 px-2">
          <ul>
            <li
              v-for="(im, ix) in warehouseList"
              :key="ix"
              class="text-center text-xs text-666 py-3 cursor-pointer"
              :class="{ 'active-market': activeMarker == ix }"
              @click="markerSideClick(markerPointList[ix], ix)"
            >
              <div class="w-10 h-10 rounded-md overflow-hidden m-auto mb-1">
                <img class="fit-cover" src="../../assets/img/a2.jpg" alt="" />
              </div>
              <div class="line-clamp-2">
                {{ im.name }}
              </div>
            </li>
          </ul>
        </div>
      </div>
      <el-drawer v-model="isCollapse" title="I am the title" :size="320" direction="ltr">
        <span>Hi, there!</span>
      </el-drawer>
    </div>
    <div class="map-container">
      <div id="container"></div>
      <div class="info-container" v-if="showInfo">
        <div class="flex justify-between items-center px-4 py-4 border-b border-gray-200 text-lg">
          {{ currentWarehouse.name }}
          <el-icon @click="closeLeftWindow" :size="24"><Close /></el-icon>
        </div>
      </div>
      <div ref="infoData" class="pb-3" @mouseover="atInfoWindow = true" @mouseleave="infoWindowMouseLeave">
        <div
          class="bg-white rounded"
          style="box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 2px 6px 2px rgba(60, 64, 67, 0.15)"
        >
          <div class="flex justify-between items-center px-4 py-2 border-b border-gray-200">
            <span>仓库信息</span>
            <el-icon @click="closeInfoWindow" class="cursor-pointer"><Close /></el-icon>
          </div>
          <div
            v-for="(im, ix) in warehouseInfo"
            :key="ix"
            class="px-4 cursor-pointer"
            @click="selectWarehouse(im)"
          >
            <div :class="{ 'border-t': ix > 0 }" class="border-gray-200 py-3">
              <div class="text-black">仓库名称：{{ im.name }}</div>
              <div class="text-gray-700">
                地址：xxxxxxxxx
                <br />
                联系电话：18522345536
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, nextTick } from 'vue'
import AMapLoader from '@amap/amap-jsapi-loader'
import { Close, Operation } from '@element-plus/icons-vue'
const isCollapse = ref(false)
const map = ref()
const currentWarehouse = ref(null)
const warehouseInfo = ref([])
const showInfo = ref(false)
const infoWindow = ref(null)
const atInfoWindow = ref(false)
const centerPoint = [104.098645, 38.254319]  // 全国地图中心点
// 标记点对应的仓库数据
const warehouseList = ref([
  {
    longitude: 113.326938,
    latitude: 23.118104,
    statusType: 'position-tag',
    name: '广州1仓'
  },
  {
    longitude: 113.326938,
    latitude: 23.118104,
    statusType: 'position-tag',
    name: '广州2仓'
  },
  {
    longitude: 106.550539,
    latitude: 29.502387,
    statusType: 'position-tag',
    name: '重庆仓'
  },
  {
    longitude: 115.866945,
    latitude: 28.715301,
    statusType: 'position-tag',
    name: '南昌仓'
  },
  {
    longitude: 116.361536,
    latitude: 39.92098,
    statusType: 'position-tag',
    name: '北京仓'
  },
  {
    longitude: 87.487317,
    latitude: 43.808379,
    statusType: 'position-tag',
    name: '乌鲁木齐仓'
  },
  {
    longitude: 101.725598,
    latitude: 36.533486,
    statusType: 'position-tag',
    name: '西宁仓'
  },
  {
    longitude: 91.161109,
    latitude: 29.723416,
    statusType: 'position-tag',
    name: '拉萨仓'
  }
])
// 标记点列表
let markerPointList = []
const activeMarker = ref(null)
const initMap = async () => {
  window._AMapSecurityConfig = {
    securityJsCode: '7788cd228050738a8d7dcb1978b2fb19'
  }
  await AMapLoader.load({
    key: '700ae1a3c31530a5bfa73b5aa8b0b862', // 申请好的Web端开发者Key，首次调用 load 时必填
    version: '2.0', // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
    plugins: [''] // 需要使用的的插件列表，如比例尺'AMap.Scale'等
  })
  map.value = new AMap.Map('container', {
    //设置地图容器id
    viewMode: '3D', //是否为3D地图模式
    zoom: 5, //初始化地图级别
    center: centerPoint //初始化地图中心点位置
  })

  // 自定义 官方默认自定义样式
  var styleName = 'amap://styles/normal'
  map.value.setMapStyle(styleName)

  // 添加插件
  map.value.plugin(
    ['AMap.ToolBar', 'AMap.Scale', 'AMap.HawkEye', 'AMap.Geolocation', 'AMap.MapType', 'AMap.MouseTool'],
    function () {
      //异步同时加载多个插件
      // 添加地图插件
      map.value.addControl(new AMap.ToolBar()) // 工具条控件;范围选择控件
      map.value.addControl(new AMap.Scale()) // 显示当前地图中心的比例尺
      map.value.addControl(new AMap.HawkEye()) // 显示缩略图
      // map.value.addControl(new AMap.Geolocation()) // 定位当前位置
      map.value.addControl(new AMap.MapType()) // 实现默认图层与卫星图,实时交通图层之间切换
    }
  )

  var contextMenu = new AMap.ContextMenu()
  //右键放大
  contextMenu.addItem(
    '放大一级',
    function () {
      map.value.zoomIn()
    },
    0
  )
  //右键缩小
  contextMenu.addItem(
    '缩小一级',
    function () {
      map.value.zoomOut()
    },
    1
  )
  //右键显示全国范围
  contextMenu.addItem(
    '缩放至全国范围',
    function (e) {
      map.value.setZoomAndCenter(5, centerPoint)
    },
    2
  )

  //地图绑定鼠标右击事件——弹出右键菜单
  map.value.on('rightclick', function (e) {
    contextMenu.open(map.value, e.lnglat)
  })

  let shanghai = [
    [
      [
        [121.7789, 31.3102],
        [121.7279, 31.3548],
        [121.5723, 31.4361],
        [121.5093, 31.4898],
        [121.5624, 31.4864],
        [121.5856, 31.4547],
        [121.7694, 31.3907],
        [121.796, 31.3456],
        [121.7789, 31.3102]
      ]
    ],
    [
      [
        [121.627, 31.445],
        [121.5942, 31.4586],
        [121.5758, 31.4782],
        [121.6137, 31.4713],
        [121.635, 31.453],
        [121.627, 31.445]
      ]
    ],
    [
      [
        [121.8018, 31.357],
        [121.7939, 31.3805],
        [121.8759, 31.3642],
        [121.8882, 31.3227],
        [121.8725, 31.2968],
        [121.8406, 31.2954],
        [121.8038, 31.3284],
        [121.8018, 31.357]
      ]
    ],
    [
      [
        [121.9752, 31.617],
        [121.9917, 31.4768],
        [121.918, 31.4347],
        [121.8454, 31.4319],
        [121.6088, 31.5069],
        [121.4342, 31.5903],
        [121.3955, 31.5854],
        [121.3722, 31.5532],
        [121.1453, 31.7539],
        [121.1185, 31.7591],
        [121.2003, 31.8351],
        [121.3104, 31.8725],
        [121.385, 31.8335],
        [121.4315, 31.7693],
        [121.4986, 31.7533],
        [121.5933, 31.7046],
        [121.9752, 31.617]
      ]
    ],
    [
      [
        [121.1571, 31.4114],
        [121.1462, 31.4211],
        [121.1643, 31.4272],
        [121.1643, 31.4311],
        [121.1473, 31.4439],
        [121.2304, 31.4774],
        [121.2352, 31.4931],
        [121.2594, 31.4779],
        [121.3435, 31.5121],
        [121.4054, 31.4872],
        [121.5212, 31.3948],
        [121.5989, 31.3746],
        [121.7225, 31.3035],
        [121.9627, 31.0473],
        [121.9985, 30.9],
        [121.994, 30.8631],
        [121.9547, 30.8258],
        [121.9697, 30.7892],
        [121.9437, 30.7771],
        [121.9045, 30.8142],
        [121.6484, 30.8162],
        [121.5172, 30.7754],
        [121.3619, 30.6795],
        [121.2747, 30.6774],
        [121.2715, 30.7327],
        [121.2355, 30.7523],
        [121.218, 30.785],
        [121.1747, 30.772],
        [121.1178, 30.7848],
        [121.1376, 30.8262],
        [121.1177, 30.8353],
        [121.1217, 30.85],
        [121.067, 30.8488],
        [121.0379, 30.8139],
        [121.0144, 30.8358],
        [120.9909, 30.8227],
        [121.0219, 30.8752],
        [120.9904, 30.8956],
        [121.0046, 30.9093],
        [120.9904, 31.0138],
        [120.9522, 31.0303],
        [120.9398, 31.0094],
        [120.911, 31.0106],
        [120.8946, 31.0539],
        [120.9021, 31.0857],
        [120.8569, 31.105],
        [120.8813, 31.1347],
        [121.0185, 31.1341],
        [121.0408, 31.1376],
        [121.0454, 31.154],
        [121.0691, 31.1487],
        [121.0625, 31.2675],
        [121.0821, 31.2715],
        [121.0888, 31.2921],
        [121.0988, 31.2763],
        [121.1054, 31.2737],
        [121.1206, 31.287],
        [121.1616, 31.283],
        [121.1438, 31.3097],
        [121.1299, 31.3026],
        [121.1304, 31.3442],
        [121.1068, 31.3645],
        [121.1484, 31.3854],
        [121.1571, 31.4114]
      ]
    ],
    [
      [
        [121.9433, 31.2155],
        [121.9573, 31.2304],
        [122.0086, 31.221],
        [121.9957, 31.1608],
        [121.9596, 31.1593],
        [121.9433, 31.2155]
      ]
    ]
  ]
  let singleHole = [
    [
      [113.321171, 23.119323],
      [113.32116, 23.114014],
      [113.333729, 23.113068],
      [113.333708, 23.118347]
    ],
    [
      [113.326809, 23.118568],
      [113.327828, 23.118662],
      [113.327828, 23.11769],
      [113.326847, 23.11768]
    ]
  ]
  var multiHole = [
    [
      [113.288443, 23.112144],
      [113.286769, 23.107684],
      [113.33389, 23.106065],
      [113.333976, 23.112815],
      [113.315308, 23.114236]
    ],
    [
      [113.289258, 23.110644],
      [113.30445, 23.10717],
      [113.319599, 23.110328],
      [113.30548, 23.112065]
    ],
    [
      [113.321058, 23.111394],
      [113.323161, 23.110368],
      [113.330071, 23.111078],
      [113.325607, 23.112262]
    ],
    [
      [113.328526, 23.109776],
      [113.329599, 23.109855],
      [113.330071, 23.107999]
    ]
  ]
  // 绘制区域
  const handlePolygon = (paths) => {
    let polygon = new AMap.Polygon({
      path: paths,
      fillColor: '#256edc',
      strokeOpacity: 1,
      fillOpacity: 0.5,
      strokeColor: '#256edc',
      strokeWeight: 1,
      strokeStyle: 'dashed',
      strokeDasharray: [5, 5]
    })
    polygon.on('mouseover', () => {
      polygon.setOptions({
        fillOpacity: 0.7,
        fillColor: '#256edc'
      })
      console.log('鼠标移入')
    })
    polygon.on('mouseout', () => {
      polygon.setOptions({
        fillOpacity: 0.5,
        fillColor: '#256edc'
      })
      console.log('鼠标移出')
    })
    map.value.add(polygon)
  }
  await handlePolygon(shanghai)
  await handlePolygon(singleHole)
  await handlePolygon(multiHole)

  await nextTick()
  markerPointList = []
  // 绘制标记点
  warehouseList.value.forEach((value, index) => {
    if (value.longitude !== null && value.latitude !== null) {
      var markerPoint = new AMap.Marker({
        position: [value.longitude, value.latitude],
        offset: new AMap.Pixel(0, -3) //点偏移量
      })
      // markerPoint.setLabel({
      //   offset: new AMap.Pixel(0, -4), //设置文本标注偏移量
      //   content: "<div class='warehouse-info'>" + value.name + '</div>', //设置文本标注内容
      //   direction: 'top' //设置文本标注方位
      // })
      markerPointList.push(markerPoint)
      markerPoint.on('click', (e) => markerClick(e, index))
      markerPoint.on('mouseover', (e) => markerMouseover(e, index))
      markerPoint.on('mouseout', (e) => markerMouseout(e, index))
      // 创建标记点的div
      var markerDiv = document.createElement('div')
      // 设置标记点className,用于设置点的样式（动画）
      markerDiv.className = value.statusType
      // 点内部文字:找到相同经纬度点列表:sameLonLatPoint，列表长度即为该点显示个数
      var sameLonLatPoint = warehouseList.value.filter((val) => {
        return val.longitude === value.longitude && val.latitude === value.latitude
      })
      // 创建标记点内容span
      var markerSpan = document.createElement('span')
      // 某位置点不唯一时展示个数
      // markerSpan.innerText = sameLonLatPoint.length !== 1 ? sameLonLatPoint.length : ''
      // 位置点均展示个数
      markerSpan.innerText = sameLonLatPoint.length
      // 将内容span放到标记点div
      markerDiv.appendChild(markerSpan)
      // 将标记点div，设置为标记点内容
      markerPoint.setContent(markerDiv)
      // 将标记点绘制到地图
      map.value.add(markerPoint)
    }
  })
}

const markerSideClick = async (e, index) => {
  console.log(map.value)
  markerPointList.forEach((im) => {
    if (im._position[0] == e._position[0] && im._position[1] == e._position[1]) {
      im.dom.children[0].className += ' markerBreath'
    } else {
      im.dom.children[0].className = 'position-tag'
    }
  })
  map.value.setZoomAndCenter(5, e._position)
  activeMarker.value = index
  selectWarehouse(warehouseList.value[index])
}

// 鼠标点击标记点
const markerClick = (e, index) => {
  console.log('鼠标点击标记点：', e)
  showInfo.value = false
  // 放大地图
  // map是地图本身 第一个参数14是地图显示的缩放级别 第二个参数是高德的自带的函数getPosition,拿到当前标记点的经纬度
  map.value.setZoomAndCenter(14, e.target.getPosition())
  let position = e.target.getPosition()
  console.log(position)

  markerPointList.forEach((im) => {
    if (im._position[0] == position.lng && im._position[1] == position.lat) {
      im.dom.children[0].className += ' markerBreath'
    } else {
      im.dom.children[0].className = 'position-tag'
    }
  })

  let warehouses = warehouseList.value.filter(
    (im) => position.lat === im.latitude && position.lng === im.longitude
  )
  if (warehouses.length > 1) {
    console.log('同一区域存在多个仓，不直接走下一步展示仓库信息')
    activeMarker.value = null
    return // 同一区域存在多个仓，不直接走下一步展示仓库信息
  }
  activeMarker.value = index
  selectWarehouse(warehouseList.value[index])
}

// 鼠标在标记点上
const markerMouseover = async (e, index) => {
  console.log('鼠标在标记点上：', e)
  let position = e.target.getPosition()
  let warehouses = warehouseList.value.filter(
    (im) => position.lat === im.latitude && position.lng === im.longitude
  )
  warehouseInfo.value = warehouses
  infoWindow.value = await createInfoWindow()
  //打开信息窗体
  await openInfoWindow(e.target)
}
// 鼠标移开标记点
const markerMouseout = (e, index) => {
  //打开信息窗体
  // infoWindow.value.close()
  if (!atInfoWindow.value) {
    closeInfoWindow()
  }
}

// 构建自定义窗体
let infoData = ref()
const createInfoWindow = () => {
  let infoWindowData = new AMap.InfoWindow({
    isCustom: true, //使用自定义窗体
    content: infoData.value,
    offset: new AMap.Pixel(0, -35)
  })
  return infoWindowData
}

// 打开窗体
const openInfoWindow = (marker) => {
  infoWindow.value.open(map.value, marker.getPosition())
}

// 关闭窗体
const closeInfoWindow = () => {
  map.value.clearInfoWindow()
}
const infoWindowMouseLeave = () => {
  atInfoWindow.value = false
  warehouseInfo.value = []

  closeInfoWindow()
}

// 选择展示的仓库
const selectWarehouse = (warehouse) => {
  //左侧弹窗的内容
  currentWarehouse.value = warehouse
  showInfo.value = true
  let fIndex = warehouseList.value.findIndex((im) => im.name == warehouse.name)
  console.log(fIndex)
  activeMarker.value = fIndex
  markerPointList.forEach((im) => {
    if (im._position[0] == warehouse.longitude && im._position[1] == warehouse.latitude) {
      im.dom.children[0].className += ' markerBreath'
    } else {
      im.dom.children[0].className = 'position-tag'
    }
  })
}

const closeLeftWindow = () => {
  showInfo.value = false
  markerPointList.forEach((im) => {
    im.dom.children[0].className = 'position-tag'
  })
  activeMarker.value = null
  atInfoWindow.value = false
  warehouseInfo.value = []
  map.value.setZoomAndCenter(5, centerPoint)
}

onMounted(() => {
  initMap()
})
</script>

<style lang="less">
.page-container {
  width: 100vw;
  height: 100vh;
  display: flex;
}
.side-container {
  background: #fff;
  width: 72px;
  height: 100%;
  position: relative;
  z-index: 2;
  box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 2px 6px 2px rgba(60, 64, 67, 0.15);
  transition: all 0.2s ease;
  display: flex;
  flex-direction: column;
}
/*定义滚动条高宽及背景 高宽分别对应横竖滚动条的尺寸*/
.side-warehouse-container::-webkit-scrollbar {
  width: 6px;
  height: 6px;
  background-color: #f5f5f5;
}

/*定义滚动条轨道 内阴影+圆角*/
.side-warehouse-container::-webkit-scrollbar-track {
  border-radius: 10px;
  background-color: #f5f5f5;
}

/*定义滑块 内阴影+圆角*/
.side-warehouse-container::-webkit-scrollbar-thumb {
  border-radius: 10px;
  -webkit-box-shadow: inset 0 0 6px rgba(172, 172, 172, 0.3);
  background-color: #bebebe;
}
.active-market {
  color: rgb(63, 175, 250);
}
.map-container {
  flex: 1;
  width: 100%;
  height: 100vh;
  position: relative;
}
#container {
  padding: 0px;
  margin: 0px;
  width: 100%;
  height: 100%;
}
.info-container {
  position: absolute;
  z-index: 999;
  top: 16px;
  left: 16px;
  width: 350px;
  height: calc(100vh - 32px);
  background-color: #fff;
  box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 2px 6px 2px rgba(60, 64, 67, 0.15);
  border-radius: 12px;
}
.position-tag {
  text-align: center;
  margin: 0 auto;
  width: 40px;
  height: 40px;
  background-image: url('../../assets/img/position.png'); // 标记定位图标图片
  background-size: 100%;
}
.markerBreath {
  background-image: url('../../assets/img/position-current.png'); // 标记定位图标图片
  -webkit-animation-name: 'markerBreath'; /*动画属性名，也就是我们前面keyframes定义的动画名*/
  -webkit-animation-duration: 0.8s; /*动画持续时间*/
  -webkit-animation-timing-function: ease; /*动画频率，和transition-timing-function是一样的*/
  -webkit-animation-delay: 0s; /*动画延迟时间*/
  -webkit-animation-iteration-count: infinite; /*定义循环资料，infinite为无限次*/
  -webkit-animation-direction: alternate; /*定义动画方式*/
}
@keyframes markerBreath {
  0% {
    margin-top: 0;
  }
  100% {
    margin-top: 5px;
  }
}
.normalDevice span,
.offLineDevice span,
.position-tag span {
  line-height: 34px;
  font-size: 16px;
  color: #fff;
  font-weight: 600;
}
</style>

```
