<template>
  <div style="width: 100%; height: 100%;">
    <div id="cesiumContainer"></div>

    <div id="popupContainer">
      <div id="popupTitle" style="font-weight: bold; margin-bottom: 5px;">信息窗口</div>
      <div id="popupContent">这是一个自定义的弹窗示例。</div>
      <button id="closePopupBtn" style="margin-top: 10px;">关闭</button>
    </div>

  </div>
</template>

<script>
import * as echarts from 'echarts';
import iconPath from '../assets/logo.png';

export default {
  data() {
    return {
      viewer: "",
      scene: "",
      handler: "",
    }
  },
  mounted() {
    const vm = this;
    setTimeout(function () {

      console.log(echarts)
      vm.viewer = new Cesium.Viewer('cesiumContainer', {
        geocoder: false,                //是否显示地名查找控件
        sceneModePicker: false,         //是否显示投影方式控件
        navigationHelpButton: false,    //是否显示帮助信息控件
        baseLayerPicker: false,         //是否显示图层选择控件
        homeButton: false,              //是否显示Home按钮
        fullscreenButton: false,        //是否显示全屏按钮
        animation: false, //左下角的动画控件的显示
        shouldAnimate: false, //控制模型动画
        timeline: false, //底部的时间轴
        selectionIndicator: true,
        infoBox: true,
        // 使用中国在线地图服务作为底图
        imageryProvider: new Cesium.ArcGisMapServerImageryProvider({
          url: "http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineStreetPurplishBlue/MapServer"
        })
      });
      // 定义目标位置
      var destination = Cesium.Cartesian3.fromDegrees(116.397408, 39.909165, 5000);

      // 获取当前摄像机位置
      var currentPosition = vm.viewer.camera.position;

      // 计算当前摄像机位置和目标位置之间的距离
      var distance = Cesium.Cartesian3.distance(currentPosition, destination);
      console.log('Distance:', distance, 'meters');

      // 设置飞行方向
      var heading = Cesium.Math.toRadians(0); // 头部朝向，以弧度为单位
      var pitch = Cesium.Math.toRadians(-100); // 俯仰角度，以弧度为单位
      var roll = 0; // 滚动角度

      // 缓慢飞行到指定位置并控制方向
      vm.viewer.camera.flyTo({
        destination: destination,
        orientation: {
          heading: heading,
          pitch: pitch,
          roll: roll
        },
        duration: 5 // 以秒为单位的飞行时间，时间越长速度越慢
      });




      //去cesium logo水印 或 css
      // vm.viewer.cesiumWidget.creditContainer.style.display = "none";
      //创建场景
      vm.scene = vm.viewer.scene;
      if (!vm.scene.pickPositionSupported) {
        window.alert("此浏览器不支持拾取位置！")
      }
      vm.handler = new Cesium.ScreenSpaceEventHandler(vm.scene.canvas);


      vm.handler.setInputAction(function (event) {
        // 获取点击位置的地理坐标
        var cartesian = vm.viewer.scene.pickPosition(event.position);

        // 将地理坐标转换为经纬度
        if (Cesium.defined(cartesian)) {
          var cartographic = Cesium.Cartographic.fromCartesian(cartesian);
          var longitude = Cesium.Math.toDegrees(cartographic.longitude);
          var latitude = Cesium.Math.toDegrees(cartographic.latitude);

          // 在这里使用经纬度进行操作，例如更新描述
          console.log("点击位置的经度：" + longitude);
          console.log("点击位置的纬度：" + latitude);
        }
      }, Cesium.ScreenSpaceEventType.LEFT_CLICK)


      // vm.viewer.entities.add({
      //   name: '添加点',
      //   position: new Cesium.Cartesian3.fromDegrees(116.397408, 39.909165, 0),
      //   point: {
      //     color: new Cesium.Color(1.0, 0.0, 0.0, 1.0),  //颜色
      //     pixelSize: 10,  //大小
      //     outlineColor: Cesium.Color.YELLOW, //轮廓颜色
      //     outlineWidth: 2
      //   },

      // });

      // 创建Billboard
      var billboard = vm.viewer.entities.add({
        name: '自定义图标',
        position: Cesium.Cartesian3.fromDegrees(116.393696, 39.908798),
        billboard: {
          image: iconPath,
          width: 30,
          height: 30
        },
        label: { // 文字标签
          text: '12312312',
          font: "400 30px MicroSoft YaHei", // 15pt monospace
          scale: 0.6,
          fillColor: "#e4393c",
          show: false,
          outlineWidth: 3,
          outlineColor: "#000",
          pixelOffset: Cesium.Cartesian2(0, -60) // 偏移量
        }
      });
      billboard.description = `内容文字`



      // 添加初始点实体
      var radarEffectEntity = vm.viewer.entities.add({
        name: '雷达扩散光效',
        position: Cesium.Cartesian3.fromDegrees(116.397408, 39.909165, 0),
        point: {
          color: new Cesium.Color(1.0, 0.0, 0.0, 1.0), // 初始颜色
          pixelSize: 5, // 初始大小
          outlineColor: Cesium.Color.YELLOW,
          outlineWidth: 2
        }
      });

      // 动画控制参数
      var maxRadius = 20; // 最大半径
      var expansionRate = 0.2; // 扩散速度
      var initialOpacity = 1.0; // 初始透明度
      var fadeRate = 0.01; // 透明度变化速度

      // 创建动画
      function animateRadarEffect() {
        var currentSize = radarEffectEntity.point.pixelSize.getValue();
        var currentOpacity = radarEffectEntity.point.color.getValue().alpha;

        if (currentSize >= maxRadius) {
          // 重置大小和透明度
          currentSize = 5;
          currentOpacity = initialOpacity;
        } else {
          // 扩散并渐隐
          currentSize += expansionRate;
          currentOpacity -= fadeRate;
        }

        // 更新点的大小和透明度
        radarEffectEntity.point.pixelSize = currentSize;
        radarEffectEntity.point.color = new Cesium.Color(1.0, 0.0, 0.0, currentOpacity);

        // 继续动画
        requestAnimationFrame(animateRadarEffect);
      }

      // 开始动画
      requestAnimationFrame(animateRadarEffect);

      // 添加点击事件
      radarEffectEntity.description = `<table border="1">
                                          <tr>
                                          <th>属性</th>
                                          <th>值</th>
                                          </tr>
                                          <tr>
                                          <td>名称</td>
                                          <td>entity.name</td>
                                          </tr>
                                          <tr>
                                          <td>经度</td>
                                          <td>longitude</td>
                                          </tr>
                                          <tr>
                                          <td>纬度</td>
                                          <td>latitude</td>
                                          </tr>
                                        </table>`

      radarEffectEntity.isPickable = true; // 设置实体可被拾取

      radarEffectEntity.onmouseup = function (e) {
        console.log('点击了雷达扩散光效实体');
      };



      vm.viewer.entities.add({
        name: '添加文字',
        position: new Cesium.Cartesian3.fromDegrees(116.389747, 39.904963, 20),
        label: {
          text: '这是我的标记点',
          font: '24px',
          fillColor: Cesium.Color.YELLOW
        }
      })


      // 倾斜视图 鼠标左键平移
      vm.viewer.scene.screenSpaceCameraController.tiltEventTypes = [Cesium.CameraEventType.RIGHT_DRAG]

      // 缩放设置 重新设置缩放成员 
      vm.viewer.scene.screenSpaceCameraController.zoomEventTypes = [Cesium.CameraEventType.MIDDLE_DRAG, Cesium.CameraEventType.WHEEL, Cesium.CameraEventType.PINCH];

      // 偏斜平移
      //vm.viewer.scene.screenSpaceCameraController.lookEventTypes = [ Cesium.CameraEventType.RIGHT_DRAG]

      // 平移 添加鼠标右键  鼠标右键旋转
      vm.viewer.scene.screenSpaceCameraController.rotateEventTypes = [Cesium.CameraEventType.LEFT_DRAG];







      var popupContainer = document.getElementById('popupContainer');
      var closePopupBtn = document.getElementById('closePopupBtn');

      closePopupBtn.onclick = function () {
        popupContainer.style.display = 'none';
      };


      vm.viewer.screenSpaceEventHandler.setInputAction(function onClick(click) {
        var pickedObject = vm.viewer.scene.pick(click.position);
        if (Cesium.defined(pickedObject)) {
          var cartesian = vm.viewer.scene.pickPosition(click.position);
          if (Cesium.defined(cartesian)) {
            var cartographic = Cesium.Cartographic.fromCartesian(cartesian);
            var longitude = Cesium.Math.toDegrees(cartographic.longitude).toFixed(6);
            var latitude = Cesium.Math.toDegrees(cartographic.latitude).toFixed(6);
            var height = cartographic.height.toFixed(2);

            popupContainer.style.left = click.position.x + 'px';
            popupContainer.style.top = click.position.y + 'px';

            document.getElementById('popupContent').innerHTML = '经度：' + longitude + '<br>纬度：' + latitude + '<br>高度：' + height;

            popupContainer.style.display = 'block';
          }
        } else {
          popupContainer.style.display = 'none';
        }
      }, Cesium.ScreenSpaceEventType.LEFT_CLICK);





      // // 监听地图拖动事件
      // vm.viewer.scene.screenSpaceCameraController.dragEnd.addEventListener(updatePopupPosition());

      // // 更新弹出窗口位置的函数
      // function updatePopupPosition() {
      //   var cameraPosition = vm.viewer.camera.position;
      //   var screenPosition = vm.viewer.scene.cartesianToCanvasCoordinates(cameraPosition);
      //   if (popupContainer.style.display === 'block') {
      //     if (!isNaN(screenPosition.x) && !isNaN(screenPosition.y)) {
      //       popupContainer.style.left = screenPosition.x + 'px';
      //       popupContainer.style.top = screenPosition.y + 'px';
      //     }
      //   }
      // }



      // 核心代码
      const div = document.createElement("div");
      div.innerHTML = "弹窗里需要填充点内容，支持html字符串";
      vm.viewer.container.append(div);

      vm.viewer.scene.screenSpaceCameraController.dragEnd.addEventListener(() => {
        let windowCoord = Cesium.SceneTransforms.wgs84ToWindowCoordinates(
          vm.scene,
          Cesium.Cartesian3.fromDegrees(116.389747, 39.904963, 0)
        );
        let x = windowCoord.x - div.offsetWidth / 2;
        let y = windowCoord.y - div.offsetHeight;
        div.style.cssText = `
        position:absolute;
        left:0;
        top:0;
        height:100px;
        width:100px;
        background:rgba(0,0,0,0.7);
        color:white;
        transform:translate3d(${Math.round(x)}px,${Math.round(y)}px, 0);
    `;
      });



    }, 2000)
  }
}
</script>

<style>
#cesiumContainer {
  height: 100%;
  width: 100%;
}

#popupContainer {
  display: none;
  position: absolute;
  background-color: #fff;
  padding: 10px;
  border: 1px solid #ccc;
  z-index: 10001;
}
</style>