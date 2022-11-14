<template>
    <view class="content">
        <progress class="progress-box" :percent="progress" activeColor="#10AEFF" stroke-width="3" show-info></progress>
        <uni-title class="title" type="h2" :title="question_content"></uni-title>
        <form @submit="sumbit_form" class="form-block">
            <view class="options-block">
                <view v-if="question_type == 0 && !answered">
                    <radio-group name="ans">
                        <view v-for="(option, index) in question_options" :key="index">
                            <view class="options-sub-block">
                                <label>
                                    <radio :value="option.id"></radio>
                                    <text class="option">{{ option.refer + ". " + option.name }}</text>
                                </label>
                            </view>
                        </view>
                    </radio-group>
                </view>
                <view v-if="question_type == 1 && !answered">
                    <checkbox-group name="ans">
                        <view v-for="(option, index) in question_options" :key="index">
                            <view class="options-sub-block">
                                <label>
                                    <checkbox :value="option.id"></checkbox>
                                    <text class="option">{{ option.refer + ". " + option.name }}</text>
                                </label>
                            </view>
                        </view>
                    </checkbox-group>
                </view>
                <view v-if="answered">
                    <view v-for="(option, index) in question_options" :key="index">
                        <view class="options-sub-block">
                            <text class="option" :style="color_option(option.id)">{{ option.refer + ". " + option.name }}</text>
                        </view>
                    </view>
                </view>
                <view class="message">
                    <text :style="{color: this.previous_answers_is_correct ? '#00ff00' : '#ff0000'}">{{ message_to_show }}</text>
                </view>
            </view>
            <view class="buttons">
                <button class="navigate_button" type="normal" @click="onclick_previous">上一题</button>
                <button class="navigate_button" :disabled="answered" type="primary" form-type="submit">提交</button>
                <button class="navigate_button" type="primary" :disabled="!answered" @click="onclick_next">下一题</button>
            </view>
        </form>
    </view>
</template>

<script>
import loadeed_question_json from "../../static/data.json";
var current_question = null; // current question
let complete_question_list = JSON.parse(JSON.stringify(loadeed_question_json));
export default {
    // required by uni-app
    data() {
        return {
            progress: 0, // percentage in Int(0-100)
            question_content: "*",
            question_options: [],
            question_type: 0, // 0: single choice; 1: multiple choice
            answered: false, // whether the question has been answered

            // assert(answered==true)
            previous_answers: [],
            message_to_show: "",
            previous_answers_is_correct: false,
        };
    },
    onReady: function () {},
    onLoad: function (options) {
        let current_question_index = null;
        // try serialize json file to object
        if (options.id == "null" || options.id == undefined) {
            current_question_index = 0;
            current_question = complete_question_list[0];
        } else {
            // given an id, search for element that have identical id in json file
            current_question_index = complete_question_list.findIndex((element) => element.id == options.id);
            current_question = complete_question_list[current_question_index];
            // fallback if the id isn't found
            if (current_question == undefined) {
                current_question_index = 0;
                current_question = complete_question_list[0];
            }
        }
        this.progress = parseInt(((current_question_index + 1) / complete_question_list.length) * 100);
        this.question_content = (current_question_index == null ? "?" : current_question_index + 1) + ". " + current_question.name;
        if (current_question.answer.length > 1) {
            // multiple-choice question
            this.question_type = 1;
        }
        this.question_options = current_question.options;
        // now, try to load every answers from localStrorage
        this.previous_answers = [];
        try {
            const answers_list = JSON.parse(uni.getStorageSync("answers"));
            for (let i = 0; i < answers_list.length; i++) {
                if (answers_list[i].id == current_question.id) {
                    this.previous_answers = answers_list[i].answers;
                    if (this.previous_answers.length) {
                        this.answered = true;
                    }
                    break;
                }
            }
        } catch (e) {
            console.warn(e);
            uni.setStorageSync("answers", "[]");
        }

        this.previous_answers_is_correct = this.answered && this.previous_answers.sort().equals(current_question.answer.sort());
        if (this.previous_answers.sort().equals(current_question.answer.sort())) {
            this.message_to_show = "回答正确，";
        }
        if (this.answered) {
            var correct_refers = "";
            for (let i = 0; i < current_question.options.length; i++) {
                if (current_question.answer.includes(current_question.options[i].id)) {
                    correct_refers += current_question.options[i].refer;
                }
            }
            this.message_to_show = "正确答案是：" + correct_refers;
        }
    },
    computed: {},
    methods: {
        onclick_previous() {
            // if the current question is the first one, navigate to index page
            if (current_question.id == complete_question_list[0].id) {
                uni.redirectTo({url: "/pages/index/index"});
                return;
            }
            let previous_question_id = complete_question_list[complete_question_list.findIndex((element) => element.id == current_question.id) - 1].id;
            uni.redirectTo({url: "/pages/quiz/quiz?id=" + previous_question_id});
        },
        onclick_next() {
            // if the current question is the last one, navigate to result page
            if (current_question.id == complete_question_list[complete_question_list.length - 1].id) {
                uni.navigateTo({url: "/pages/result/result"});
                return;
            }
            let next_question_id = complete_question_list[complete_question_list.findIndex((element) => element.id == current_question.id) + 1].id;
            uni.redirectTo({url: "/pages/quiz/quiz?id=" + next_question_id});
        },
        sumbit_form(e) {
            // if the question has already been answered, no need to do anything
            if (this.answered) return;
            // get the answer from the form and this.type
            if (this.question_type == 0) {
                // single choice
                if (e.detail.value.ans == undefined) return;
                this.answers = [e.detail.value.ans];
            } else if (this.question_type == 1) {
                // multiple choice
                if (e.detail.value.ans == undefined || e.detail.value.ans.length == 0) return;
                this.answers = e.detail.value.ans;
            }

            // save the results to localStrorage
            try {
                const answers_list = JSON.parse(uni.getStorageSync("answers"));
                let index = answers_list.findIndex((element) => element.id == current_question.id);
                if (index == -1) {
                    // if the question hasn't been answered before, then add it to the list
                    answers_list.push({
                        id: current_question.id,
                        answers: this.answers,
                    });
                } else {
                    // if the question has been answered before, then update the answer
                    answers_list[index].answers = this.answers;
                }
                uni.setStorageSync("answers", JSON.stringify(answers_list));
            } catch (e) {
                console.warn(e);
            }
            // try reloading the page to ignore all refresh problems
            uni.redirectTo({
                url: "/pages/quiz/quiz?id=" + current_question.id,
            });
        },
        color_option(id) {
            if (this.previous_answers == undefined || current_question.answer == undefined) return {color: "#000000"};
            if (current_question.answer.includes(id)) {
                if (this.previous_answers.includes(id)) return {color: "#00ff00"};
                return {color: "#9ed99d"};
            }
            if (this.previous_answers.includes(id)) return {color: "#ff0000"};
            return {color: "#000000"};
        },
    },
};
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
}

.form-block {
    width: 80%;
}

.options-block {
    height: 400rpx;
}

.options-sub-block {
    margin-top: 20rpx;
}

.option {
    font-size: 30rpx;
}

.message {
    font-size: 30rpx;
    margin-top: 20rpx;
}

.buttons {
    display: flex;
    margin: 30rpx;
}

.navigate_button {
    font-size: 30rpx;
    margin: 15rpx;
}

.progress-box {
    height: 50rpx;
    width: 90%;
    margin-top: 10rpx;
}
</style>
