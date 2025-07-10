<template>
	<view class="valve-control-container">
        <image class="banner" src="/static/images/valve-control-banner.png"></image>
        <view class="u-collapse-box ">
            <u-collapse
                :value="collapseValue"
                @change="handleCollapseChange"
                @close="handleCollapseClose"
                @open="handleCollapseOpen"
            >
                <u-collapse-item
             		v-for="(item, index) in valveList"
					:key="index"
                    :title=item.name
                    :name="index"
                >   
                    <view slot="title1" class="u-collapse-item-left">
                        <i 
                            :class="['iconfont u-collapse__arrow', collapseValue.indexOf(index) > -1? 'u-cell__right-icon-wrap--down' : 'u-cell__right-icon-wrap--up' ]"
                        >&#xe711;</i>
                        <text class="u-collapse__title">{{item.name}}</text>
                    </view>
                    <view slot="value1" class="u-collapse-item-right">
                        <view class="line"></view>
                        <view class="info" @click.stop="handleNavigateToAll(k)">
                            <i class="iconfont">&#xe70a;</i>
                            <text class="num">{{ item.num }}</text>
                        </view>
                    </view>
                    <!-- <text slot="value1" class="u-collapse__title">{{item.name}}</text> -->
                    <view                           
                         v-for="(k, index) in item.detail" 
                         class="valve-details-content"
                         @click="() => handleNavigateTo(k)"
                    >
                        <view class="valve-details-content-item">
                            <view class="valve-details-content-item left">
                                <view class="img" :style="{backgroundColor: k.bgColor}">{{k.locationName[0] }}</view>
                                <view class="name">{{ k.locationName }}</view>
                            </view>
                            <view class="valve-details-content-item right">
                                <view class="line"></view>
                                <view class="info">
                                    <i class="iconfont">&#xe70a;</i>
                                    <text class="num">{{ k.num }}</text>
                                </view>
                                <i class="iconfont arrow-right" >&#xe711;</i>
                            </view>
                        </view>
                    </view>
                </u-collapse-item>
            </u-collapse>
        </view>
        <view class="tabbar">
            <u-tabbar
                :value="activeTabbar"
                @change="handleTabbarChange"
                activeColor="#81B337"
                :fixed="true"
                :placeholder="true"
                :safeAreaInsetBottom="true"
            >
                <u-tabbar-item text="返回首页">
                    <i class="iconfont active" slot="active-icon">&#xe704;</i>
                    <i class="iconfont inactive" slot="inactive-icon">&#xe705;</i>
                </u-tabbar-item>
                <u-tabbar-item text="阀门控制">
                    <i class="iconfont active" slot="active-icon">&#xe706;</i>
                    <i class="iconfont inactive" slot="inactive-icon">&#xe707;</i>
                </u-tabbar-item>
                <u-tabbar-item text="设备管理">
                    <i class="iconfont active" slot="active-icon">&#xe708;</i>
                    <i class="iconfont inactive" slot="inactive-icon">&#xe709;</i>
                </u-tabbar-item>
            </u-tabbar>
        </view>
	</view>
</template>
<script>
	export default {
		data() {
			return {
                collapseValue: [0],
                valveList: [
                    {
                        name: '18678960523',
                        num: 50,
                        detail: [
                            {
                                locationName: '西地块',
                                num: 3,
                                bgColor: ''
                            },
                            {
                                locationName: '东地块',
                                num: 3,
                                bgColor: ''
                            },
                            {
                                locationName: '六连北地块',
                                num: 3,
                                bgColor: ''
                            }
                        ]
                    }, 
                    {
                        name: '16878964567',
                        num: 50,
                        detail: [
                            {
                                locationName: '西地块',
                                num: 3,
                                bgColor: ''
                            },
                            {
                                locationName: '东地块',
                                num: 3,
                                bgColor: ''
                            },
                            {
                                locationName: '六连北地块六连北地块',
                                num: 3,
                                bgColor: ''
                            }
                        ]
                    }
                ],
                activeTabbar: 1
			}
		},
        onLoad() {
            // 给阀门信息中的 class=img 的元素随机添加背景色
            const colorList = ['#81B337','#FCCA00','#E99D42','#BD3124', '#815f25','#1677b3', '#1491a8', '#207f4c','#645822', '#ed3b2f'];
            this.valveList.forEach(item => {
                item.detail.forEach(k => {
                    let color = this.getRandomIndex(colorList);
                    k.bgColor = color;
                    colorList.splice(colorList.indexOf(color), 1);
                    if(colorList.length === 0) {
                        colorList.push(...colorList);
                    }
                })
            })
        },
		methods: {
            // 获取随机数
            getRandomIndex(arr) {
                const randomIndex = Math.floor(Math.random() * arr.length);
                return arr[randomIndex];
            },

            handleNavigateToAll(valveInfo) {
                console.log(valveInfo);
                uni.navigateTo({
                    url: '/pagesA/valveControl/allValveInfo?valveInfo=' + JSON.stringify(valveInfo),
                });
            },

            handleNavigateTo(valveInfo) {
                console.log(valveInfo.locationName);
                uni.navigateTo({
                    url: '/pagesA/valveControl/newValueInfo?valveInfo=' + JSON.stringify(valveInfo),
                });
            },

            handleCollapseChange(activeNames) {
                const openValues = activeNames.filter(item => item.status === 'open').map(item => item.name);
                this.collapseValue = openValues;
                console.log('1', this.collapseValue);
            },

            handleCollapseClose(activeNames) {
                console.log('2', activeNames);
            },

            handleCollapseOpen(activeNames) {
                console.log('3', activeNames);
            },

            handleTabbarChange(name) {
                if(name === 0) {
                    uni.reLaunch({
						url: '/pages/index/index'
					});
                }else if(name === 2) {
                    uni.navigateTo({
						url: '/pagesC/equipManage/index'
					});
                }
            }

		}
	}
</script>

<style lang="scss" scoped>
.valve-control-container {
    .banner {
        display: block;
        width: 100%;
        height: 80px;
    }
    ::v-deep .u-cell{
        &__title-text {
            font-size: 14px;
            font-weight: 600;
            color: #4f4f4f;
        }
    }
    ::v-deep .u-collapse-item__content {
        background: #efefef;
        &__text {
            padding: 10px !important;
        }
    }
    .valve-details-content {
        display: flex;
        align-items: center;
        padding: 10px 14px;
        border-radius: 6px;
        background: #fff;
        box-shadow: 0px 2px 6px 0px rgba(0,0,0,0.05);
        &+ .valve-details-content {
            margin-top: 10px;
        }
        &-item {
            display: flex;
            align-items: center;
            width: 100%;
            & .left {
                display: flex;
                .img {
                    display: block;
                    width: 40px;
                    height: 40px;
                    border-radius: 50%;
                    text-align: center;
                    line-height: 40px;
                    font-size: 20px;
                    font-weight: 600;
                    color: #fff;
                }
                .name {
                    padding-left: 10px;
                    font-size: 16px;
                    font-weight: 600;
                    color: #4f4f4f;
                    overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;
                    width: 130px;
                }
            }
            &.right {
                flex: 1;
                display: flex;
                font-size: 14px;
                .line {
                    width: 1px;
                    height: 30px;
                    background: #efefef;
                    margin-right: 20px;
                }
                .info {
                    display: flex;
                    .iconfont {
                        padding-right: 10px;
                        color: #cecece;
                    }
                    .num {
                        color: $u-main-color;
                    }
                }
                .arrow-right {
                    padding-left: 23px;
                    color: #cecece;
                }
            }
        }
    }
    // .u-collapse-content {}
    // ::v-deep .u-collapse-item {
    //     &__content {
    //         background: #efefef;
    //         &__text {
    //             margin: 10px !important;
    //             background: #fff;
    //         }
    //     }
    // }
    .tabbar {
        ::v-deep .u-tabbar {
            .u-tabbar--fixed {
                border-color: #cecece !important;
            }
            .u-tabbar-item {
                flex-direction: row;
                .u-tabbar-item__icon {
                    width: auto;
                    font-size: 14px;
                }
                .u-tabbar-item__text {
                    font-size: 14px;
                    margin-left: 10px;
                    margin-top: 0;
                }
                &:nth-of-type(2) {
                    position: relative;
                    &::before {
                        content: "";
                        position: absolute;
                        top: 20%;
                        left: 0;
                        width: 1px;
                        height: 60%;
                        background: #cecece;
                    }
                    &::after {
                        content: "";
                        position: absolute;
                        top: 20%;
                        right: 0;
                        width: 1px;
                        height: 60%;
                        background: #cecece;
                    }
                }
            }
        }
    }
    .u-collapse-box {
        ::v-deep .u-collapse-item-left {
            display: flex;
            align-items: center;
            .iconfont {
                display: inline-block;
                width: 16px;
                margin-right: 10px;
            }
            .u-cell__right-icon-wrap--up {
                transform: rotate(-90deg);
            }
            .u-cell__right-icon-wrap--down {
                transform: rotate(90deg);
            }
        }
        ::v-deep .u-collapse-item-right{
            flex: 1;
            display: flex;
            align-items: center;
            font-size: 14px;
            .line {
                width: 1px;
                height: 30px;
                background: #efefef;
                margin: 0 20px;
            }
            .info {
                display: flex;
                .iconfont {
                    padding-right: 10px;
                    color: #bebebe;
                }
                .num {
                    color: $u-main-color;
                }
            }
            .arrow-right {
                padding-left: 23px;
                color: #bebebe;
            }
        }
        ::v-deep .u-cell__body {
            padding: 15px !important;
        }
        .u-collapse__title {
            font-size: 14px;
            font-weight: 600;
            color: #4f4f4f;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
            width: 130px;
        }
        .u-collapse__value {
            display: flex;
            align-items: center;
            font-size: 12px;
            color: #bebebe;
            padding-right: 15px;
            i {
                padding-right: 5px;
                color: #bebebe;
            }
        }
        .u-collapse__arrow {
            color: #cecece;
        }
        
    }
}
</style>