<template>
    <view>
        <canvas :id="canvasId" :canvas-id="canvasId" :style="{width: cWidth+'px',height: cHeight+'px'}"></canvas>
    </view>
</template>

<script>
    export default {
        name: "c-build",
        props: {
            width: {
                type: String,
                default: '400px'
            },
            height: {
                type: String,
                default: '600px'
            },
            qrcode: String,
            nickname: String,
            uclass: String
        },
        data() {
            return {
                context: "",
                bg: "",
                cWidth: "",
                cHeight: "",
                canvasId: "canvasQrcode",
            };
        },
        mounted() {
            const context = uni.createCanvasContext(this.canvasId, this)
            this.context = context
            uni.getImageInfo({
                src: '/static/bg.png',
                success: (res) => {
                    this.bg = res.path
                    this.cWidth = res.width
                    this.cHeight = res.height
                },
                fail: (e) => {
                    uni.showToast({
                        title: "模板载入失败",
                        icon: false
                    })
                }
            })
        },
        methods: {
            async build(qrcode) {
                const context = this.context
                qrcode = await this.imgInfo(qrcode)

                //背景图
                context.drawImage(this.bg, 0, 0, this.cWidth, this.cHeight, 0, 0, this.cWidth, this.cHeight)

                const qrcodeW = this.cWidth / 2

                //距形
                context.beginPath()
                context.rect(qrcodeW / 2 - 30, qrcodeW + 220, qrcodeW + 60, qrcodeW + 60)
                context.fillStyle = "rgb(255, 255, 255)"
                context.fill()
                context.closePath()
                //二维码
                context.drawImage(qrcode, this.cWidth / 4, qrcodeW + 250, qrcodeW, qrcodeW)

                if (this.nickname) {
                    //姓名
                    this.fillText(this.nickname, qrcodeW * 2 + 400, '80px')
                }
                if (this.uclass) {
                    //班级
                    this.fillText(this.uclass, qrcodeW * 2 + 580, '72px')
                }

                this.fillText('核酸检测', this.cHeight - 310, '172px')
                context.draw(false, ()=>{
                    uni.canvasToTempFilePath({
                        width: this.cWidth,
                        height: this.cHeight,
                        canvasId: this.canvasId,
                        success: (res)=> {
                            uni.showModal({
                                showCancel: false,
                                content:"生成完毕，长按保存至相册",
                                success: (e) => {
                                    uni.previewImage({
                                        urls:[res.tempFilePath],
                                        fail: (e) => {
                                            uni.showToast({
                                                title: e.errMsg || '图片预览失败',
                                                icon:"error"
                                            })
                                        }
                                    })
                                }
                            })
                            
                        },
                        fail: (res)=> {
                            console.log(res)
                        }
                    }, this);
                })

            },
            imgInfo(src) {
                return new Promise((reslove, reject) => {
                    uni.getImageInfo({
                        src: src,
                        success: (res) => {
                            reslove(res.path)
                        },
                        fail: (e) => {
                            reject(e)
                        }
                    })
                })
            },
            fillText(text, y, font = '72px', ) {
                const canvas = this.context
                canvas.beginPath()
                canvas.fillStyle = '#fff';
                canvas.strokeStyle = '#00ff00'; //设置笔触的颜色
                canvas.font = `bold ${font} '黑体','宋体'`; //设置字体
                let textW = canvas.measureText(text).width
                canvas.fillText(text, this.cWidth / 2 - textW / 2, y)
                canvas.fill()
                canvas.closePath()
            }
        }
    }
</script>

<style lang="scss">

</style>
