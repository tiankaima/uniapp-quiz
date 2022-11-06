<template>
    <view class="content">
        <progress class="progress-box" :percent="progress" activeColor="#10AEFF" stroke-width="3" show-info></progress>
        <uni-title class="title" type="h2" :title="question"></uni-title>
        <!-- <text class="title">{{ question }}</text> -->
        <form @submit="sumbit_form" class="form-block">
            <view class="options-block">
                <view v-if="type == 0 && !checked">
                    <radio-group name="ans">
                        <view v-for="(option, index) in options" :key="index">
                            <view class="options-sub-block">
                                <label>
                                    <radio :value="option.id"></radio>
                                    <text class="option">{{ option.refer + ". " + option.name }}</text>
                                </label>
                            </view>
                        </view>
                    </radio-group>
                </view>
                <view v-if="type == 1 && !checked">
                    <checkbox-group name="ans">
                        <view v-for="(option, index) in options" :key="index">
                            <view class="options-sub-block">
                                <label>
                                    <checkbox :value="option.id"></checkbox>
                                    <text class="option">{{ option.refer + ". " + option.name }}</text>
                                </label>
                            </view>
                        </view>
                    </checkbox-group>
                </view>
                <view v-if="checked">
                    <view v-for="(option, index) in options" :key="index">
                        <view class="options-sub-block">
                            <text class="option" :style="color_option(option.id)">{{ option.refer + ". " + option.name
                            }}</text>
                        </view>
                    </view>
                </view>
                <view class="message">
                    <text :style="hintTextStyle">{{ message }}</text>
                </view>
            </view>
            <view class="buttons">
                <button class="navigate_button" type="normal" @click="onclick_previous">上一题</button>
                <button class="navigate_button" :disabled="checked" type="primary" form-type="submit">提交</button>
                <button class="navigate_button" type="primary" :disabled="!checked" @click="onclick_next">下一题</button>
            </view>
        </form>
    </view>
</template>

<script>
import question_json from '../../static/data.json'
var _question = null
let question_list = JSON.parse(JSON.stringify(question_json))
export default {
    data() {
        return {
            progress: 0,
            question: "*",
            options: [],
            // type == 0: single choice, type == 1: multiple choice
            type: 0,
            // the following data is only required if checked == true to record the answer
            answers: [],
            checked: false,
            message: "",
            correct: false,
        }
    },
    onReady: function () {
    },
    onLoad: function (options) {
        let index = null
        // try serialize json file to object
        if (options.id == "null" || options.id == undefined) {
            index = 0
            _question = question_list[0]
        } else {
            // given an id, search for element that have identical id in json file
            index = question_list.findIndex(element => element.id == options.id)
            _question = question_list[index]
            // fallback if the id isn't found
            if (_question == undefined) {
                _question = question_list[0]
            }
        }
        this.progress = parseInt((index + 1) / question_list.length * 100)
        this.question = (index == null ? "?" : index + 1) + '. ' + _question.name
        if (_question.answer.length > 1) {
            // multiple-choice question
            this.type = 1
        }
        this.options = _question.options
        this.answers = []
        // now, try to load every answers from localStrorage
        // console.log(uni.getStorageSync('answers'))
        try {
            const answers_list = JSON.parse(uni.getStorageSync('answers'))
            for (let i = 0; i < answers_list.length; i++) {
                if (answers_list[i].id == _question.id) {
                    this.answers = answers_list[i].answers
                    if (this.answers.length) {
                        this.checked = true
                    }
                    break
                }
            }
        } catch (e) {
            console.warn(e)
            uni.setStorageSync('answers', '[]')
        }

        this.correct = this.checked && this.answers.sort().equals(_question.answer.sort())
        if (this.answers.sort().equals(_question.answer.sort())) {
            this.message = "回答正确，"
        }
        if (this.checked) {
            var correct_refer = ""
            for (let i = 0; i < _question.options.length; i++) {
                if (_question.answer.includes(_question.options[i].id)) {
                    correct_refer += _question.options[i].refer
                }
            }
            this.message = "正确答案是：" + correct_refer
        }
    },
    computed: {
        hintTextStyle() {
            return {
                color: this.correct ? '#00ff00' : '#ff0000'
            }
        },
    },
    methods: {
        onclick_previous() {
            // check if the current question is the first one
            if (_question.id == question_list[0].id) {
                // navigate back to home page:
                uni.redirectTo({
                    url: '/pages/index/index'
                })
                return
            }
            let previous_question_id = question_list[question_list.findIndex(element => element.id == _question.id) - 1].id
            uni.redirectTo({
                url: '/pages/quiz/quiz?id=' + previous_question_id,
            })
        },
        onclick_next() {
            // check if the current question is the last one
            if (_question.id == question_list[question_list.length - 1].id) {
                // bring user to the result page
                uni.navigateTo({
                    url: '/pages/result/result'
                })
                return
            }
            let next_question_id = question_list[question_list.findIndex(element => element.id == _question.id) + 1].id
            uni.redirectTo({
                url: '/pages/quiz/quiz?id=' + next_question_id,
            })
        },
        sumbit_form(e) {
            if (this.checked) {
                // if the question has been answered, then we don't need to do anything
                // also this shouldn't happen since submit button is disabled when checked == true
                return
            }
            // determine the answer from the form and this.type
            if (this.type == 0) {
                // single choice
                if (e.detail.value.ans == undefined) {
                    return
                }
                this.answers = [e.detail.value.ans]
            } else if (this.type == 1) {
                // multiple choice
                if (e.detail.value.ans == undefined || e.detail.value.ans.length == 0) {
                    return
                }
                this.answers = e.detail.value.ans
            }
            // console.log(e);
            // debug purpose:
            // this.checked = true

            // save the results to localStrorage
            try {
                const answers_list = JSON.parse(uni.getStorageSync('answers'))
                // console.log(answers_list)
                let index = answers_list.findIndex(element => element.id == _question.id)
                if (index == -1) {
                    // if the question hasn't been answered before, then add it to the list
                    answers_list.push({
                        id: _question.id,
                        answers: this.answers
                    })
                } else {
                    // if the question has been answered before, then update the answer
                    // however, this shouldn't happen since submit button is disabled when checked == true(hence the question has been answered)
                    answers_list[index].answers = this.answers
                }
                uni.setStorageSync('answers', JSON.stringify(answers_list))
            } catch (e) {
                console.warn(e)
                // debug purpose:
                uni.clearStorageSync()
                // console.log('!')
            }
            // try reloading the page to ignore all refresh problems
            uni.redirectTo({
                url: '/pages/quiz/quiz?id=' + _question.id,
            })
        },
        color_option(id) {
            if (this.answers == undefined || _question.answer == undefined) {
                // something not loaded in AppCycle, this should not be a concern since it would automatically reload the page after all data is loaded
                return {
                    color: '#000000'
                }
            }
            if (_question.answer.includes(id)) {
                if (this.answers.includes(id)) {
                    return {
                        color: '#00ff00'
                    }
                }
                return {
                    color: '#9ed99d'
                }
            }
            if (this.answers.includes(id)) {
                return {
                    color: '#ff0000'
                }
            }
            return {
                color: "#000000"
            }
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


.title {
    text-indent: 50rpx;
    margin: 30rpx;
    align-content: center;
    font-size: 36rpx;
    line-height: 150%;
    height: 500rpx;
    /* background-color: lightblue; */
    /* background-clip: border-box; */
}

.form-block {
    width: 80%;
    /* height: 400rpx; */
    /* margin: 60rpx; */
}

.options-block {
    /* width: 80%; */
    height: 400rpx;
}

.options-sub-block {
    /* width: 80%; */
    /* align-content: center; */
    margin-top: 20rpx;
    /* margin: 20rpx; */
}

.option {
    font-size: 30rpx;
    /* margin: 120rpx; */
}

.message {
    font-size: 30rpx;
    /* color: #ff0000; */
    margin-top: 20rpx;
}

.buttons {
    display: flex;
    margin: 30rpx;
}

.navigate_button {
    font-size: 30rpx;
    margin: 15 rpx;
}

.progress-box {
    /* display: flex; */
    height: 50rpx;
    width: 90%;
    margin-top: 10rpx;
    /* background-color: brown; */
}
</style>
