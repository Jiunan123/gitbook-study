# vue-amap

1. 安装

```code
npm i vue-amap --save
```

2. 初始化

```js
// amap.js

import Vue from 'vue'
import VueAMap from 'vue-amap'

Vue.use(VueAMap)

// 初始化vue-amap
VueAMap.initAMapApiLoader({
    // 高德的key（这个key要申请，以下可用）
    key: 'bd5caaca9818686ff671dd11e3a5dce3',
    // 插件集合
    plugin: [
        'AMap.Autocomplete',
        'AMap.PlaceSearch',  // 查找附近
        'AMap.Scale',        // 缩放
        'AMap.OverView',
        'AMap.ToolBar',
        'AMap.MapType',      // 地图类型
        'AMap.PolyEditor',
        'AMap.CircleEditor', // 圆编辑插件
        'AMap.Geolocation',  // 定位
        'AMap.Weather',      // 天气
        'AMap.CitySearch'    // 城市
    ],
    // 高德 sdk 版本
    v: '1.4.4'
})
```

```js
// main.js
import './assets/js/amap';
```

3. 官方文档

[高德地图API](https://lbs.amap.com/api/javascript-api/guide/services/autocomplete/)

[vue-amap](https://elemefe.github.io/vue-amap/#/zh-cn/introduction/init)

测试：

- API上有提供测试运行的功能
- 定位只能在手机端起作用。

4. 使用

```vue
<template>
    <van-cell icon="location-o" class="c-position_panel" v-if="show" size="15px">
        <template v-if="position.lng">{{ position.lng }}，{{ position.lat }}</template>
        <template v-else>定位获取失败，请刷新定位</template>
        <template #right-icon>
            <van-icon name="replay" class="van-icon van-cell__right-icon" size="15px" @click="reloadLocation" />
        </template>
    </van-cell>
</template>

<script>
// 该文件功能为“我要接单”-附近地点，功能被砍，指做备份作用。
import { lazyAMapApiLoaderInstance } from "vue-amap";

/**
 * 功能简介：定位控件
 * 父组件使用ref引用本组件实例，获取position数据。
 */
export default {
    name: 'Mytest',
    data() {
        return {
            show: false,
            position: {
                lng: "",
                lat: "",
            },
        };
    },
    methods: {
        getLocalLocation() {
            lazyAMapApiLoaderInstance.load().then(() => {
                let Geolocation = new AMap.Geolocation(); // 定位
                let p = new Promise((res, rej) => {
                    Geolocation.getCurrentPosition((status, result) => {
                        if (result && result.position) {
                            this.position.lng = result.position.lng;
                            this.position.lat = result.position.lat;
                            // console.log(result.position);
                            this.$toast.fail(JSON.stringify(result.position))
                            res([result.position.lng, result.position.lat])
                        } else {
                            this.$toast.fail("定位失败");
                            rej();
                        }
                    });
                });
                p.then((position) => {
                    var placeSearch = new AMap.PlaceSearch({
                        pageSize: 10,    // 每页10条
                        pageIndex: 1,    // 获取第一页
                    });
                    // 第一个参数是关键字，这里传入的空表示不需要根据关键字过滤
                    // 第二个参数是经纬度，数组类型
                    // 第三个参数是半径，周边的范围
                    // 第四个参数为回调函数
                    placeSearch.searchNearBy('', position, 1000, (status, result) => {
                        if (result.info === 'OK') {
                            var locationList = result.poiList.pois; // 周边地标建筑列表
                            // console.log(locationList);
                            this.$toast.fail(JSON.stringify(locationList));
                            // 生成地址列表html
                            // createLocationHtml(locationList);
                            // 返回的result格式如下：
                            
                        } else {
                            console.log('获取位置信息失败!');
                        }
                    });
                }).catch(() => { })
            });
        },
        reloadLocation() {
            this.getLocalLocation();
        },
        getLocation() {
            return this.position;
        }
    },
    mounted() {
        this.show = true;
        this.getLocalLocation();
    },
};
</script>

<style lang="scss" scoped>
.c-position_panel {
    border-radius: 2em;
    line-height: 2;
    background: var(--color-grey-bg-area);
    color: var(--color-grey);
    padding: 0.5em 2em;
    vertical-align: middle;
    font-size: var(--size-lg);
    .van-cell__value {
        text-align: center;
        color: inherit;
    }
    .van-icon {
        top: 0.2em;
    }
    .van-cell__right-icon {
        color: var(--color-info);
    }
}
</style>
```

```json
// 查看附近的功能。
{
    "info": "OK",
    "poiList": {
        "pois": [{
                "id": "B0HASU7OIE",
                "name": "YOLO餐吧",
                "type": "餐饮服务;餐饮相关场所;餐饮相关",
                "location": {
                    "Q": 39.909235,
                    "R": 116.40455199999997,
                    "lng": 116.404552,
                    "lat": 39.909235
                },
                "address": "菖蒲河公园",
                "tel": "13501087611",
                "distance": 181,
                "shopinfo": "0",
                "website": "",
                "pcode": "110000",
                "citycode": "010",
                "adcode": "110101",
                "postcode": "",
                "pname": "北京市",
                "cityname": "北京市",
                "adname": "东城区",
                "email": "",
                "photos": "",
                "entr_location": null,
                "exit_location": null,
                "groupbuy": false,
                "discount": false,
                "indoor_map": false
            },
            {
                "id": "B000A7BLUQ",
                "name": "北京贵宾楼饭店",
                "type": "住宿服务;宾馆酒店;五星级宾馆|餐饮服务;餐饮相关场所;餐饮相关",
                "location": {
                    "Q": 39.908888,
                    "R": 116.407243,
                    "lng": 116.407243,
                    "lat": 39.908888
                },
                "address": "东长安街35号",
                "tel": "010-65137788",
                "distance": 196,
                "shopinfo": "0",
                "website": "www.grandhotelbeijing.com",
                "pcode": "110000",
                "citycode": "010",
                "adcode": "110101",
                "postcode": "100006",
                "pname": "北京市",
                "cityname": "北京市",
                "adname": "东城区",
                "email": "",
                "photos": [{
                    "title": "Logo",
                    "url": "http://store.is.autonavi.com/showpic/f488538345e1919e4bd941b34dddffef",
                    "provider": ""
                }, {
                    "title": "酒店外观",
                    "url": "http://store.is.autonavi.com/showpic/9f71c8b78463bec94d3d63bd45eb4e3e",
                    "provider": ""
                }, {
                    "title": "外观",
                    "url": "http://store.is.autonavi.com/showpic/d3741098f1661db9a19e64d5a0a14d0b",
                    "provider": ""
                }],
                "entr_location": {
                    "Q": 39.908137,
                    "R": 116.40750100000002,
                    "lng": 116.407501,
                    "lat": 39.908137
                },
                "exit_location": null,
                "groupbuy": false,
                "discount": false,
                "indoor_map": false
            }
        ],
        "count": 2,
        "pageIndex": 1,
        "pageSize": 5
    }
}
```

