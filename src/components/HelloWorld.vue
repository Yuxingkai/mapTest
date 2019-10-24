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
      msg: 'Welcome to Your Vue.js App'
    }
  },
  methods: {
    init () {
      // 创建地图
      let map = new AMap.Map('mapUI', {
        zoom: 4
      })

      AMapUI.load(['ui/misc/PathSimplifier', 'lib/$'], function (PathSimplifier, $) {
        if (!PathSimplifier.supportCanvas) {
          alert('当前环境不支持 Canvas！')
          return
        }

        // just some colors
        var colors = [
          '#3366cc', '#dc3912', '#ff9900', '#109618', '#990099', '#0099c6', '#dd4477', '#66aa00',
          '#b82e2e', '#316395', '#994499', '#22aa99', '#aaaa11', '#6633cc', '#e67300', '#8b0707',
          '#651067', '#329262', '#5574a6', '#3b3eac'
        ]

        var pathSimplifierIns = new PathSimplifier({
          zIndex: 100,
          // autoSetFitView:false,
          map: map, // 所属的地图实例

          getPath: function (pathData, pathIndex) {
            return pathData.path
          },
          getHoverTitle: function (pathData, pathIndex, pointIndex) {
            if (pointIndex >= 0) {
              // point
              return pathData.name + '，点：' + pointIndex + '/' + pathData.path.length
            }

            return pathData.name + '，点数量' + pathData.path.length
          },
          renderOptions: {
            pathLineStyle: {
              dirArrowStyle: true
            },
            getPathStyle: function (pathItem, zoom) {
              var color = colors[pathItem.pathIndex % colors.length],
                lineWidth = Math.round(4 * Math.pow(1.1, zoom - 3))

              return {
                pathLineStyle: {
                  strokeStyle: color,
                  lineWidth: lineWidth
                },
                pathLineSelectedStyle: {
                  lineWidth: lineWidth + 2
                },
                pathNavigatorStyle: {
                  fillStyle: color
                }
              }
            }
          }
        })

        window.pathSimplifierIns = pathSimplifierIns

        $('<div id="loadingTip">加载数据，请稍候...</div>').appendTo(document.body)
        setTimeout(() => {
          $('#loadingTip').remove()
          var flyRoutes = []

          for (var i = 0, len = d.length; i < len; i++) {
            if (d[i].name.indexOf('乌鲁木齐') >= 0) {
              d.splice(i, 0, {
                name: '飞行 - ' + d[i].name,
                path: PathSimplifier.getGeodesicPath(
                  d[i].path[0], d[i].path[d[i].path.length - 1], 100)
              })

              i++
              len++
            }
          }

          d.push.apply(d, flyRoutes)
          pathSimplifierIns.setData(d)

          // initRoutesContainer(d);

          function onload () {
            pathSimplifierIns.renderLater()
          }

          function onerror (e) {
            console.log(e)
            alert('图片加载失败！')
          }

          // 创建一个巡航器
          var navg1 = pathSimplifierIns.createPathNavigator(0, {
            loop: true,
            speed: 100000,
            pathNavigatorStyle: {
              width: 48,
              height: 48,
              // 使用图片
              content: PathSimplifier.Render.Canvas.getImageContent('../assets/smallP.gif', onload, onerror),
              strokeStyle: null,
              fillStyle: null,
              initRotateDegree: 90,
              // 经过路径的样式
              pathLinePassedStyle: {
                lineWidth: 6,
                strokeStyle: 'black',
                dirArrowStyle: {
                  stepSpace: 15,
                  strokeStyle: 'red'
                }
              }
            }
          })

          navg1.start()
        }, 500)
      })
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
</style>
