<template>
    <view class="valve-control-container">
        <!-- 日期选择和搜索 -->
        <view class="top-container">
            <view class="date-input-box" @click="handleOpenCalendar">
                <!-- 日期input -->
                <u-input
                    placeholder="请选择日期区间"
                    v-model="dateRangeText"
                    readonly
                >
                    <template #suffix>
                        <i class="iconfont" @click.stop="handleSearch"
                            >&#xe712;</i
                        >
                    </template>
                </u-input>
            </view>
            <!-- 日历组件 -->
            <u-popup
                :show="showCalendar"
                :round="10"
                :closeOnClickOverlay="false"
                @close="showCalendar = false"
            >
                <uni-calendar
                    :lunar="true"
                    :range="true"
                    @change="handleCalendarChange"
                />
                <view class="calendar-button-group">
                    <view class="calendar-button cancel-button">
                        <u-button @click="handleCloseCalendar">取消</u-button>
                    </view>
                    <view class="calendar-button confirm-button">
                        <u-button type="primary" @click="handleConfirmCalendar"
                            >确定</u-button
                        >
                    </view>
                </view>
            </u-popup>
            <!-- 按钮区 -->
            <view class="button-group">
                <view class="checkbox-button" @click="handleCheckAll">
                    <view>
                        <u-icon
                            v-if="checkAll"
                            name="checkmark-circle-fill"
                            size="20"
                            color="#fff"
                        ></u-icon>
                        <view v-else class="circle-check"></view>
                    </view>
                    <text class="check-text">全选</text>
                </view>
                <view class="action-btn" @click="handleOpenCloseValveModal"
                    >一键关阀</view
                >
                <view class="action-btn" @click="handleOpenValveBatch"
                    >批量控制</view
                >
                <view class="action-btn">解除时控</view>
            </view>
        </view>
        <!-- 阀门卡片列表 -->
        <scroll-view
            class="valve-list-scroll"
            scroll-y="true"
            @scrolltolower="loadMore"
            :style="{ height: scrollViewHeight + 'px' }"
        >
            <view class="valve-list">
                <view
                    v-for="(item, idx) in valveList"
                    :key="idx"
                    class="valve-card"
                    :class="{ selected: item.selected }"
                >
                    <view class="card-header">
                        <view class="card-header-left">
                            <view @click="toggleSelect(idx)">
                                <u-icon
                                    v-if="item.selected"
                                    name="checkmark-circle-fill"
                                    size="20"
                                    color="#0F40F5"
                                ></u-icon>
                                <view v-else class="circle-check"></view>
                            </view>
                            <view class="valve-avatar">
                                <image src="@/static/images/common.jpg" />
                            </view>
                            <view class="valve-title-group">
                                <text class="valve-title">{{ item.name }}</text>
                                <view class="valve-status-row">
                                    <view class="valve-singal">
                                        <!-- 信号大于75 -->
                                        <i
                                            v-if="Number(item.signal) > 75"
                                            class="iconfont valve-singal-iconfont orange"
                                            >&#xe70b;</i
                                        >
                                        <!-- 信号大于50 -->
                                        <i
                                            v-else-if="
                                                Number(item.signal) >= 50
                                            "
                                            class="iconfont valve-singal-iconfont orange"
                                            >&#xe71b;</i
                                        >
                                        <!-- 信号小于50 -->
                                        <i
                                            v-else="Number(item.signal) < 50"
                                            class="iconfont valve-singal-iconfont orange"
                                            >&#xe71c;</i
                                        >
                                        <i
                                            :class="[
                                                'iconfont valve-singal-iconfont',
                                                Number(item.batteryNum) >= 50
                                                    ? 'green'
                                                    : Number(item.batteryNum) <=
                                                        10
                                                      ? 'red'
                                                      : 'orange'
                                            ]"
                                            >&#xe70d;</i
                                        >
                                    </view>
                                    <text
                                        :class="[
                                            'battery',
                                            Number(item.batteryNum) >= 50
                                                ? 'green'
                                                : Number(item.batteryNum) <= 10
                                                  ? 'red'
                                                  : 'orange'
                                        ]"
                                        >{{ item.battery }}%</text
                                    >
                                    <text class="update-time"
                                        >{{ item.updateTime }} 更新</text
                                    >
                                </view>
                            </view>
                        </view>
                        <view class="card-header-right">
                            <i class="iconfont" style="color: #cecece"
                                >&#xe711;</i
                            >
                        </view>
                    </view>
                    <view class="card-body">
                        <view
                            v-if="item.flow !== undefined"
                            class="card-body-top"
                        >
                            <view class="info-item" style="margin-left: 30px">
                                <text class="blue" style="padding-right: 10px"
                                    >{{ item.flow }}:</text
                                >
                                <text class="info-item-text">压力</text>
                                <view class="dot blue"></view>
                            </view>
                            <view class="info-item" style="margin-right: 30px">
                                <view class="dot blue"></view>
                                <text
                                    class="info-item-text"
                                    style="padding-right: 5px"
                                    >压力:</text
                                >
                                <text class="blue">{{ item.flow }}</text>
                            </view>
                        </view>
                        <view class="card-body-content">
                            <view class="card-body-left">
                                <view
                                    :class="{
                                        'valve-btn': true,
                                        active: item.bOpen
                                    }"
                                    >B阀一开</view
                                >
                                <view>
                                    <view
                                        v-if="item.flow !== undefined"
                                        class="info-item"
                                    >
                                        <view class="dot green"></view>
                                        <text class="info-item-text"
                                            >实时流量:</text
                                        >
                                        <text class="green">{{
                                            item.flow
                                        }}</text>
                                    </view>
                                    <view v-else class="info-item">
                                        <view class="dot blue"></view>
                                        <text class="info-item-text"
                                            >实时压力:</text
                                        >
                                        <text class="blue">{{
                                            item.pressure
                                        }}</text>
                                    </view>
                                </view>
                            </view>
                            <view class="card-body-center">
                                <view class="valve-img">
                                    <image
                                        src="@/static/images/valve.png"
                                        mode="widthFix"
                                    />
                                </view>
                                <view
                                    class="close-btn"
                                    @click="() => handleOpenQuickOpenValveModal(item)"
                                >
                                    <i class="iconfont">&#xe728;</i>
                                    <text
                                        style="margin-left: 5px; color: #6c6c6c"
                                        >{{
                                            item.bOpen ? '关阀' : '开阀'
                                        }}</text
                                    >
                                </view>
                            </view>
                            <view class="card-body-right">
                                <view
                                    :class="{
                                        'valve-btn': true,
                                        active: item.aOpen
                                    }"
                                    >A阀一开</view
                                >
                                <view class="info-item">
                                    <view class="dot orange"></view>
                                    <text>开度:</text>
                                    <text
                                        class="orange"
                                        style="padding-left: 4px"
                                        >{{ item.open }}</text
                                    >
                                    <i class="iconfont left-time-icon"
                                        >&#xe726;</i
                                    >
                                </view>
                            </view>
                        </view>
                    </view>
                </view>
            </view>

            <!-- 加载更多提示 -->
            <view class="load-more-container" v-if="hasMore">
                <view class="loading-text" v-if="loading">
                    <u-loading-icon mode="spinner" size="20"></u-loading-icon>
                    <text style="margin-left: 10px">加载中...</text>
                </view>
                <view class="loading-text" v-else>
                    <text>上拉加载更多</text>
                </view>
            </view>

            <!-- 没有更多数据提示 -->
            <view
                class="no-more-container"
                v-if="!hasMore && valveList.length > 0"
            >
                <text class="no-more-text">没有更多数据了</text>
            </view>
        </scroll-view>
        <!-- 批量控制 -->
        <u-popup :closeOnClickOverlay="false" :show="valveBatchShow">
            <view class="popup-container">
                <view class="popup-header">
                    <u-button
                        type="primary"
                        :plain="true"
                        text="取消"
                        @click="handleSettingCancel"
                    ></u-button>
                    <text>请设置</text>
                    <u-button
                        type="primary"
                        :plain="true"
                        text="确定"
                        @click="handleSettingCinfirm"
                    ></u-button>
                </view>
                <view class="popup-body">
                    <view class="popup-title">设置阀门工作参数</view>
                    <view class="popup-body-item">
                        <view class="popup-select-box">
                            <u-row gutter="10">
                                <u-col span="4">
                                    <view class="popup-select-box-item">
                                        <text>选择阀门</text>
                                        <view>
                                            <view
                                                class="cover-container"
                                                @click="handleShowValve1"
                                            >
                                                <u-input
                                                    placeholder="请选择"
                                                    v-model="valve1"
                                                    suffixIcon="arrow-down"
                                                    :customStyle="{
                                                        borderColor: '#bbb'
                                                    }"
                                                    readonly
                                                >
                                                </u-input>
                                                <view class="cover-box"></view>
                                            </view>
                                            <u-action-sheet
                                                :show="showValve1"
                                                :actions="valveActions1"
                                                closeOnClickOverlay
                                                @close="showValve1 = false"
                                                @select="handleChangeValve1"
                                            >
                                            </u-action-sheet>
                                        </view>
                                    </view>
                                </u-col>
                                <u-col span="4">
                                    <view class="popup-select-box-item">
                                        <text>工作模式</text>
                                        <view>
                                            <view
                                                class="cover-container"
                                                @click="handleShowValve2"
                                            >
                                                <u-input
                                                    placeholder="请选择"
                                                    v-model="valve2"
                                                    suffixIcon="arrow-down"
                                                    :customStyle="{
                                                        borderColor: '#bbb'
                                                    }"
                                                    readonly
                                                >
                                                </u-input>
                                                <view class="cover-box"></view>
                                            </view>
                                            <u-action-sheet
                                                :show="showValve2"
                                                :actions="valveActions2"
                                                closeOnClickOverlay
                                                @close="showValve2 = false"
                                                @select="handleChangeValve2"
                                            >
                                            </u-action-sheet>
                                        </view>
                                    </view>
                                </u-col>
                                <u-col span="4">
                                    <view class="popup-select-box-item">
                                        <text>参数</text>
                                        <view>
                                            <view class="cover-container">
                                                <u-input
                                                    placeholder="请输入"
                                                    v-model="valve3"
                                                    :customStyle="{
                                                        borderColor: '#bbb'
                                                    }"
                                                    @confirm="
                                                        handleChangeValve3
                                                    "
                                                >
                                                </u-input>
                                                <!-- <view class="cover-box"></view> -->
                                            </view>
                                            <!-- <u-action-sheet
                                                :show="showValve3"
                                                :actions="valveActions3"
                                                closeOnClickOverlay
                                                @close="showValve3 = false"
                                                @select="handleChangeValve3"
                                            >
                                            </u-action-sheet> -->
                                        </view>
                                    </view>
                                </u-col>
                            </u-row>
                        </view>
                    </view>
                    <view class="popup-title">设置阀门结束参数</view>
                    <view class="popup-body-item">
                        <view>
                            <u-radio-group
                                v-model="valveSettingData.endParam"
                                shape="circle"
                                placement="row"
                                activeColor="#0F40F5"
                                @change="handleValveSetting"
                            >
                                <u-radio
                                    v-for="(item, index) in valveSettingList"
                                    :key="index"
                                    :label="item.name"
                                    :name="item.name"
                                >
                                </u-radio>
                            </u-radio-group>
                        </view>
                        <u-line color="#bbb"></u-line>
                        <view class="popup-body-item-input">
                            <u--input
                                placeholder="请输入阀门结束容量m³，如150m³，直接填写150即可"
                                v-model="valveSettingData.endNum"
                                :customStyle="{
                                    borderColor: '#bbb'
                                }"
                            ></u--input>
                        </view>
                    </view>
                </view>
            </view>
        </u-popup>
        <!-- 一键关阀 -->
        <CustomModal
            :showModal="closeValveModalVisiable"
            title="一键关阀"
            @cancel="handlecloseValveModal"
            @confirm="handlecloseValveCinfirm"
        >
            <view slot="content">
                <view class="modal-text"> 你确定要关闭所选的阀门吗？ </view>
            </view>
        </CustomModal>
        <!-- 快捷开阀 -->
        <CustomModal
            :showModal="quickOpenValveModalVisiable"
            title="快捷开阀"
            @cancel="handleQuickOpenValveCancel"
            @confirm="handleQuickOpenValveConfirm"
        >
            <view slot="content">
                <view class="modal-content-top">
                    <view class="title">是否开启此阀门？</view>
                    <view class="sup-title"> · 阀门默认开度为 100%。 </view>
                    <view class="sup-title">
                        · 如需调整开度，请在下方设定后点击 “确定”即可。
                    </view>
                </view>
                <view class="modal-content-center">
                    <view class="slider-value-text">{{ stepValue }}%</view>
                    <view class="slider-group">
                        <view class="slider-minus" @click="handleSliderMinus">
                            <i class="iconfont">&#xe72a;</i>
                        </view>
                        <u-slider
                            v-if="quickOpenValveModalVisiable"
                            v-model="stepValue"
                            min="0"
                            max="100"
                            height="20px"
                            blockSize="20px"
                            blockWidth="20px"
                            activeColor="#0F40F5"
                            inactiveColor="#E0E0E0"
                            blockColor="#0F40F5"
                        ></u-slider>
                        <view class="slider-plus" @click="handleSliderPlus">
                            <i class="iconfont">&#xe729;</i>
                        </view>
                    </view>
                </view>
            </view>
        </CustomModal>
          <!-- 关阀 -->
          <CustomModal
            :showModal="quickCloseValveModalVisiable"
            title="关阀"
            @cancel="handleQuickCloseValveCancel"
            @confirm="handleQuickCloseValveConfirm"
        >
            <view slot="content">
                <view class="modal-content-top">
                    <view class="title">你是否确定立即关闭此阀门？</view>
                    <view class="sup-title"> · 阀门关闭时请注意管道压力以防爆管。</view>
                </view>
            </view>
        </CustomModal>
    </view>
</template>

<script>
import CustomModal from '@/components/CustomModal/index'

export default {
    data() {
        return {
            showCalendar: false,
            startDate: '',
            endDate: '',
            checkAll: false,
            valveList: [
                {
                    name: '13-1-1',
                    battery: 98,
                    updateTime: '2024-05-15 12:45:56',
                    selected: true,
                    bOpen: true,
                    aOpen: false,
                    flow: 10,
                    open: 100,
                    timing: true,
                    signal: 78,
                    batteryNum: 75
                },
                {
                    name: '13-1-1',
                    battery: 98,
                    updateTime: '2024-05-15 12:45:56',
                    selected: false,
                    bOpen: false,
                    aOpen: false,
                    pressure: 120,
                    open: 0,
                    timing: false,
                    signal: 51,
                    batteryNum: 50
                },
                {
                    name: '13-1-1',
                    battery: 98,
                    updateTime: '2024-05-15 12:45:56',
                    selected: true,
                    bOpen: true,
                    aOpen: false,
                    pressure: 100,
                    open: 90,
                    timing: false,
                    signal: 26,
                    batteryNum: 25
                },
                {
                    name: '13-1-1',
                    battery: 98,
                    updateTime: '2024-05-15 12:45:56',
                    selected: true,
                    bOpen: true,
                    aOpen: false,
                    flow: 10,
                    open: 100,
                    timing: false,
                    signal: 25,
                    batteryNum: 25
                }
            ],
            loading: false,
            hasMore: true,
            page: 1,
            pageSize: 10,
            scrollViewHeight: 0,
            valveBatchShow: false,
            valveSettingData: {
                endParam: [],
                endNum: ''
            },
            valve1: '',
            showValve1: false,
            valveActions1: [
                {
                    name: 'A阀'
                },
                {
                    name: 'B阀'
                }
            ],
            valve2: '',
            showValve2: false,
            valveActions2: [
                {
                    name: '开度模式%'
                },
                {
                    name: '流量模式m³/h'
                }
            ],
            valve3: '',
            showValve3: false,
            valveActions3: Array.from({ length: 101 }, (_, i) => {
                return {
                    name: 100 - i // 从100开始倒序生成
                }
            }),
            valveSettingList: [
                {
                    name: '定时结束'
                },
                {
                    name: '定量结束'
                },
                {
                    name: '手动结束'
                }
            ],
            closeValveModalVisiable: false,
            quickOpenValveModalVisiable: false,
            quickCloseValveModalVisiable: false,
            stepValue: 0
        }
    },
    computed: {
        dateRangeText() {
            if (this.startDate === '' || this.endDate === '') {
                return '请选择日期区间'
            }
            return `${this.startDate} 至 ${this.endDate}`
        }
    },
    components: {
        CustomModal
    },
    methods: {
        // 打开日历
        handleOpenCalendar() {
            console.log('打开日历')
            this.showCalendar = true
        },

        // 关闭日历
        handleCloseCalendar() {
            this.showCalendar = false
            console.log('关闭日历')
        },

        // 确认日历选择
        handleConfirmCalendar(value) {
            this.showCalendar = false
            console.log('选择的日期区间:', value)
        },

        // 日期改变
        handleCalendarChange(e) {
            console.log('日期改变', e)
            this.startDate = e.range.before
            this.endDate = e.range.after
        },

        // 搜索
        handleSearch() {
            console.log('搜索')
        },

        // 全选
        handleCheckAll() {
            this.checkAll = !this.checkAll
            this.valveList.forEach((item) => {
                item.selected = this.checkAll
            })
        },

        // 选择阀门切换
        toggleSelect(idx) {
            this.valveList[idx].selected = !this.valveList[idx].selected
        },

        // 加载更多
        loadMore() {
            if (!this.hasMore || this.loading) {
                return
            }
            this.loading = true
            // 模拟API请求
            setTimeout(() => {
                // 模拟新数据
                const newData = [
                    {
                        name: `13-1-${this.page + 1}`,
                        battery: Math.floor(Math.random() * 100),
                        updateTime: '2024-05-15 12:45:56',
                        selected: false,
                        bOpen: Math.random() > 0.5,
                        aOpen: Math.random() > 0.5,
                        flow:
                            Math.random() > 0.5
                                ? Math.floor(Math.random() * 20)
                                : undefined,
                        pressure:
                            Math.random() > 0.5
                                ? Math.floor(Math.random() * 200)
                                : undefined,
                        open: Math.floor(Math.random() * 100),
                        timing: Math.random() > 0.5,
                        signal: Math.floor(Math.random() * 100),
                        batteryNum: Math.floor(Math.random() * 100)
                    },
                    {
                        name: `13-1-${this.page + 2}`,
                        battery: Math.floor(Math.random() * 100),
                        updateTime: '2024-05-15 12:45:56',
                        selected: false,
                        bOpen: Math.random() > 0.5,
                        aOpen: Math.random() > 0.5,
                        flow:
                            Math.random() > 0.5
                                ? Math.floor(Math.random() * 20)
                                : undefined,
                        pressure:
                            Math.random() > 0.5
                                ? Math.floor(Math.random() * 200)
                                : undefined,
                        open: Math.floor(Math.random() * 100),
                        timing: Math.random() > 0.5,
                        signal: Math.floor(Math.random() * 100),
                        batteryNum: Math.floor(Math.random() * 100)
                    }
                ]
                // 添加新数据到列表
                this.valveList.push(...newData)
                this.page++
                // 模拟只有3页数据
                this.hasMore = this.page < 3
                this.loading = false
            }, 1000)
        },

        // 打开批量控制
        handleOpenValveBatch() {
            this.valveBatchShow = true
        },

        // 关闭设置
        handleSettingCancel() {
            this.valveBatchShow = false
        },

        // 确认设置
        handleSettingCinfirm() {
            this.valveBatchShow = false
        },

        // 打开阀门1选择
        handleShowValve1() {
            this.showValve1 = true
        },

        // 阀门1选择
        handleChangeValve1(action) {
            this.valve1 = action.name
            this.showValve1 = false
        },

        // 打开阀门2选择
        handleShowValve2() {
            this.showValve2 = true
        },

        // 阀门2选择
        handleChangeValve2(action) {
            this.valve2 = action.name
            this.showValve2 = false
        },

        // 打开阀门3选择
        handleShowValve3() {
            this.showValve3 = true
        },

        // 阀门3选择
        handleChangeValve3(action) {
            this.valve3 = action.name
        },

        // 批量控制参数设置
        handleValveSetting(value) {
            this.$set(this.valveSettingData, 'endParam', value)
            console.log('value', this.valveSettingData)
        },

        // 批量控制参数设置
        handleValveSetting(value) {
            this.$set(this.valveSettingData, 'endParam', value)
            console.log('value', this.valveSettingData)
        },

        // 打开一键关阀弹窗
        handleOpenCloseValveModal() {
            this.closeValveModalVisiable = true
        },

        // 关闭一键关阀弹窗
        handlecloseValveModal() {
            this.closeValveModalVisiable = false
        },

        // 确认一键关阀
        handlecloseValveCinfirm() {
            this.closeValveModalVisiable = false
        },

        // 打开快捷开阀弹窗
        handleOpenQuickOpenValveModal(item) {
            console.log('打开快捷开阀弹窗', item)
            if (!item.bOpen) {
                this.quickOpenValveModalVisiable = true
            } else {
                this.quickCloseValveModalVisiable = true
            }
        },

        // 关闭快捷开阀弹窗
        handleQuickOpenValveCancel() {
            this.quickOpenValveModalVisiable = false
            this.stepValue = 0
        },

        // 快捷开阀确认
        handleQuickOpenValveConfirm() {
            this.quickOpenValveModalVisiable = false
            this.stepValue = 0
        },

        // 快捷开阀减
        handleSliderMinus() {
            console.log('快捷开阀减')
            this.stepValue--
            if (this.stepValue < 0) {
                this.stepValue = 0
            }
        },

        // 快捷开阀加
        handleSliderPlus() {
            console.log('快捷开阀加')
            this.stepValue++
            if (this.stepValue > 100) {
                this.stepValue = 100
            }
        },

        // 关闭快捷关阀弹窗
        handleQuickCloseValveCancel() {
            this.quickCloseValveModalVisiable = false
        },

        // 快捷关阀确认
        handleQuickCloseValveConfirm() {
            this.quickCloseValveModalVisiable = false
        },
    },
    mounted() {
        // 获取屏幕高度并计算scroll-view高度
        const systemInfo = uni.getSystemInfoSync()
        // 减去顶部固定区域高度（约110px）
        this.scrollViewHeight = systemInfo.windowHeight - 110
    }
}
</script>

<style lang="scss" scoped>
.valve-control-container {
    position: relative;
    min-height: 100vh;
    background: #f6f8ff;
    padding-bottom: 32rpx;
    .top-container {
        position: fixed;
        left: 0;
        top: 0;
        right: 0;
        z-index: 10;
        background: $u-blue;
        padding: 12px 10px;
        .date-input-box {
            width: 100%;
            ::v-deep .u-input {
                border-radius: 20px;
                background: #fff;
            }
            ::v-deep .iconfont {
                color: $u-blue;
            }
        }
        .button-group {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-top: 10px;
            .checkbox-button {
                display: flex;
                align-items: center;
                .check-text {
                    color: #fff;
                    margin-left: 10px;
                }
            }
            .action-btn {
                width: 80px;
                height: 32px;
                line-height: 32px;
                text-align: center;
                border: 1px solid #fff;
                border-radius: 20px;
                background: $u-blue;
                color: #fff;
                font-size: 14px;
            }
        }
    }
    .circle-check {
        width: 18px;
        height: 18px;
        border-radius: 50%;
        border: 1px solid #a0a7b3;
        background: #fff;
        position: relative;
    }
    .red {
        color: $u-red;
    }
    .orange {
        color: $u-orange;
    }
    .green {
        color: $u-green;
    }
    .blue {
        color: $u-blue;
    }
    .valve-list-scroll {
        position: relative;
        top: 110px;
        height: calc(100vh - 110px);
        .valve-list {
            padding: 15px;
            .valve-card {
                background: #fff;
                border-radius: 6px;
                margin-bottom: 12px;
                padding: 15px;
                .card-header {
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    padding-bottom: 6px;
                    border-bottom: 1px solid #efefef;
                    .card-header-left {
                        display: flex;
                        align-items: center;
                        .valve-avatar {
                            display: flex;
                            align-items: center;
                            justify-content: center;
                            width: 32px;
                            height: 32px;
                            border-radius: 50%;
                            background: #cecece;
                            margin-left: 10px;
                            margin-right: 10px;
                            image {
                                width: 80%;
                                height: 80%;
                                border-radius: 50%;
                            }
                        }
                        .valve-title-group {
                            flex: 1;
                            .valve-title {
                                font-size: 36rpx;
                                font-weight: 600;
                                color: #222;
                            }
                            .valve-status-row {
                                display: flex;
                                align-items: center;
                                font-size: 24rpx;
                                color: #a0a7b3;
                                margin-top: 4rpx;
                                .valve-singal {
                                    display: flex;
                                    align-items: center;
                                    .valve-singal-iconfont {
                                        margin-right: 5px;
                                    }
                                }
                                .battery {
                                    color: $u-green;
                                    margin-right: 10px;
                                }
                                .update-time {
                                    color: #a0a7b3;
                                }
                            }
                        }
                    }
                }
                .card-body {
                    margin-top: 6px;
                    .card-body-top {
                        display: flex;
                        justify-content: space-between;
                        align-items: center;
                    }
                    .card-body-content {
                        display: flex;
                        justify-content: space-around;
                        align-items: center;
                    }
                    .card-body-left {
                        display: flex;
                        flex-direction: column;
                        align-items: center;
                        justify-content: space-around;
                    }
                    .card-body-right {
                        display: flex;
                        flex-direction: column;
                        align-items: center;
                        justify-content: space-around;
                    }
                    .valve-btn {
                        display: flex;
                        align-items: center;
                        justify-content: center;
                        width: 80px;
                        height: 32px;
                        border-radius: 25px;
                        background: rgba(15, 64, 245, 0.08);
                        color: rgba(15, 64, 245, 0.5);
                        border: 1px solid rgba(15, 64, 245, 0.2);
                        font-size: 14px;
                        font-weight: 600;
                        margin-top: 5px;
                        margin-bottom: 20px;
                    }
                    .left-time-icon {
                        padding-left: 10px;
                        color: rgba(15, 64, 245, 0.5);
                    }
                    .close-btn {
                        display: flex;
                        justify-content: center;
                        align-items: center;
                        background: #efefef;
                        color: #a0a7b3;
                        border-radius: 25px;
                        border: 1px solid #cecece;
                        padding: 6px 14px;
                        font-size: 14px;
                        margin-top: 10px;
                    }
                    .valve-btn.active {
                        background: $u-blue;
                        color: #fff;
                    }
                    .valve-img {
                        width: 90px;
                        height: 50px;
                        image {
                            width: 100%;
                            height: 100%;
                        }
                    }
                    .info-item {
                        display: flex;
                        align-items: center;
                        font-size: 28rpx;
                        color: #222;
                        font-size: 13px;
                    }
                    .dot {
                        width: 16rpx;
                        height: 16rpx;
                        border-radius: 50%;
                        background: #a0a7b3;
                        margin-right: 8rpx;
                        border: 1px solid #bbb;
                        &.green {
                            background: $u-green;
                        }
                        &.blue {
                            background: $u-blue;
                        }
                        &.orange {
                            background: $u-orange;
                        }
                    }
                    .info-item-text {
                        color: #4f4f4f;
                        padding-right: 4px;
                    }
                }
            }
        }
        .load-more-container {
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 0 0 20px 0;
            .loading-text {
                display: flex;
                align-items: center;
                color: #999;
                font-size: 14px;
            }
        }
        .no-more-container {
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 0 0 20px 0;
            .no-more-text {
                color: #999;
                font-size: 14px;
            }
        }
    }
    .popup-container {
        margin-bottom: 35px;
        .popup-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 15px 17px;
            color: #fff;
            background: $u-blue;
            text {
                margin: auto;
            }
            ::v-deep .u-button {
                width: 80px;
                height: 32px !important;
                font-size: 16px;
                color: #fff;
                border-color: #fff;
                background: $u-blue;
                border-radius: 20px;
                font-size: 12px !important;
                margin: 0;
                & + .u-button {
                    margin-left: 10px;
                }
            }
        }
        .popup-body-item {
            padding: 0 15px;
            ::v-deep .u-checkbox {
                & > text {
                    color: #333 !important;
                    font-size: 14px !important;
                }
            }
            ::v-deep .u-checkbox-group {
                justify-content: space-around;
                padding: 20px 0;
            }
            &-input {
                margin-top: 13px;
            }

            ::v-deep .u-radio-group {
                justify-content: space-around;
                padding: 20px 0;
            }
        }
        .popup-title {
            height: 40px;
            line-height: 40px;
            background: #efefef;
            color: #333;
            font-size: 16px;
            text-align: center;
        }
        .popup-select-box {
            margin: 10px 10px 40px 10px;
            &-item {
                display: flex;
                justify-content: center;
                flex-direction: column;
                align-items: center;
                text {
                    font-size: 14px;
                    color: #101010;
                    padding-bottom: 10px;
                }
                .cover-container {
                    position: relative;
                    .cover-box {
                        position: absolute;
                        top: 0;
                        left: 0;
                        z-index: 10;
                        width: 100%;
                        height: 100%;
                    }
                }
            }
        }
        ::v-deep .u-border {
            border-color: #bbb !important;
        }
        ::v-deep .u-action-sheet__item-wrap__item__name {
            color: #333 !important;
        }
    }
    .modal-text {
        display: block;
        font-size: 14px;
        margin-top: 10px;
        text-align: center;
        color: #000;
    }
    .modal-content-top {
        border-bottom: 1px solid #efefef;
        padding-bottom: 10px;
        .title {
            font-size: 16px;
            color: #101010;
            text-align: center;
            margin-bottom: 10px;
        }
        .sup-title {
            font-size: 13px;
            color: #666;
            &.sup-title {
                margin-top: 5px;
            }
        }
    }
    .modal-content-center {
        margin-top: 10px;
        .slider-value-text {
            font-size: 14px;
            color: $u-blue;
            text-align: center;
        }
        .slider-group {
            display: flex;
            align-items: center;
            justify-content: center;
            .slider-minus {
                display: flex;
                align-items: center;
                justify-content: center;
                width: 40px;
                height: 40px;
                border-radius: 50%;
                background: rgba(15, 64, 245, 0.1);
                .iconfont {
                    font-size: 28px;
                    color: $u-blue;
                }
            }
            ::v-deep .u-slider {
                width: 200px;
            }
            .slider-plus {
                display: flex;
                align-items: center;
                justify-content: center;
                width: 40px;
                height: 40px;
                border-radius: 50%;
                background: rgba(15, 64, 245, 0.1);
                .iconfont {
                    font-size: 28px;
                    color: $u-blue;
                }
            }
        }
    }
    ::v-deep .uni-calendar__weeks-item {
        .uni-calendar-item__weeks-box {
            .uni-calendar-item--isDay-text {
                color: $u-blue;
            }
            &.uni-calendar-item--isDay {
                background: $u-blue;
                .uni-calendar-item__weeks-box-text {
                    color: #fff;
                    background: $u-blue;
                    &.uni-calendar-item--isDay-text {
                        color: #fff;
                    }
                }
                .uni-calendar-item__weeks-lunar-text {
                    color: #fff;
                    background: $u-blue;
                }
            }
            &.uni-calendar-item--before-checked {
                color: #fff;
                background: $u-blue;
                .uni-calendar-item__weeks-box-text {
                    color: #fff;
                    background: $u-blue;
                }
                .uni-calendar-item__weeks-lunar-text {
                    color: #fff;
                    background: $u-blue;
                }
            }
            &.uni-calendar-item--after-checked {
                color: #fff;
                background: $u-blue;
            }
            &.uni-calendar-item--multiple {
                color: #fff;
                background: $u-blue;
                .uni-calendar-item__weeks-box-text {
                    color: #fff;
                    background: $u-blue;
                }
                .uni-calendar-item__weeks-lunar-text {
                    color: #fff;
                    background: $u-blue;
                }
            }
        }
    }
    .calendar-button-group {
        display: flex;
        justify-content: center;
        align-items: center;
        margin: 10px 0 15px 0;
        border-top: 1px solid #efefef;
        padding-top: 10px;
        .calendar-button {
            &.cancel-button {
                margin-right: 10px;
                ::v-deep .u-button {
                    width: 100px;
                    height: 32px;
                    border-radius: 25px;
                }
            }
            &.confirm-button {
                ::v-deep .u-button {
                    width: 100px;
                    height: 32px;
                    border-radius: 25px;
                }
            }
        }
    }
}
</style>
