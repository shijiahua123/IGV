<template>
    <view class="valve-control-bg">
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
            <u-calendar
                :show="showCalendar"
                mode="range"
                startText="开始"
                endText="结束"
                color="#0F40F5"
                @confirm="handleConfirmCalendar"
                @close="handleCloseCalendar"
            >
            </u-calendar>
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
                <view class="action-btn">一键关阀</view>
                <view class="action-btn">批量控制</view>
                <view class="action-btn">解除时控</view>
            </view>
        </view>

        <!-- 阀门卡片列表 -->
        <view class="valve-list">
            <view
                v-for="(item, idx) in valveList"
                :key="idx"
                class="valve-card"
                :class="{ selected: item.selected }"
            >
                <view class="card-header">
                    <view @click="toggleSelect(idx)">
                        <u-icon
                            v-if="item.selected"
                            name="checkmark-circle-fill"
                            size="20"
                            color="#0F40F5"
                        ></u-icon>
                        <view v-else class="circle-check"></view>
                    </view>
                    <view class="valve-avatar"></view>
                    <view class="valve-title-group">
                        <text class="valve-title">{{ item.name }}</text>
                        <view class="valve-status-row">
                            <u-icon name="wifi" size="16" color="#FF9900" />
                            <text class="battery">{{ item.battery }}%</text>
                            <text class="update-time"
                                >{{ item.updateTime }} 更新</text
                            >
                        </view>
                    </view>
                </view>
                <view class="card-body">
                    <view class="valve-btn left" :class="{ active: item.bOpen }"
                        >B阀一开</view
                    >
                    <view class="valve-img"></view>
                    <view
                        class="valve-btn right"
                        :class="{ active: item.aOpen }"
                        >A阀一开</view
                    >
                </view>
                <view class="card-info-row">
                    <view class="info-item">
                        <view
                            class="dot"
                            :class="{
                                green: item.flow > 0,
                                blue: item.pressure > 0
                            }"
                        ></view>
                        <text v-if="item.flow !== undefined"
                            >实时流量: {{ item.flow }}</text
                        >
                        <text v-else>实时压力: {{ item.pressure }}</text>
                    </view>
                    <view class="close-btn">关阀</view>
                    <view class="info-item">
                        <view class="dot orange"></view>
                        <text>开度: {{ item.open }}</text>
                        <u-icon
                            v-if="item.timing"
                            name="clock"
                            size="16"
                            color="#FF9900"
                        />
                    </view>
                </view>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            showCalendar: false,
            checkAll: false,
            selectedRange: ['2024-10-01', '2024-10-31'],
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
                    timing: true
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
                    timing: false
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
                    timing: false
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
                    timing: false
                }
            ]
        }
    },
    computed: {
        dateRangeText() {
            return `${this.selectedRange[0]} 至 ${this.selectedRange[this.selectedRange.length - 1]}`
        }
    },
    methods: {
        handleOpenCalendar() {
            console.log('打开日历')
            this.showCalendar = true
        },
        handleConfirmCalendar(value) {
            this.selectedRange = value
            this.showCalendar = false
            console.log('选择的日期区间:', value)
        },
        handleCloseCalendar() {
            this.showCalendar = false
            console.log('关闭日历')
        },
        handleSearch() {
            console.log('搜索')
        },
        handleCheckAll() {
            this.checkAll = !this.checkAll
        },
        onDateChange(e) {
            this.selectedRange = e
        },
        toggleSelect(idx) {
            this.valveList[idx].selected = !this.valveList[idx].selected
        }
    }
}
</script>

<style lang="scss" scoped>
.valve-control-bg {
    min-height: 100vh;
    background: #f6f8ff;
    padding-bottom: 32rpx;
}
.title {
    font-size: 40rpx;
    font-weight: 600;
    color: #fff;
    background: $u-main-color;
    text-align: center;
    padding: 40rpx 0 24rpx 0;
}
.top-container {
    //   display: flex;
    //   align-items: center;
    background: $u-main-color;
    padding: 12px 10px;
}
.date-input-box {
    width: 100%;
    ::v-deep .u-input {
        border-radius: 20px;
        background: #fff;
    }
    ::v-deep .iconfont {
        color: $u-main-color;
    }
}
.button-group {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 10px;
}
.checkbox-button {
    display: flex;
    align-items: center;
}
.circle-check {
    width: 18px;
    height: 18px;
    border-radius: 50%;
    border: 1px solid #a0a7b3;
    background: #fff;
    position: relative;
}
.check-text {
    color: #fff;
    margin-left: 10px;
}
.action-btn {
    width: 80px;
    height: 32px;
    line-height: 32px;
    text-align: center;
    border: 1px solid #fff;
    border-radius: 20px;
    background: $u-main-color;
    color: #fff;
    font-size: 14px;
}
.date-picker-box {
    flex: 1;
    background: #fff;
    border-radius: 40rpx;
    padding: 16rpx 32rpx;
    font-size: 32rpx;
    color: #a0a7b3;
    margin-right: 24rpx;
    display: flex;
    align-items: center;
}
.search-icon {
    background: #fff;
    border-radius: 50%;
    padding: 12rpx;
}
.button-row {
    display: flex;
    align-items: center;
    background: #fff;
    border-radius: 40rpx;
    margin: -32rpx 32rpx 24rpx 32rpx;
    box-shadow: 0 8rpx 32rpx 0 #e6efff;
    padding: 0 24rpx;
    height: 80rpx;
}

.all-select {
    font-size: 32rpx;
    color: #222;
    margin-right: 32rpx;
}

.valve-list {
    padding: 15px;
}
.valve-card {
    background: #fff;
    border-radius: 6px;
    margin-bottom: 12px;
    padding: 15px;
}
.card-header {
    display: flex;
    align-items: center;
    padding-bottom: 6px;
    border-bottom: 1px solid #efefef;
}
.valve-avatar {
    width: 32px;
    height: 32px;
    background: #e6efff;
    border-radius: 50%;
    margin-left: 10px;
    margin-right: 10px;
}
.valve-title-group {
    flex: 1;
}
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
}
.battery {
    color: #4caf50;
    margin: 0 12rpx;
}
.update-time {
    color: #a0a7b3;
}
.card-body {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-top: 14px;
}
.valve-btn {
    width: 160rpx;
    height: 56rpx;
    border-radius: 32rpx;
    background: #f6f8ff;
    color: $u-main-color;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 28rpx;
    font-weight: 600;
}
.valve-btn.active {
    background: $u-main-color;
    color: #fff;
}
.valve-img {
    width: 120rpx;
    height: 64rpx;
    background: #d9d9d9;
    border-radius: 16rpx;
}
.card-info-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-top: 8rpx;
}
.info-item {
    display: flex;
    align-items: center;
    font-size: 28rpx;
    color: #222;
}
.dot {
    width: 16rpx;
    height: 16rpx;
    border-radius: 50%;
    background: #a0a7b3;
    margin-right: 8rpx;
}
.dot.green {
    background: #4caf50;
}
.dot.blue {
    background: $u-main-color;
}
.dot.orange {
    background: #ff9900;
}
.close-btn {
    background: #f6f8ff;
    color: #a0a7b3;
    border-radius: 32rpx;
    padding: 0 32rpx;
    height: 48rpx;
    display: flex;
    align-items: center;
    font-size: 28rpx;
    margin: 0 16rpx;
}
</style>
