<template>
    <div style="width: 100%; height: 100%;">
        <div id="cesiumContainer"></div>


        <div class="state" id="stateShow"
            style="position: absolute; top: 0; left: 0; z-index: 99999; background-color: red;">
            <div class="box">
                <div class="box-wrap">
                    <div class="close" @click="closeClick">X</div>
                    <div class="area">
                        <div class="area-title fontColor" id="title"></div>
                    </div>
                    asdasdas
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import iconPath from '../assets/logo.png';

export default {
    data() {
        return {
            cesiumLoaded: false,

        }
    },
    mounted() {
        const vm = this;
        vm.checkCesiumLoaded()
    },
    methods: {
        checkCesiumLoaded() {
            // 设置定时器，每隔一段时间检查Cesium对象是否存在
            const intervalId = setInterval(() => {
                if (typeof Cesium !== 'undefined') {
                    // Cesium对象已加载
                    clearInterval(intervalId); // 停止定时器
                    this.cesiumLoaded = true;
                    // 执行你的操作，例如初始化Cesium地图
                    this.initializeCesium();
                }
            }, 1000); // 每隔1秒检查一次
        },
        // 初始化地球
        initializeCesium() {
            const vm = this;
            // 请自己去cesium官网注册申请一个token替换
            Cesium.Ion.defaultAccessToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIxNmJkYzFiZi1jMGE2LTQ2YmYtYTAyZS1jOTNhODEwZTYzZjYiLCJpZCI6MjE2NTk4LCJpYXQiOjE3MTYyNTc5OTJ9.AbNDyzzy3zB6vFXXXnJ9HwVhNvBSbKAnhFRo3k9D3hE";

            vm.viewer = new Cesium.Viewer("cesiumContainer", {
                // geocoder: false,                //是否显示地名查找控件
                // sceneModePicker: false,         //是否显示投影方式控件
                // navigationHelpButton: false,    //是否显示帮助信息控件
                // baseLayerPicker: false,         //是否显示图层选择控件
                // homeButton: false,              //是否显示Home按钮
                // fullscreenButton: false,        //是否显示全屏按钮
                // animation: false, //左下角的动画控件的显示
                // shouldAnimate: false, //控制模型动画
                // timeline: false, //底部的时间轴
                // selectionIndicator: true,
                // infoBox: true,
                // 使用中国在线地图服务作为底图
                // imageryProvider: new Cesium.ArcGisMapServerImageryProvider({
                //   url: "http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineStreetPurplishBlue/MapServer"
                // })
            });
            // 定义目标位置
            var destination = Cesium.Cartesian3.fromDegrees(116.397408, 39.8926, 5000);

            // 缓慢飞行到指定位置并控制方向
            vm.viewer.camera.flyTo({
                destination: destination,
                duration: 2 // 以秒为单位的飞行时间，时间越长速度越慢
            });


            const poin = [
                { id: '12321321', name: "颍红测试点", type: "固定枪机", state: "在线", position: { x: 116.4568, y: 39.8926 }, text: '1' },
                { id: '43244324', name: "解放修理厂门口", type: "固定枪机", state: "在线", position: { x: 116.4568, y: 39.8944 }, text: '2' },
                { id: '43764324', name: "新华路加油站", type: "固定枪机", state: "在线", position: { x: 116.4566, y: 39.8923 }, text: '3' },
                { id: '437543345', name: "康佳大药房", type: "固定枪机", state: "在线", position: { x: 116.4513, y: 39.8923 }, text: '4' }
            ]
            poin.forEach(res => {
                vm.viewer.entities.add({
                    id: res.id,
                    name: res.name,
                    position: Cesium.Cartesian3.fromDegrees(res.position.x, res.position.y),
                    billboard: {
                        image: new Cesium.PinBuilder().fromText(res.text, Cesium.Color.ROYALBLUE, 48).toDataURL(),
                        verticalOrigin: Cesium.VerticalOrigin.BOTTOM
                    },

                    monitoItems: {
                        data: res
                    },
                })
            });



            // 添加弹窗
            var handler = new Cesium.ScreenSpaceEventHandler(vm.viewer.scene.canvas);

            // 获取弹窗元素
            var stateShowElement = document.getElementById('stateShow');
            var postRenderListener = null; // 保存 postRender 监听器的引用

            handler.setInputAction(function (movement) {
                var picked = vm.viewer.scene.pick(movement.position);
                if (Cesium.defined(picked)) {
                    console.log(232323232323);
                    let data = picked.id.monitoItems.data;
                    let gisPosition = Cesium.Cartesian3.fromDegrees(data.position.x, data.position.y, 0);

                    // 在添加新的 postRender 事件监听器之前，先移除之前的监听器
                    if (postRenderListener) {
                        vm.viewer.scene.postRender.removeEventListener(postRenderListener);
                    }

                    postRenderListener = vm.viewer.scene.postRender.addEventListener(() => {
                        console.log('2222222222222222222222222222222222222222222222222222222-099-=0')
                        var windowPosition = Cesium.SceneTransforms.wgs84ToWindowCoordinates(vm.viewer.scene, gisPosition);
                        stateShowElement.style.left = windowPosition.x + 100 + 'px';
                        stateShowElement.style.top = windowPosition.y + 100 + 'px';

                        const camerPosition = vm.viewer.camera.position;
                        let height = vm.viewer.scene.globe.ellipsoid.cartesianToCartographic(camerPosition).height;
                        height += vm.viewer.scene.globe.ellipsoid.maximumRadius;
                        if (!(Cesium.Cartesian3.distance(camerPosition, gisPosition) > height) && vm.viewer.camera.positionCartographic.height < 50000000) {
                            stateShowElement.style.display = 'block'; // 显示弹窗
                        } else {
                            stateShowElement.style.display = 'none'; // 隐藏弹窗
                        }
                    });
                } else {
                    console.log(2323232);
                    stateShowElement.style.display = 'none'; // 隐藏弹窗
                }
            }, Cesium.ScreenSpaceEventType.LEFT_CLICK);




        },

        closeClick() {
            console.log('222222222')
            document.getElementById('stateShow').style.display = "none"
        }



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