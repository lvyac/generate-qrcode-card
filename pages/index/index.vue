<template>
    <view>
        <view class="container">
            <view class="content">
                <block v-if="isUpload === false">
                    <view class="upload" @tap="uploadImg">+</view>
                    <view class="upload-text">请上传核酸二维码载图</view>
                </block>
        
                <view v-else class="qrocde-image" @tap="uploadImg">
                    <uqrcode ref="uqrcode" canvas-id="qrcode" :options="options" :auto="true" :value="qrContent"></uqrcode>
                </view>
                <view class="input">
                    <uni-easyinput v-model="name" placeholder="姓名"></uni-easyinput>
                </view>

                <view class="input">
                    <button type="primary" :disabled="disabled" :loading="loading" @tap="build">生成吊牌</button>
                </view>
                <view class="help">
                    微信小程序搜索“采检登记”，点击底下菜单栏“预登记记录”，打开“采样二维码”截屏上传，姓名为选填，点击“生成吊牌”按钮，生成后长按保存到手机相册。不用担心用户信息，所有内容均在手机完成，不会上传至服务器。
                </view>
            </view>
            
        </view>
        <view>
            <c-build ref="bqrcode" :qrcode="qrcodePth" :nickname="name"></c-build>
        </view>
    </view>
</template>

<script>
    import QrCode from '../../node_modules/qrcode-decoder/dist/index.js';
    import uqrcode from '../../uni_modules/Sansnn-uQRCode/components/uqrcode/uqrcode.vue'
    import cBuild from '@/components/c-build.vue'
    export default {
        components: {
            uqrcode,
            cBuild
        },
        data() {
            return {
                isUpload: false,
                imageTemp: "",
                qrContent: "",
                name: '',
                class0: '',
                disabled: false,
                loading: false,
                qrcodePth: ""
            }
        },
        onLoad() {

        },

        methods: {
            // 获取上传状态
            uploadImg() {
                try{
                    uni.chooseImage({
                        count: 1,
                        sourceType: ['album'],
                        success: (chooseImageRes) => {
                            const tempFilePaths = chooseImageRes.tempFilePaths;
                            this.imageTemp = tempFilePaths[0]
                            this.isUpload = true
                            const qr = new QrCode()
                            qr.decodeFromImage(tempFilePaths[0]).then(res => {
                                if(!res.data) {
                                    this.isUpload=false
                                    uni.showModal({
                                        showCancel: false,
                                        content: "二维解析失败，请检查是否‘采检登记’小程序里的‘采样二维码'"
                                    })
                                    return false
                                }
                                this.qrContent = res.data
                            }).catch(err => {
                                console.log(err)
                            })
                        }
                    })
                }catch(e){
                    uni.showModal({
                        showCancel: false,
                        content: "二维解析失败，请检查是否‘采检登记’小程序里的‘采样二维码'"
                    })
                }
            },
            build() {
                try{
                    if(this.isUpload === false || !this.qrContent) {
                        uni.showToast({
                            title: "请先上传二维码",
                            icon:'error'
                        })
                    }
                    this.$refs.uqrcode.toTempFilePath({
                      success: res => {
                        this.qrcodePth = res.tempFilePath
                        this.$refs.bqrcode.build(res.tempFilePath)
                      }
                    });
                }catch(e){
                    uni.showModal({
                        showCancel: false,
                        content: "二维码解析失败请重新上传"
                    })
                }
            }
        }
    }
</script>

<style lang="scss">
    page{
        width: 100%;
        height: 125vh;
        overflow: hidden;
    }
    .container {
        background: url('https://vkceyugu.cdn.bspapp.com/VKCEYUGU-7791bacc-1882-410d-ab0b-886dd3e695aa/80be98c5-bf58-4c78-ae29-4d134da83196.png')no-repeat #163A8A;
        background-size: 100% auto;
        padding: 0;
        margin: 0;
        height: 125vh;
        padding-top: 550rpx;
        overflow: hidden;

        .content {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;

            .upload {
                $size: 300rpx;
                width: $size;
                border: 3px solid #fff;
                border-radius: 8px;
                height: $size;
                color: #fff;
                font-size: 180rpx;
                text-align: center;
                line-height: $size - 40rpx;
            }

            .upload-text {
                font-size: 32rpx;
                color: #fff;
                margin-top: 10rpx;
            }

            .input {
                width: 600rpx;
                margin-top: 40px;
            }
            .qrocde-image{
                border: 3px solid #fff;
                background-color: #fff;
                padding: 15px;
                box-sizing: border-box;
            }
            .help{
                padding: 40rpx;
                box-sizing: border-box;
                font-size: 32rpx;
                line-height: 1.75em;
                color: #fff;
            }
        }
    }
</style>
