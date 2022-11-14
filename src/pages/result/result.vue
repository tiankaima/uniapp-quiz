<template>
    <view class="content">
        <uni-title class="title" type="h2" :title="title"></uni-title>
        <uni-title class="hint" type="h4" title="可分享本页截图作为成绩凭证"></uni-title>
        <view class="canvas">
            <canvas style="width: 300px; height: 300px;" canvas-id="avatar" id="avatar"></canvas>
        </view>
        <uni-title type="h5" :title="calc_uuid()"></uni-title>
        <text class="taffy">关注永雏塔菲喵，关注永雏塔菲谢谢喵</text>
    </view>
</template>

<script>
import question_json from '../../static/data.json'
import sha1 from '../../static/sha1.js'
let question_list = JSON.parse(JSON.stringify(question_json))
var percent = 0.0
export default {
    data() {
        return {
            title: "result",
        }
    },
    onReady() {
        // Modified from https://github.com/viveketic/gavatar, Apache 2.0 License
        // Great thanks to Vivek Etić for these amazing code
        function rand(x) {
            var z = (x << 13) ^ x;
            return ((1.0 - ((z * (z * z * 15731 + 789221) + 1376312589) & 0x7fffffff) / 1073741824.0) + 1.0) / 2.0;
        }
        let blockSize = 50;
        let avatarMargin = 20;
        let avatarCanvasSize = 290; //5*50 + 2*20
        var ctx = uni.createCanvasContext('avatar')
        var hex = Math.floor(Math.random() * 16777215).toString(16);;
        var num = Math.max(0.25, percent); // set with a minimum of 25% to prevet (0) => empty drawing.
        var seed = 50;

        ctx.setFillStyle("#ffffff");
        ctx.fillRect(0, 0, avatarCanvasSize, avatarCanvasSize);
        ctx.setStrokeStyle("f0f0f0")
        ctx.setLineWidth(5)
        ctx.rect(0, 0, avatarCanvasSize, avatarCanvasSize)
        ctx.stroke()

        for (var x = 0; x < 3; x++) {
            for (var y = 0; y < 5; y++) {
                var r = rand(parseInt(seed) + x * 3 + y);
                if (r < num) {
                    ctx.fillStyle = "#" + hex;
                    ctx.fillRect(x * blockSize + avatarMargin, y * blockSize + avatarMargin, blockSize, blockSize);
                    ctx.fillStyle = "#" + hex;
                    ctx.fillRect((5 - 1 - x) * blockSize + avatarMargin, y * blockSize + avatarMargin, blockSize, blockSize);
                }
            }
        }
        ctx.draw()
    },
    onLoad() {
        var count = 0;
        try {
            const answers_list = JSON.parse(uni.getStorageSync('answers'))
            for (let i = 0; i < question_list.length; i++) {
                let answer = answers_list.find(element => element.id == question_list[i].id)
                if (answer == undefined) continue;
                if (answer.answers.length == 0) continue;
                if (question_list[i].answer.sort().equals(answer.answers.sort())) count++;
            }
        } catch (e) {
            console.warn(e)
            uni.setStorageSync('answers', '[]')
        }
        this.title = "您的得分为：" + count + "/" + question_list.length
        percent = count / question_list.length
    },
    methods: {
        calc_uuid: () => {
            return sha1(uni.getSystemInfoSync().deviceId)
        }
    },
}
</script>

<style>
.content {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.hint {
    margin-top: 200rpx;
}

.canvas {
    height: 300px;
    width: 300px;
}

.taffy {
    color: rgb(245, 245, 245);
    font-size: 10px;
}
</style>
