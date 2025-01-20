<template>
    <div style="width: 100%; height: 100%;">
        <div id="cesiumContainer"></div>


        <!-- 鼠标点击窗口 -->
        <div v-show="clickPopupShow">
            <div class="clickModal-content" id="clickPopup">
                <span class="close" @click="closeModal">&times;</span>
                <div id="clickPopup-title">我是标题</div>
                <p id="clickPopup-p">这是一个漂亮的弹窗，带有关闭按钮。</p>
            </div>
        </div>


        <!-- 鼠标悬浮窗口 -->
        <div v-show="hoverPopupShow">
            <div class="hoverModal-content" id="hoverPopup">
                <p id="hoverPopup-p">我悬浮窗口哦</p>
            </div>
        </div>
    </div>
</template>

<script>
import iconPath from '../assets/logo.png';
// 绿色水滴   http://mars3d.cn/project/vue/img/marker/mark-green.png
// 红色公司   http://mars3d.cn/project/vue/img/marker/mark-red.png
// 蓝色标志   http://mars3d.cn/project/vue/img/marker/mark-blue.png
export default {
    data() {
        return {
            cesiumLoaded: false,

            clickPopupShow: false, // 控制弹窗显示与隐藏

            // 悬浮弹窗
            hoverPopupShow: false,

            // 记录标记为
            lastClickedEntityPosition: null,
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
                geocoder: false,                //是否显示地名查找控件
                geocoder: false,                // 是否显示地名查找控件
                sceneModePicker: false,         // 是否显示投影方式控件
                navigationHelpButton: false,    // 是否显示帮助信息控件
                baseLayerPicker: false,         // 是否显示图层选择控件
                homeButton: false,              // 是否显示Home按钮
                fullscreenButton: false,        // 是否显示全屏按钮
                animation: false,               // 是否显示动画控件
                shouldAnimate: false,           // 控制模型动画
                timeline: false,                // 是否显示时间轴
                selectionIndicator: false,      // 是否显示选中指示器
                infoBox: false,                 // 是否显示信息框
                // 使用中国在线地图服务作为底图
                // imageryProvider: new Cesium.ArcGisMapServerImageryProvider({
                //     url: "http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineStreetPurplishBlue/MapServer"
                // })
            });

            // // 倾斜视图 鼠标左键平移
            // vm.viewer.scene.screenSpaceCameraController.tiltEventTypes = [Cesium.CameraEventType.RIGHT_DRAG]

            // // 缩放设置 重新设置缩放成员 
            // vm.viewer.scene.screenSpaceCameraController.zoomEventTypes = [Cesium.CameraEventType.MIDDLE_DRAG, Cesium.CameraEventType.WHEEL, Cesium.CameraEventType.PINCH];

            // // 偏斜平移
            // //vm.viewer.scene.screenSpaceCameraController.lookEventTypes = [ Cesium.CameraEventType.RIGHT_DRAG]

            // // 平移 添加鼠标右键  鼠标右键旋转
            // vm.viewer.scene.screenSpaceCameraController.rotateEventTypes = [Cesium.CameraEventType.LEFT_DRAG];


            //隐藏logo
            vm.viewer._cesiumWidget._creditContainer.style.display = "none";

            // // 定义目标位置
            var destination = Cesium.Cartesian3.fromDegrees(117.085053, 39.102347, 10000);

            // 缓慢飞行到指定位置并控制方向
            vm.viewer.camera.flyTo({
                destination: destination,
                duration: 5,// 以秒为单位的飞行时间，时间越长速度越慢
                complete: function () {
                    vm.startRotation();
                }
            });

            // 创建带有图片的点
            vm.viewer.entities.add({
                id: '000000001',
                name: '公司',
                position: Cesium.Cartesian3.fromDegrees(117.085053, 39.102347, 0),
                billboard: {
                    image: 'http://mars3d.cn/project/vue/img/marker/mark-red.png', // 图片路径
                    width: 25,  // 图片宽度（像素）
                    height: 34, // 图片高度（像素）
                    verticalOrigin: Cesium.VerticalOrigin.BOTTOM // 垂直方向上的对齐方式
                },
                monitoItems: {
                    data: {
                        "name": "公司位置"
                    }
                },
            });


            // 生成随机数函数
            function getRandom(min, max) {
                return Math.random() * (max - min) + min;
            }


            // 生成100个点位的经纬度数据
            const points = [];
            for (let i = 1; i <= 100; i++) {
                points.push({
                    id: i.toString(),
                    name: `点位${i}`,
                    type: `类型${i}`,
                    state: `状态${i}`,
                    text: i.toString(),
                    position: {
                        longitude: getRandom(117.085053, 118),
                        latitude: getRandom(39.102347, 40)
                    }
                });
            }



            // 创建一个数组来保存所有的实体
            const entities = [];


            // 广告牌
            // points.forEach(res => {
            //     const entity = vm.viewer.entities.add({
            //         id: res.id,
            //         name: res.name,
            //         position: Cesium.Cartesian3.fromDegrees(res.position.longitude, res.position.latitude),
            //         billboard: {
            //             image: new Cesium.PinBuilder().fromText(res.text, Cesium.Color.ROYALBLUE, 48).toDataURL(),
            //             verticalOrigin: Cesium.VerticalOrigin.BOTTOM
            //         },
            //         monitoItems: {
            //             data: res
            //         },
            //     });
            //     // 将实体添加到数组中
            //     entities.push(entity);
            // });

            // 点位信息
            points.forEach(res => {
                const entity = vm.viewer.entities.add({
                    id: res.id,
                    name: res.name,
                    position: Cesium.Cartesian3.fromDegrees(res.position.longitude, res.position.latitude),
                    billboard: {
                        image: 'http://mars3d.cn/project/vue/img/marker/mark-green.png', // 图片路径
                        width: 25,  // 图片宽度（像素）
                        height: 34, // 图片高度（像素）
                        verticalOrigin: Cesium.VerticalOrigin.BOTTOM // 垂直方向上的对齐方式
                    },
                    // point: {
                    //     pixelSize: 10, // 点的大小（像素）
                    //     color: Cesium.Color.ROYALBLUE, // 点的颜色
                    //     outlineColor: Cesium.Color.WHITE, // 点的轮廓颜色
                    //     outlineWidth: 1 // 点的轮廓宽度
                    // },
                    monitoItems: {
                        data: res
                    }
                });
                // 将实体添加到数组中
                entities.push(entity);
            });



            // 监听 Cesium 实体点击事件
            const handler = new Cesium.ScreenSpaceEventHandler(vm.viewer.scene.canvas);
            handler.setInputAction(vm.onEntityClick, Cesium.ScreenSpaceEventType.LEFT_CLICK);

            // 鼠标悬浮
            handler.setInputAction(vm.onMovement, Cesium.ScreenSpaceEventType.MOUSE_MOVE);



            // 监听地图视图改变事件
            vm.viewer.scene.postRender.addEventListener(vm.onMapViewChange, vm);

        },

        // 实体点击事件处理函数
        onEntityClick(movement) {
            const vm = this;
            // 获取点击位置的实体对象
            var pickedObject = vm.viewer.scene.pick(movement.position);
            console.log(pickedObject)
            // 如果点击到了一个具有 monitoItems 属性的实体对象
            if (pickedObject && pickedObject.id && pickedObject.id.monitoItems) {
                // 记录点击的位置，保存为当前时间的 Cartesian 坐标
                vm.lastClickedEntityPosition = pickedObject.id.position.getValue(Cesium.JulianDate.now());

                vm.showModal(); // 点击后显示信息框

                // 获取实体的位置信息（Cartesian 坐标）
                const entityPosition = pickedObject.id.position.getValue(Cesium.JulianDate.now());

                // 将 Cartesian 坐标转换为窗口坐标
                const windowPosition = Cesium.SceneTransforms.wgs84ToWindowCoordinates(vm.viewer.scene, entityPosition);

                // 获取当前相机高度（Cartographic 坐标中的高度）
                const currentHeight = vm.viewer.camera.positionCartographic.height;
                // 将实体位置从 Cartesian 坐标转换为 Cartographic 坐标（经度、纬度、高度）
                const cartographicPosition = Cesium.Cartographic.fromCartesian(entityPosition);
                console.log(cartographicPosition); // 输出 Cartographic 坐标

                // 将相机飞行到实体位置，并保持当前高度
                vm.viewer.camera.flyTo({
                    // 目标位置为实体的经度、纬度和当前高度
                    destination: Cesium.Cartesian3.fromRadians(cartographicPosition.longitude, cartographicPosition.latitude, currentHeight),
                    duration: 2 // 飞行时间（秒）
                });

                // 获取弹窗元素
                const modalElement = document.getElementById('clickPopup');
                const modal_P_Element = document.getElementById('clickPopup-p');

                // 将弹窗定位到实体位置的窗口坐标
                modalElement.style.left = windowPosition.x + 'px';
                modalElement.style.top = windowPosition.y + 'px';

                // 在弹窗中显示实体的详细信息
                modal_P_Element.innerHTML = `${JSON.stringify(pickedObject.id.monitoItems.data)}`;
            } else {
                // 如果没有点击到目标实体，则关闭弹窗
                vm.closeModal();
            }
        },


        // 显示弹窗
        showModal() {
            this.clickPopupShow = true;
        },
        // 关闭弹窗
        closeModal() {
            this.clickPopupShow = false;
        },


        // 地图视图改变事件处理函数
        onMapViewChange() {
            const vm = this;

            // 如果弹窗正在显示且存在点击的位置，则更新弹窗位置为点击的位置
            if (vm.clickPopupShow && vm.lastClickedEntityPosition) {
                const windowPosition = Cesium.SceneTransforms.wgs84ToWindowCoordinates(vm.viewer.scene, vm.lastClickedEntityPosition);

                const modalElement = document.getElementById('clickPopup');
                // 将弹窗定位到实体位置的窗口坐标
                modalElement.style.left = windowPosition.x + 'px';
                modalElement.style.top = windowPosition.y + 'px';
            }
        },


        // 鼠标悬浮
        onMovement(movement) {
            const vm = this;
            // 获取点击位置的实体对象
            var pickedObject = vm.viewer.scene.pick(movement.endPosition);

            // 如果点击到了一个具有 monitoItems 属性的实体对象
            if (pickedObject && pickedObject.id && pickedObject.id.monitoItems) {

                // 获取实体的位置信息（Cartesian 坐标）
                const entityPosition = pickedObject.id.position.getValue(Cesium.JulianDate.now());
                // 将 Cartesian 坐标转换为窗口坐标
                const windowPosition = Cesium.SceneTransforms.wgs84ToWindowCoordinates(vm.viewer.scene, entityPosition);

                // 获取弹窗元素
                const modalElement = document.getElementById('hoverPopup');
                const modal_P_Element = document.getElementById('hoverPopup-p');

                // 将弹窗定位到实体位置的窗口坐标
                modalElement.style.left = windowPosition.x + 'px';
                modalElement.style.top = windowPosition.y + 'px';

                // 在弹窗中显示实体的详细信息
                modal_P_Element.innerHTML = `${JSON.stringify(pickedObject.id.monitoItems.data.name)}`;


                // 判断clickPopup状态决定是否显示hoverPopup
                if (!vm.clickPopupShow) {
                }
                vm.clickPopupShow ? vm.hoverPopupShow = false : vm.hoverPopupShow = true;
            } else {
                vm.hoverPopupShow = false;
            }
        },

        // 开始旋转的函数
        startRotation() {
            const vm = this;

            // 给定的坐标
            var position = Cesium.Cartesian3.fromDegrees(117.085653, 39.102947);

            // 初始朝向角度
            var heading = 0;
            // 初始俯仰角度
            var pitch = 90;
            // 初始距离
            var distance = 10000;
            // 初始偏移量
            var offset = new Cesium.HeadingPitchRange(Cesium.Math.toRadians(heading), -Cesium.Math.toRadians(pitch), distance);

            vm.rotate = function () {
                heading += 0.1;
                if (pitch >= 30) {
                    pitch -= 0.02; // 减少俯仰角度
                }
                if (distance < 100000) {
                    distance += 30; // 减少距离
                }
                offset = new Cesium.HeadingPitchRange(Cesium.Math.toRadians(heading), -Cesium.Math.toRadians(pitch), distance);

                // 锁定相机
                vm.viewer.camera.lookAt(position, offset);
            }
            // 禁用相机控制器的输入事件
            vm.viewer.scene.screenSpaceCameraController.enableInputs = false;

            // 修改旋转的方法 
            vm.viewer.zoomTo(position, offset).then(function () {
                vm.viewer.clock.onTick.addEventListener(rotate);
            });

            setTimeout(function () {
                //====取消旋转====== 
                vm.viewer.clock.onTick.removeEventListener(rotate);

                // 解锁相机
                vm.viewer.camera.lookAtTransform(Cesium.Matrix4.IDENTITY);
                console.log(vm.viewer.clock.onTick)

            }, 100000);
        },






    }
}
</script>

<style>
#cesiumContainer {
    height: 100%;
    width: 100%;
}


/* 弹窗内容容器 */
.clickModal-content {
    width: 300px;
    position: fixed;
    top: 0;
    left: 0;
    background-color: #fefefe;
    padding: 0 10px;
    border-radius: 5px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
    /* position: relative; */

    /* X 轴左移一半，Y 轴向上偏移 100% */
    transform: translate(-50%, -125%);
}

.clickModal-content::after {
    content: "";
    position: absolute;
    bottom: -8px;
    /* 调整这个值来定位三角形 */
    left: 50%;
    transform: translateX(-50%);
    width: 0;
    height: 0;
    border-left: 15px solid transparent;
    border-right: 15px solid transparent;
    border-top: 14px solid #ffffff;
}


.clickModal-content p {
    margin-top: 30px;
    word-wrap: break-word;
}


/* 关闭按钮样式 */
.close {
    position: absolute;
    top: 5px;
    right: 10px;
    cursor: pointer;
}

/* 关闭按钮的样式 */
.close:hover {
    color: #aaa;
}

/* 鼠标悬浮窗口 */
.hoverModal-content {
    width: 300px;
    height: 40px;
    line-height: 40px;
    padding: 0 10px;
    position: fixed;
    top: 0;
    left: 50%;
    color: #ffffff;
    background-color: rgba(0, 0, 0, 0.8);
    border-radius: 5px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
    /* X 轴左移一半，Y 轴向上偏移 100% */
    transform: translate(-50%, -220%);
}

.hoverModal-content::after {
    content: "";
    position: absolute;
    bottom: -8px;
    /* 调整这个值来定位三角形 */
    left: 50%;
    transform: translateX(-50%);
    width: 0;
    height: 0;
    border-left: 15px solid transparent;
    border-right: 15px solid transparent;
    border-top: 14px solid rgba(0, 0, 0, 0.8);
}


.hoverModal-content p {
    padding: 0;
    margin: 0;
}
</style>