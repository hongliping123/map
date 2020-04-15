<template>
  <div>
    <div id="js-container" class="map">正在加载数据 ...</div>
  </div>
</template>

<script>
import { MapKey, MapCityName } from "../../config/config";
import remoteLoad from "../../utils/remoteLoad";
export default {
  name: "HelloWorld",
  data() {
    return {
      AMapUI: null,
      AMap: null,
      map: null
    };
  },
  methods: {
    // 实例化地图
    initMap() {
      var _this = this;
      // 加载PositionPicker，loadUI的路径参数为模块名中 'ui/' 之后的部分
      let AMapUI = (this.AMapUI = window.AMapUI);
      let AMap = (this.AMap = window.AMap);
      let mapConfig = {
        zoom: 8
        // cityName: MapCityName
      };
      _this.map = new AMap.Map("js-container", mapConfig);
      AMapUI.loadUI(["geo/DistrictExplorer"], function(DistrictExplorer) {
        // 绘制载入区域
        _this.loadnode(DistrictExplorer);
      });
    },
    // 区域聚合
    clustering() {
      var _this = this;
      AMapUI.load(["ui/geo/DistrictCluster", "lib/$"], function(
        DistrictCluster,
        $
      ) {
        var distCluster = new DistrictCluster({
          map: _this.map, //所属的地图实例
          zIndex: 11,
          areaNodeCacheLimit:10,
          autoSetFitView:false,
          topAdcodes:[430000],
          getPosition: function(item) {
            if (!item) {
              return null;
            }
            var parts = item.split(",");
            //返回经纬度
            return [parseFloat(parts[0]), parseFloat(parts[1])];
          }
        });
        _this.$http.get("https://a.amap.com/amap-ui/static/data/10w.txt").then(res=>{
             var data = res.data.split('\n');
            distCluster.setData(data)
        })
      });
    },

    // 绘制载入区域
    loadnode(DistrictExplorer) {
      var _this = this;
      var districtExplorer = new DistrictExplorer({
        map: _this.map //关联的地图实例
      });
      var adcode = 430000; //全国的区划编码
      districtExplorer.loadAreaNode(adcode, function(error, areaNode) {
        if (error) {
          console.error(error);
          return;
        }
        //绘制载入的区划节点
        _this.renderAreaNode(districtExplorer, areaNode);
      });
    },
    // 绘制载入区域加上颜色
    renderAreaNode(districtExplorer, areaNode) {
      var _this = this;
      //清除已有的绘制内容
      districtExplorer.clearFeaturePolygons();

      //just some colors
      var colors = [
        "#3366cc",
        "#dc3912",
        "#ff9900",
        "#109618",
        "#990099",
        "#0099c6",
        "#dd4477",
        "#66aa00"
      ];

      //绘制子级区划
      districtExplorer.renderSubFeatures(areaNode, function(feature, i) {
        var fillColor = colors[i % colors.length];
        var strokeColor = colors[colors.length - 1 - (i % colors.length)];
        return {
          cursor: "default",
          bubble: true,
          strokeColor: strokeColor, //线颜色
          strokeOpacity: 1, //线透明度
          strokeWeight: 1, //线宽
          fillColor: fillColor, //填充色
          fillOpacity: 0.35 //填充透明度
        };
      });

      //绘制父级区划，仅用黑色描边
      districtExplorer.renderParentFeature(areaNode, {
        cursor: "default",
        bubble: true,
        strokeColor: "black", //线颜色
        fillColor: null,
        strokeWeight: 3 //线宽
      });

      //更新地图视野以适合区划面
      _this.map.setFitView(districtExplorer.getAllFeaturePolygons());
    }
  },

  async created() {
    // 已载入高德地图API，则直接初始化地图
    if (window.AMap && window.AMapUI) {
      this.initMap();
      this.clustering();
      // 未载入高德地图API，则先载入API再初始化
    } else {
      await remoteLoad(`http://webapi.amap.com/maps?v=1.3&key=${MapKey}`);
      await remoteLoad("http://webapi.amap.com/ui/1.0/main.js");
      this.initMap();
      this.clustering();
    }
  }
};
</script>

<style scoped>
.map {
  width: 1200px;
  height: 100vh;
}
</style>
