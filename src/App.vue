<template>
  <div id="container">
    <div class="table">
      <div v-for="item in data"
           class="ml-2"
           :class="item.properties.id === currentTag && 'active'"
           type="info"
           @click="setCenter(item)">
        {{ item.properties.name }}
      </div>
    </div>
  </div>
</template>

<script setup>
import {onMounted} from "vue";

import AMapLoader from '@amap/amap-jsapi-loader';
import beijing_json from "./assets/beijing.json";
import chongqing_json from "./assets/chongqing.json";
import panzhihua_json from "./assets/panzhihua.json";
import chengdu_json from "./assets/chengdu.json";
import deyang_json from "./assets/deyang.json";
import tabledata_json from "./assets/tabledata.json";

let currentTag = $ref(-1)
let currentMarker = $ref('')
const {data} = tabledata_json
let map
let globalAMap

// 按钮点击事件
function setCenter(item) {
  console.log(item);
  const {properties: {cp, name, id}} = item
  currentTag = id
  const marker = new globalAMap.Marker({
    position: new globalAMap.LngLat(cp[0], cp[1]),
    title: '',
    icon: 'src/assets/kisser.png',
  });
  if (currentMarker !== '') {
    map.remove(currentMarker)
  }
  currentMarker = marker
// 创建 infoWindow 实例
  const infoWindow = new AMap.InfoWindow({
    content: `
             坐标：${cp}<br/>
             城市：${name}<br/>`
  });
// 打开信息窗体
  map.add(marker)
  map.setCenter(cp)
  map.add(infoWindow)
  setTimeout(() => {
    infoWindow.open(map, [cp[0], cp[1]]);
  }, 500)
}

onMounted(() => {
  AMapLoader.load({
    key: '1cc52c249a56654a972eb0756e51e444',
    version: '2.0',
    plugins: ['AMap.GeoJSON'],
    zoom: 8.8,
    resizeEnable: true,
    center: [106.6663, 29.5367],
    // viewMode: '3D',
  }).then((AMap) => {
    map = new AMap.Map('container')
    globalAMap = AMap

    // 区域描边阴影加多边形绘制
    const geojson = new AMap.GeoJSON({
      geoJSON: chongqing_json,
      // 还可以自定义getMarker和getPolyline
      getPolygon: function (geojson, lnglats) {
        // 计算面积
        const area = AMap.GeometryUtil.ringArea(lnglats[0]);
        const polygon = new AMap.Polygon({
          path: lnglats,
          fillOpacity: 0.8 - Math.sqrt(area / 9000000000),// 面积越大透明度越高
          strokeColor: 'white',
          fillColor: 'red'
        })
        // 鼠标点击事件
        polygon.on('click', () => {
          setCenter(geojson)
        })
        // 鼠标移入
        polygon.on('mouseover', () => {
          polygon.setOptions({
            fillOpacity: 0.7,
            fillColor: '#facb68'
          })
        })
        // 鼠标移出
        polygon.on('mouseout', () => {
          polygon.setOptions({
            fillOpacity: 0.8 - Math.sqrt(area / 9000000000),// 面积越大
            fillColor: 'red'
          })
        })
        return polygon
      }
    });
    map.add(geojson)

    /*    // 两个点之间的距离
    const m1 = new AMap.Marker({
      map: map,
      draggable: true,
      position: new AMap.LngLat(109.82902, 34.367209)
    });
    const m2 = new AMap.Marker({
      map: map,
      draggable: true,
      position: new AMap.LngLat(103.934423, 30.478674)
    });
    map.setFitView();
    const line = new AMap.Polyline({
      strokeColor: '#80d8ff',
      isOutline: true,
      outlineColor: 'white'
    });
    line.setMap(map);
    const text = new AMap.Text({
      text: '',
      style: {
        'background-color': '#29b6f6',
        'border-color': '#e1f5fe',
        'font-size': '12px'
      }
    });
    text.setMap(map)
    function computeDis() {
      const p1 = m1.getPosition();
      const p2 = m2.getPosition();
      const textPos = p1.divideBy(2).add(p2.divideBy(2));
      const distance = Math.round(p1.distance(p2));
      const path = [p1, p2];
      line.setPath(path);
      text.setText('两点相距' + distance + '米')
      text.setPosition(textPos)
    }
    computeDis();
    m1.on('dragging', computeDis)
    m2.on('dragging', computeDis)*/

    //点哪儿哪儿就画图标
    map.on('click', function (e) {
      const {lng, lat} = e.lnglat
      console.log(e)
      const marker = new AMap.Marker({
        position: new AMap.LngLat(lng, lat),
        title: '',
        icon: 'src/assets/kisser.png',
      });
      if (currentMarker !== '') {
        map.remove(currentMarker)
      }
      map.add(marker)
      currentMarker = marker
    })
  })
})

</script>

<style scoped lang="scss">
#container {
  padding: 0;
  margin: 0;
  width: 100vw;
  height: 100vh;
  position: relative;

  .table {
    position: absolute;
    top: 10px;
    left: 20px;
    height: 35px;
    width: 300px;
    display: flex;
    justify-content: space-around;
    flex-wrap: wrap; //自动换行
    z-index: 9999;

    .ml-2 {
      background: #d5d3d3;
      width: 80px;
      height: 35px;
      align-items: center;
      text-align: center;
      line-height: 35px;
      border-radius: 15px;
      -o-text-overflow: ellipsis;
      margin-bottom: 8px;
      white-space: nowrap;

      overflow: hidden;
      text-overflow: ellipsis;
    }

    .ml-2:hover {
      background: palevioletred;
      color: #fff;
    }

    .active {
      background: palevioletred;
      color: #fff;
    }
  }
}

.btn {
  position: relative;
  width: 12rem;
  left: 3.6rem;
  margin: 10px 0 0 0;
}
</style>
