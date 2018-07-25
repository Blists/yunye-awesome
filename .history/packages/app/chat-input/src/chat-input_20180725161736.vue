<template>
    <div class="ui-input">
        <!-- 禁言中或者环信不在线 -->
        <div v-if="gag || logOut" class="ui-input-panel">
            <div class="ui-input-emoji"></div>
            <div class="ui-input-input">
                <input v-if="gag" class="gag" :placeholder="gag" readonly>
                <input v-if="logOut" class="gag" placeholder="请登录" readonly>
            </div>
            <div class="ui-input-add"></div>
        </div>
        <!-- 正常情况 -->
        <template v-else>
            <div class="ui-input-panel">
                <div v-if="hasEmoji" class="ui-input-emoji" @click.stop="clickEmoji"></div>
                <div v-else class="ui-input-no"></div>
                <div v-if="hasPhoto" class="ui-input-photo">
                    <input ref="img" type="file" @change="sendImgMsg">
                </div>
                <div class="ui-input-input">
                    <input ref="input" :class="{gag:gag}" v-model="textMsg" type="text" @focus="scrollBottom" @keyup.enter="sendTextMsg" :placeholder="gag||placeholder" :readonly="gag">
                </div>
                <!-- <template v-if="moreBtn">
                    <div v-if="hasVideo" class="ui-input-video" @click.stop="clickVideo"></div>
                    <div v-if="hasPhoto" class="ui-input-photo">
                        <input ref="img" type="file" @change="sendImgMsg">
                    </div>
                </template>
                <div v-if="hasVideo||hasPhoto" class="ui-input-add" :class="{active:moreBtn}" @click="moreBtn = !moreBtn"></div>
                <div v-else class="ui-input-no"></div> -->
                <div v-if="textMsg" class="ui-input-send">
                    <button @click="sendTextMsg">发送</button>
                </div>
            </div>
            <div class="ui-input-container">
                <div class="ui-input-container-emojis" v-if="showEmojis">
                    <div v-for="(v,k) in expressions.expressions" :key="k" class="ui-input-container-emoji" @click.stop="addEmoji(k)">
                        <img :src="expressions.expressionsPrefix+v" alt="k">
                    </div>
                </div>
                <div class="ui-input-container-video" v-if="showVideoBtn" @click.stop=";">
                    <div class="button" @touchstart.stop="startRecord" @touchend.stop="endRecord" :class="{active:record.state}"></div>
                    <div v-text="record.state?'松手发送，上滑动取消':'按住说话'"></div>
                </div>
            </div>
            <div v-if="record.state" class="ui-input-recording"></div>
        </template>
    </div>
</template>

<script>
import expressions from "./../../../utils/expressions";
import { MSG_TYPE, CMD } from "@src/utils/constants";
export default {
    props: {
        hasEmoji: {
            type: Boolean,
            default: true
        },
        hasVideo: {
            type: Boolean,
            default: true
        },
        hasPhoto: {
            type: Boolean,
            default: true
        },
        placeholder: {
            type: String,
            default: "请输入"
        },
        gag: {
            type: Boolean,
            default: false
        },
        logOut: {
            type: Boolean,
            default: false
        }
    },
    data() {
        return {
            textMsg: "",
            moreBtn: false,
            showEmojis: false,
            showVideoBtn: false,
            record: {
                state: false,
                pageY: 0
            },
            expressions: expressions
        };
    },
    created() {
        document.addEventListener("click", () => {
            this.showEmojis = false;
            this.showVideoBtn = false;
        });
    },
    methods: {
        clickEmoji() {
            this.showVideoBtn = false;
            this.showEmojis = !this.showEmojis;
            this.scrollBottom();
        },
        clickVideo() {
            this.showEmojis = false;
            this.showVideoBtn = !this.showVideoBtn;
            this.scrollBottom();
        },
        scrollBottom() {
            setTimeout(() => {
                this.$emit("scroll-bottom");
            }, 500);
        },
        // 发送文字消息
        sendTextMsg() {
            if (!this.textMsg) return this.$toast("消息不能为空");
            this.$emit("send", { msgType: MSG_TYPE.TXT, msg: this.textMsg });
            this.textMsg = "";
        },
        // 发送图片消息
        sendImgMsg() {
            this.$emit("send", { msgType: MSG_TYPE.IMG, msg: this.$refs.img.files[0] });
        },
        startRecord(e) {
            e.preventDefault();
            this.record.time = new Date().getTime();
            this.record.state = true;
            this.record.pageY = e.touches[0].pageY;
        },
        endRecord(e) {
            this.record.state = false;
            if (new Date().getTime() - this.record.time > 1000) {
                if (this.record.pageY - e.changedTouches[0].pageY < 30) {
                    // TODO 发送录音
                    this.$toast("发送录音");
                } else {
                    this.$toast("取消发送");
                }
            } else {
                this.$toast("录音时间太短");
            }
        },
        addEmoji(k) {
            this.textMsg = this.textMsg + k;
            // let i = this.$refs.input;
            // i.scrollLeft = i.scrollWidth;
        }
    }
};
</script>

<style lang="less">
@import "~@src/style/variables.less";
@input-height: 50px;
.ui-input {
    .ui-input-panel {
        display: flex;
        // border-top: 1px solid #c0c6cd;
        height: @input-height;
        background: white;
        .ui-input-no {
            width: 10px;
        }
        .ui-input-emoji {
            width: 40px;
            background: url(../assets/img/键盘表情-默认.png) no-repeat center;
            background-size: auto 23px;
            &:active {
                background-image: url(../assets/img/键盘表情-点击.png);
            }
        }
        .ui-input-input {
            flex: 1;
            padding: 6px 0;
            input {
                width: 100%;
                height: 37px;
                padding-left: 8px;
                border-radius: 4px;
                background: #fff;
                // border: 1px solid #c0c6cd;
                font-size: 15px;
                color: #1a1a1a;
                &.gag {
                    text-align: center;
                }
                &::-webkit-input-placeholder {
                    color: #afafaf;
                }
            }
        }
        .ui-input-video {
            width: 40px;
            background: url(../assets/img/键盘语音-默认.png) no-repeat center;
            background-size: auto 23px;
            &:active {
                background-image: url(../assets/img/键盘语音-选中.png);
            }
        }
        .ui-input-photo {
            width: 40px;
            height: @input-height;
            background: url(../assets/img/键盘图片-默认.png) no-repeat center;
            background-size: auto 22px;
            overflow: hidden;
            &:active {
                background-image: url(../assets/img/键盘图片-选中.png);
            }
            input {
                display: inline-block;
                width: 100%;
                height: @input-height;
                opacity: 0;
            }
        }
        .ui-input-add {
            width: 40px;
            background: url(../assets/img/键盘添加-默认.png) no-repeat center;
            background-size: auto 28px;
            &:active {
                background-image: url(../assets/img/键盘添加-点击.png);
            }
            &.active {
                background-image: url(../assets/img/键盘减少-点击.png);
                &:active {
                    background-image: url(../assets/img/键盘减少-点击.png);
                }
            }
        }
        .ui-input-send {
            padding: 10px 10px;
            text-align: center;
            button {
                height: 30px;
                background: @info;
                border-radius: 5px;
                font-size: 10px;
                color: white;
                &:active {
                    opacity: 0.8;
                }
            }
        }
    }
    .ui-input-container {
        > div {
            border-top: 1px solid #c0c6cd;
            height: 250px;
        }
        .ui-input-container-emojis {
            display: flex;
            flex-flow: row wrap;
            align-content: flex-start;
            padding: 5px 0;
            overflow: auto;
            .ui-input-container-emoji {
                flex: 0 0 10%;
                text-align: center;
                img {
                    padding: 5px 0;
                    max-width: 80%;
                }
            }
        }
        .ui-input-container-video {
            padding-top: 50px;
            text-align: center;
            .button {
                display: inline-block;
                width: 105px;
                height: 105px;
                background: url(../assets/img/按住说话-默认.png) no-repeat center;
                background-size: 105px 105px;
                -webkit-touch-callout: none;
                -webkit-user-select: none;
                &.active {
                    background-image: url(../assets/img/按住说话-点击.png);
                }
            }
            div {
                padding-top: 20px;
            }
        }
    }
    .ui-input-recording {
        position: fixed;
        z-index: 99;
        top: 50%;
        left: 50%;
        width: 100px;
        height: 100px;
        margin-top: -50px;
        margin-left: -50px;
        border-radius: 5px;
        background: rgba(0, 0, 0, 0.3);
        &:after {
            content: "";
            position: absolute;
            z-index: 100;
            width: 100%;
            height: 100%;
            background: url(../assets/img/record.gif) no-repeat center;
            background-size: 100px 100px;
        }
    }
}
</style>