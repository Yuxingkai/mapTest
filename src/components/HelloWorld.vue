<template>
  <div id="mapUI">
    mapUI
  </div>
</template>

<script>
const d = require('../assets/mock/data.json')
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      imgUrl: require('../assets/smallP.gif'),
      imgL: require('../assets/logo.png'),
      globleMarkers: [],
      bounseMarker: null
    }
  },
  methods: {
    popInfo (e) {
      let infoWindow = new AMap.InfoWindow({offset: new AMap.Pixel(0, -30)})
      window.infoWindow = infoWindow
      console.log(e.target.content)
      infoWindow.setContent(e.target.content);
      infoWindow.open(window.map, e.target.getPosition())
    },
    creatMarker (map) {
      let markers = [];
      let lineArr = d[0].path
      let newArr = [[121.458509,31.166479],[121.457817,31.166369],[121.456642,31.166209],[121.455574,31.165965],[121.456057,31.165056],[121.456256,31.164643],[121.456519,31.164152],[121.456706,31.16378],[121.457178,31.16401],[121.457017,31.164501],[121.456894,31.165056],[121.456814,31.165428],[121.456261,31.165658]]
      for (var i = 0; i < newArr.length; i++) {
        var lnglat = newArr[i];
        // 创建点实例
        var marker = new AMap.Marker({
          offset: new AMap.Pixel(-5, -5),
          position: new AMap.LngLat(lnglat[0], lnglat[1]),
          content: '<div style="width: 10px;height: 10px;border-radius: 5px;background: red"></div>',
          extData: {
            id: i + 1
          }
        });
        marker.content= '坐标' + lnglat[0] + lnglat[1] + '<br>' + '<img style="width: 50px;height: 50px;" src="' + this.imgL + 'alt="">'
        marker.on('click', this.popInfo)
        markers.push(marker);
        this.globleMarkers = markers
      }

      // 创建覆盖物群组，并将 marker 传给 OverlayGroup
      var overlayGroups = new AMap.OverlayGroup(markers);

      // 添加覆盖物群组
      function addOverlayGroup() {

        map.add(overlayGroups);
      }
      addOverlayGroup()
    },
    init () {
      // 创建地图
      let map = new AMap.Map('mapUI', {
        zooms: [10, 20],
        resizeEnable: true,
        rotateEnable:true,
        pitchEnable:false,
        zoom: 18.4,
        mapStyle: 'amap://styles/b0cb4e73e4bd6c0cf3aa8991b18388fd',
        // mapStyle:'amap://styles/76a1adafcd53c65345db2b1c56c8cec2',
        pitch:30,
        viewMode:'3D',//开启3D视图,默认为关闭
        expandZoomRange:true,
      })
      let infoWindow = new AMap.InfoWindow({offset: new AMap.Pixel(0, -30)})
      window.infoWindow = infoWindow
      window.map = map
      // let i = 0.5
      // setInterval(() => {
      //   i += 0.5
      //   window.map.setRotation(i)
      // }, 100)
      let lineArr = d[0].path
      let setLocalArr = [[121.458509,31.166479],[121.457817,31.166369],[121.456642,31.166209],[121.455574,31.165965],[121.456057,31.165056],[121.456256,31.164643],[121.456519,31.164152],[121.456706,31.16378],[121.457178,31.16401],[121.457017,31.164501],[121.456894,31.165056],[121.456814,31.165428],[121.456261,31.165658]]
      // lineArr[0] = [121.458353,31.166548]
      console.log(lineArr[0])
      if (setLocalArr[0][0] > setLocalArr[setLocalArr.length - 1][0]) {
        this.imgUrl = require('../assets/transR.gif')
      } else {
        this.imgUrl = require('../assets/transL.gif')
      }
      // 创建一个 Icon
      var startIcon = new AMap.Icon({
        // 图标尺寸
        size: new AMap.Size(50, 65),
        // 图标的取图地址
        image: this.imgUrl,
        // 图标所用图片大小
        imageSize: new AMap.Size(50, 55),
        // 图标取图偏移量
        imageOffset: new AMap.Pixel(0, 0)
      });
      let marker = new AMap.Marker({
        map: map,
        position: setLocalArr[0],
        // content: content,
        icon: startIcon,
        offset: new AMap.Pixel(-25, -25),
        autoRotation: true,
        angle: 0,
      });
      this.bounseMarker = marker
      // 绘制轨迹
      var polyline = new AMap.Polyline({
        map: map,
        path: setLocalArr,
        showDir:true,
        strokeColor: "#28F",  //线颜色
        // strokeOpacity: 1,     //线透明度
        strokeWeight: 6,      //线宽
        // strokeStyle: "solid"  //线样式
      });

      var passedPolyline = new AMap.Polyline({
        map: map,
        // path: lineArr,
        strokeColor: "#AF5",  //线颜色
        // strokeOpacity: 1,     //线透明度
        strokeWeight: 6,      //线宽
        // strokeStyle: "solid"  //线样式
      });

      marker.on('moving', (e) => {
        passedPolyline.setPath(e.passedPath);
        console.log(e.passedPath)
        // let content = '坐标' + e.passedPath[e.passedPath.length - 1].lng + e.passedPath[e.passedPath.length - 1].lat + '<br>' + '<img style="width: 50px;height: 50px;" src="' + this.imgL + 'alt="">'
        // let lnglat = [121.457817,31.166369]
        for (let i = 0; i < this.globleMarkers.length; i++) {
          console.log(this.globleMarkers[i].getPosition())
          if (e.passedPath[e.passedPath.length - 1].lng === this.globleMarkers[i].getPosition().lng && e.passedPath[e.passedPath.length - 1].lat === this.globleMarkers[i].getPosition().lat) {
            this.bounseMarker.setAnimation('AMAP_ANIMATION_DROP')
            window.infoWindow.setContent(this.globleMarkers[e.passedPath.length - 1].content);
            window.infoWindow.open(window.map, [this.globleMarkers[i].getPosition().lng, this.globleMarkers[i].getPosition().lat])
            // setTimeout(() => {
            //   window.infoWindow.close()
            // }, 1000)
          }
        }
        // if (e.passedPath.length === 5) {
        //   let i = 0
        //   let time = setInterval(() => {
        //     i++
        //     if (i > 45) {
        //       clearInterval(time)
        //     } else {
        //       window.map.setRotation(i)
        //     }
        //   }, 500)
        // } else if (e.passedPath.length === 8) {
        //   for (let i = 45; i < 90; i++) {
        //     window.map.setRotation(i)
        //   }
        // }
      });

      map.setFitView();
      function startAnimation () {
        marker.moveAlong(setLocalArr, 100,'' , false);
      }

      function pauseAnimation () {
        marker.pauseMove();
      }

      function resumeAnimation () {
        marker.resumeMove();
      }

      function stopAnimation () {
        marker.stopMove();
      }
      this.creatMarker(map)
      startAnimation()
      let that = this
      // AMapUI.load(['ui/misc/PathSimplifier', 'lib/$'], function (PathSimplifier, $) {
      //   if (!PathSimplifier.supportCanvas) {
      //     alert('当前环境不支持 Canvas！')
      //     return
      //   }
      //
      //   // just some colors
      //   var colors = [
      //     '#3366cc', '#dc3912', '#ff9900', '#109618', '#990099', '#0099c6', '#dd4477', '#66aa00',
      //     '#b82e2e', '#316395', '#994499', '#22aa99', '#aaaa11', '#6633cc', '#e67300', '#8b0707',
      //     '#651067', '#329262', '#5574a6', '#3b3eac'
      //   ]
      //
      //   var pathSimplifierIns = new PathSimplifier({
      //     zIndex: 100,
      //     // autoSetFitView:false,
      //     map: map, // 所属的地图实例
      //
      //     getPath: function (pathData, pathIndex) {
      //       return pathData.path
      //     },
      //     getHoverTitle: function (pathData, pathIndex, pointIndex) {
      //       if (pointIndex >= 0) {
      //         // point
      //         return pathData.name + '，点：' + pointIndex + '/' + pathData.path.length
      //       }
      //
      //       return pathData.name + '，点数量' + pathData.path.length
      //     },
      //     renderOptions: {
      //       pathLineStyle: {
      //         dirArrowStyle: true
      //       },
      //       getPathStyle: function (pathItem, zoom) {
      //         var color = colors[pathItem.pathIndex % colors.length],
      //           lineWidth = Math.round(4 * Math.pow(1.1, zoom - 3))
      //
      //         return {
      //           pathLineStyle: {
      //             strokeStyle: color,
      //             lineWidth: lineWidth
      //           },
      //           pathLineSelectedStyle: {
      //             lineWidth: lineWidth + 2
      //           },
      //           pathNavigatorStyle: {
      //             fillStyle: color
      //           }
      //         }
      //       }
      //     }
      //   })
      //
      //   window.pathSimplifierIns = pathSimplifierIns
      //
      //   $('<div id="loadingTip">加载数据，请稍候...</div>').appendTo(document.body)
      //   setTimeout(() => {
      //     $('#loadingTip').remove()
      //     var flyRoutes = []
      //
      //     for (var i = 0, len = d.length; i < len; i++) {
      //       if (d[i].name.indexOf('乌鲁木齐') >= 0) {
      //         d.splice(i, 0, {
      //           name: '飞行 - ' + d[i].name,
      //           path: PathSimplifier.getGeodesicPath(
      //             d[i].path[0], d[i].path[d[i].path.length - 1], 100)
      //         })
      //
      //         i++
      //         len++
      //       }
      //     }
      //
      //     d.push.apply(d, flyRoutes)
      //     pathSimplifierIns.setData(d)
      //
      //     // initRoutesContainer(d);
      //
      //     function onload () {
      //       pathSimplifierIns.renderLater()
      //     }
      //
      //     function onerror (e) {
      //       console.log(e)
      //       alert('图片加载失败！')
      //     }
      //
      //     // 创建一个巡航器
      //     var navg1 = pathSimplifierIns.createPathNavigator(0, {
      //       loop: true,
      //       speed: 100000,
      //       pathNavigatorStyle: {
      //         width: 48,
      //         height: 48,
      //         // 使用图片
      //         content: PathSimplifier.Render.Canvas.getImageContent(that.imgUrl, onload, onerror),
      //         strokeStyle: null,
      //         fillStyle: null,
      //         initRotateDegree: 90,
      //         // 经过路径的样式
      //         pathLinePassedStyle: {
      //           lineWidth: 6,
      //           strokeStyle: 'black',
      //           dirArrowStyle: {
      //             stepSpace: 15,
      //             strokeStyle: 'red'
      //           }
      //         }
      //       }
      //     })
      //
      //     navg1.start()
      //   }, 500)
      // })
    }
  },
  mounted () {
    this.init()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #mapUI {
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    right: 0;
  }
  .markerimg {
    background-image: url("../assets/smallP.gif");
    background-repeat: no-repeat;
    background-size: cover;
    width: 1rem;
    height: 1rem;
  }
</style>
