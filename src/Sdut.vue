<template>
    <div id="main">
        <div id="toolbar">
            山东理工大学 任务id:{{ taskstartId }}&nbsp小图:<input type="checkbox" v-model="smailMap" /><input type="button" @click="sync_db_filter" value="非bad值同步到远程数据库"/><br>
            <input type="number" v-model="startId" placeholder="开始id"/>
            <input type="number" v-model="taskSize" placeholder="任务数量"/>
            自动获取:<input type="checkbox" v-model="AutoSwitch" />
            <input type="button" value="更新地图" @click="getmapList" />
            <input type="button" value="提交任务" @click="upscorelist" />
            <div :class="getBackgroundColor">
                <p class="getBackgroundColorp">全局,优先级低于地图设置</p>
                <input type="radio" value="great" v-model="mapscoretype">great
                <input type="radio" value="good" v-model="mapscoretype">good
                <input type="radio" value="normal" v-model="mapscoretype">normal
                <input type="radio" value="bad" v-model="mapscoretype">bad
            </div>
        </div>
        <div id="map">
            <div v-for="(mapList_obj, mapList_index) in mapList" class="map_block" :class="setmapborder(mapList_index)" :width="canvasWidth" :height="canvasHeight" :key="mapList_index">
                <canvas :id="'canvas' + mapList_obj.id" :width="getCanvasWidth(mapList_obj.data_campus)" :height="getCanvasHeight(mapList_obj.data_campus)" class="mapcanvas"></canvas>
                <div class="colormap">
                    <div class="colormap_sel" :class="`colormap_btn_${mapList_obj.mst}`"></div>
                    <div class="colormap_btn">
                        <div class="colormap_btn_great" @click="setmapscoretype('great', mapList_index)">great</div>
                        <div class="colormap_btn_good" @click="setmapscoretype('good', mapList_index)">good</div>
                        <div class="colormap_btn_normal" @click="setmapscoretype('normal', mapList_index)">normal</div>
                        <div class="colormap_btn_bad" @click="setmapscoretype('bad', mapList_index)">bad</div>
                    </div>
                </div>
                <img :src="getMapImg(mapList_obj.data_campus)" alt="地图" class="map_img">
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'
axios.defaults.baseURL = import.meta.env.VITE_wkyd_Debugger_nodejs_SERVICE_URL
export default {
    data() {
        return {
            // 用户任务相关
            // mapList: [{ id: 1, markList: [{ isStartPosition: true, x: 287, y: 360 }, { x: 288, y: 361 }, { x: 9, y: 2 }, { isStartPosition: true, x: 10, y: 10 }, { x: 18, y: 36 }, { x: 99, y: 62 }] }], // 地图列表
            mapList_origin: [], // 地图列表,原始
            mapList: [], // 地图列表,转换后
            scorelist: { great: [], good: [], normal: [], bad: [] }, // 评分列表
            mapscoretype: "normal", // 评分类型gread, good, normal, bad
            taskstartId: 1, // 数据库任务开始id
            startId: 1, // 用户可设置的任务开始id
            taskSize: 10, // 任务数量
            AutoSwitch: true, // 自动获取下一组任务
            smailMap: false, // 小图模式
            // 画板参数相关
            canvasWidth_A: 625, //画布宽度 主校区
            canvasHeight_A: 500,//画布高度 主校区
            canvasWidth_B: 350, //画布宽度 东校区
            canvasHeight_B: 500,//画布高度 东校区
            // SX → EX + x118
            // SY → EY - y36
            OriginSX_A: 117.991931,//原点经度坐标 主校区
            OriginSY_A: 36.8167640,//原点纬度坐标 主校区
            OriginEX_A: 118.011287,//对点经度坐标 主校区
            OriginEY_A: 36.8043940,//对点纬度坐标 主校区
            OriginSX_B: 118.036910,//原点经度坐标 东校区
            OriginSY_B: 36.8141480,//原点纬度坐标 东校区
            OriginEX_B: 118.042242,//对点经度坐标 东校区
            OriginEY_B: 36.8078970,//对点纬度坐标 东校区
            // 图片
            sdutmap_A: "/sdutmap_A.png", // 主校区地图
            sdutmap_B: "/sdutmap_B.png", // 东校区地图
        }
    },
    computed: {
        smailMap: {
            set(value) {
                if (value) {
                    this.canvasWidth_A/=2;
                    this.canvasHeight_A/=2;
                    this.canvasWidth_B/=2;
                    this.canvasHeight_B/=2;
                } else {
                    this.canvasWidth_A*=2;
                    this.canvasHeight_A*=2;
                    this.canvasWidth_B*=2;
                    this.canvasHeight_B*=2;
                }
            }
        },
        // 画板尺寸修改
        canvasDimensions() {
            return {
                widthA: this.canvasWidth_A,
                heightA: this.canvasHeight_A,
                widthB: this.canvasWidth_B,
                heightB: this.canvasHeight_B
            };
        },
        // 计算属性, 根据评分类型返回背景颜色
        getBackgroundColor() {
            switch (this.mapscoretype) {
                case 'great':
                    return 'bg-great';
                case 'good':
                    return 'bg-good';
                case 'normal':
                    return 'bg-normal';
                case 'bad':
                    return 'bg-bad';
                default:
                    return '';
            }
        },
        // 计算属性, 地图相关
        OriginW_A() {
            return this.OriginEX_A - this.OriginSX_A; //经度宽度
        },
        OriginH_A() {
            return this.OriginEY_A - this.OriginSY_A; //纬度高度
        },
        RatioX_A() {
            return this.OriginW_A / this.canvasWidth_A; //经度比例
        },
        RatioY_A() {
            return this.OriginH_A / this.canvasHeight_A; //纬度比例
        },
        OriginW_B() {
            return this.OriginEX_B - this.OriginSX_B; //经度宽度
        },
        OriginH_B() {
            return this.OriginEY_B - this.OriginSY_B; //纬度高度
        },
        RatioX_B() {
            return this.OriginW_B / this.canvasWidth_B; //经度比例
        },
        RatioY_B() {
            return this.OriginH_B / this.canvasHeight_B; //纬度比例
        }
    },
    mounted() {
        this.drawMap(this.mapList);
        this.getTaskStartId(true); // 获取开始id
    },
    watch: {
        // 在页面渲染完后执行
        mapList: {
            handler: function (newVal, oldVal) {
                console.log("数组更新", newVal);
                this.drawMap(newVal);
            },
            deep: true, // 深度监听
            flush: 'post' // 保证能访问DOM
        },
        // 当宽或者高改变时, 重新绘制地图
        canvasDimensions() {
            this.mapListXY_conv(); // 重新计算地图坐标
            this.drawMap(this.mapList);
        }
    },
    methods: {
        // 将本地数据同步到远程
        sync_db_filter(){
            // get /sync_db_filter
            axios.get('/sync_db_filter').then(res => {
                console.log(res.data);
                alert(res.data.msg)
            });
        },
        // 计算属性, 设置地图边框颜色
        setmapborder(index) {
            // console.log("设置边框 索引: ", index);
            if (!this.mapList[index].mst) {
                return this.getBackgroundColor;
            } else {
                return `bg-${this.mapList[index].mst}`;
            }
        },
        // 获取数据库未开始的任务id
        async getTaskStartId(upstartId) {
            try {
                const res = await axios.get('/gettaskid');
                // console.log(res.data);
                let taskstartId = res.data.id;
                this.taskstartId = taskstartId;
                if (upstartId) {
                    this.startId = taskstartId;
                }
                return taskstartId;
            } catch (error) {
                console.error('Error fetching task ID:', error);
            }
        },
        // 获取地图列表
        getmapList() {
            this.mapList = []; // 清空地图列表
            axios.get(`/get_markLists?start=${this.startId}&end=${this.startId + this.taskSize - 1}`).then(res => {
                // console.log(res.data); // 未来可能会变成流
                this.mapList_origin = res.data;
                this.mapListXY_conv(); // 重新计算地图坐标
                console.log(this.mapList);
            });
        },
        mapListXY_conv(){
            console.log("重新计算地图坐标");
            this.mapList = []; // 清空地图列表
            // 从原始地图数据转换为不同尺寸的画板坐标
            for (let i = 0; i < this.mapList_origin.length; i++) {
                let mapList_obj = this.mapList_origin[i];
                let markList = this.mapList_calLtoXY(mapList_obj.data_markList, mapList_obj.data_campus);
                this.mapList.push({
                    id: mapList_obj.id,
                    markList: markList,
                    data_phoneInfo: mapList_obj.data_phoneInfo,
                    data_campus: mapList_obj.data_campus,
                });
            }
        },
        // 地图数据转换_地理转画板
        mapList_calLtoXY(data_markList, data_campus) {
            // data_markList列表是这样的[{"isStartPosition":true,"latLng":{"latitude":36.884830260641245,"longitude":118.77333133647896}},{"latLng":{"latitude":36.884897009246664,"longitude":118.77312943020856}}...]
            // latitude: 纬度,对应Y坐标 longitude: 经度,对应X坐标
            // 把点传给calLtoXY函数,返回画板坐标
            let markList = [];
            let marks = JSON.parse(data_markList);
            for (let i = 0; i < marks.length; i++) {
                let mark = marks[i]
                // console.log(mark);
                let markXY = this.calLtoXY(mark.latLng.longitude, mark.latLng.latitude, data_campus);
                if (mark.isStartPosition) {
                    markList.push({ "isStartPosition": true, markXY })
                } else {
                    markList.push(markXY)
                }
            }
            // console.log(markList);
            return markList;
        },
        // 点击地图的评分按钮
        setmapscoretype(type, index) {
            console.log("点击了评分按钮", type, index);
            this.mapList[index].mst = type;
        },
        // 提交评分数据
        async upscorelist() {
            this.scorelist = { great: [], good: [], normal: [], bad: [] };
            let scoreList = this.scorelist // 没有声明新变量! 直接操作this.scorelist会报错
            for (let i = 0; i < this.mapList.length; i++) {
                let mapList_obj = this.mapList[i];
                if (mapList_obj.mst) {
                    scoreList[mapList_obj.mst].push(mapList_obj.id);
                } else {
                    if (this.mapscoretype) 
                        scoreList[this.mapscoretype].push(mapList_obj.id);
                }
            }
            console.log("提交评分", scoreList);
            // 改成同步提交, 创建变量存放返回的数据
            const response = await axios.post('/updatemanualscore', scoreList)
            const responseData = response.data;
            if(responseData == "ok"){
                alert("提交成功");
                this.startId += this.taskSize;
                if (this.AutoSwitch) {
                    this.getmapList();
                }
            } else {
                console.log(responseData);
                alert("提交失败"); 
            }
        },
        // 计算坐标,将地图坐标转换为画板坐标
        calLtoXY(longitude, latitude, data_campus) {
            let ituse = {
                x: (longitude - this.getOriginSX(data_campus)) / this.getRatioX(data_campus),
                y: (latitude - this.getOriginSY(data_campus)) / this.getRatioY(data_campus)
            };
            // console.log("caXYtoL", ituse.x, ituse.y);
            return ituse;
        },
        drawMap(mapList) {
            for (let i = 0; i < mapList.length; i++) {
                let mapList_obj = mapList[i];
                // console.log("画图分段数据:", mapList_obj);
                let canvas = document.getElementById('canvas' + mapList_obj.id);
                // console.log('选择: canvas' + mapList_obj.id);
                let ctx = canvas.getContext('2d');
                ctx.strokeStyle = "rgb(74, 105, 189)";
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                ctx.font = "20px Arial";
                ctx.fillText("index: " + (i+1), 10, 20);
                ctx.fillText("uid: " + mapList_obj.id, 10, 40);
                ctx.fillText("phone: " + mapList_obj.data_phoneInfo, 10, 60);
                ctx.fillText("campus: " + mapList_obj.data_campus, 10, 80);
                ctx.beginPath();
                for (let j = 0; j < mapList_obj.markList.length; j++) {
                    let mark = mapList_obj.markList[j];
                    if (mark.isStartPosition) {
                        ctx.moveTo(mark.x, mark.y);
                    } else {
                        ctx.lineTo(mark.x, mark.y);
                    }
                }
                ctx.stroke();
            }
        },
        campusType(data_campus) { // 将原始校区数据转换成程序内部校区类型
            return data_campus == 1 ? 1 : 2;
        },
        // 获取地图图片
        getMapImg(data_campus) {
            console.log("获取地图图片", data_campus);
            return this.campusType(data_campus) == 1 ? this.sdutmap_A: this.sdutmap_B
        },
        // 获取画板宽度
        getCanvasWidth(data_campus) {
            return this.campusType(data_campus) == 1 ? this.canvasWidth_A : this.canvasWidth_B;
        },
        // 获取画板高度
        getCanvasHeight(data_campus) {
            return this.campusType(data_campus) == 1 ? this.canvasHeight_A : this.canvasHeight_B;
        },
        // 获取原点经度坐标
        getOriginSX(data_campus) {
            return this.campusType(data_campus) == 1 ? this.OriginSX_A : this.OriginSX_B;
        },
        // 获取原点纬度坐标
        getOriginSY(data_campus) {
            return this.campusType(data_campus) == 1 ? this.OriginSY_A : this.OriginSY_B;
        },
        // 获取对点经度坐标
        getOriginEX(data_campus) {
            return this.campusType(data_campus) == 1 ? this.OriginEX_A : this.OriginEX_B;
        },
        // 获取对点纬度坐标
        getOriginEY(data_campus) {
            return this.campusType(data_campus) == 1 ? this.OriginEY_A : this.OriginEY_B;
        },
        // 获取经度比例
        getRatioX(data_campus) {
            return this.campusType(data_campus) == 1 ? this.RatioX_A : this.RatioX_B;
        },
        // 获取纬度比例
        getRatioY(data_campus) {
            return this.campusType(data_campus) == 1 ? this.RatioY_A : this.RatioY_B;
        }
    }
}
</script>

<style scoped>
.bg-great {
    background-color: rgb(74, 105, 189);
    border-color: rgb(74, 105, 189);
}
.bg-good {
    background-color: rgb(12, 182, 12);
    border-color: rgb(12, 182, 12);
}

.bg-normal {
    background-color: orange;
    border-color: orange;
}

.bg-bad {
    background-color: red;
    border-color: red;
}

.colormap_btn_great{
    background: rgba(135, 192, 202, 0.3);
}

.colormap_btn_good{
    background: rgba(150, 217, 41, 0.3);
}

.colormap_btn_normal{
    background: rgba(255, 165, 2, 0.3);
}

.colormap_btn_bad{
    background: rgba(124, 25, 30, 0.3);
}

#main {
    width: 100%;
    max-height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    overflow: auto;
    margin: 0;
    padding: 0;
}

#toolbar {
    text-align: center;
}

#map {
    display: flex;
    flex-grow: 1;
    overflow: auto;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: space-around;
}

.map_block {
    position: relative;
    margin: 10px;
    border-width: 4px;
    border-style: solid;
}

.map_img {
    display: block;
    position: absolute;
    z-index: 1;
    top: 0px;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
}

.mapcanvas {
    position: relative;
    z-index: 3;
    background: rgba(255, 255, 255, 0);
    pointer-events: none;
}

.colormap {
    position: absolute;
    display: flex;
    flex-direction: row;
    z-index: 2;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}

.colormap>div {
    display: flex;
    flex-grow: 1;
}

.colormap_btn {
    display: flex;
    flex-direction: column;
}

.colormap_btn>div {
    display: flex;
    flex-grow: 1;
}

.getBackgroundColorp{
    margin: 0;
}
</style>