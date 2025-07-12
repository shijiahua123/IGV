<template>
    <view class="custom-modal">
        <u-modal 
            :show="visiable" 
            :title="title" 
            :showCancelButton="showCancelButton"
            cancelText="取消"
            confirmText="确定"
            @cancel="handleCancel"
            @confirm="handleConfirm"
        >
            <view class="slot-content">
                <slot name="content"></slot>
            </view>
        </u-modal>
    </view>
</template>
<script>
	export default {
        name: 'CustomModal',
        props: {
            showModal: {
                type: Boolean,
                default: false,
            },
            title: {
                type: String,
                default: '',
            },
            showCancelButton: {
                type: Boolean,
                default: true,
            }
        },
        computed: {
            visiable: {
                get() {
                    return this.showModal
                },
                set(newVal) {
                    return newVal
                }
            }
        },
        options: {
			styleIsolation: 'shared'
		},
		data() {
			return {
   
			}
		},
		methods: {
            handleCancel() {
                this.visiable = false
                this.$emit('cancel')
            },

            handleConfirm() {
                this.visiable = false
                this.$emit('confirm')
            }

		},
	}
</script>

<style lang="scss" scoped>
    .custom-modal {
        ::v-deep .u-modal {
            border-radius: 10px;
            .u-modal__title {
                font-size: 16px;
                font-weight: 500 !important;
                color: #000;
                height: 40px;
                line-height: 40px;
                padding-top: 0 !important;
            }
            .u-modal__content {
                border-top: 1px solid #EFEFEF;
                margin: 0;
                min-height: 60px;
                display: flex;
                align-items: center;
                justify-content: center;
                &__text {
                    font-size: 18px;
                    text-align: center;
                }
            }
            .u-line {
                display: none;
            }
            .u-modal__button-group {
                justify-content: center;
                padding-bottom: 20px;
                .u-modal__button-group__wrapper {
                    display: block !important;
                    flex: none !important;
                    width: 120px;
                    height: 36px !important;
                    line-height: 36px !important;
                    border-radius: 25px;
                    text-align: center;
                    &--cancel {
                        background-color: #f6f6f6;
                        border: 1px solid $u-blue;
                        margin-right: 20px;
                        .u-modal__button-group__wrapper__text {
                            color: $u-blue !important;
                            font-size: 14px !important;
                        }
                    }
                    &--confirm {
                        background-color: $u-blue;
                        .u-modal__button-group__wrapper__text {
                            color: #fff !important;
                            font-size: 14px !important;
                        }
                    }
                }
            }
        }
        .slot-content {
            width: 100%;
        }
    }
</style>