<template>
    <div :class="s.code">
         // 验证码输入框
        <div :class="s.code_input">
            <input
                v-for="(c, index) in code"
                :key="index"
                ref="input"
                v-model="code[index]"
                :style="{ borderBottomColor: index <= cIndex ? '#0BB27A' : '' }"
                @input="e => onInput(e.target.value, index)"
                @keydown.delete="onKeydown(index)"
                @focus="onFocus"
            />
        </div>
    </div>
</template>

<script>
export default {
    props: {
        nums: {
            type: Number,
            default: 5
        }
    },
    data() {
        return {
            code: ['']
        }
    },
    computed: {
        // 下一个空白输入框
        cIndex() {
            const i = this.code.findIndex(c => c === '')
            return (i + this.nums) % this.nums
        },
        lastCode() {
            return this.code[this.nums - 1]
        }
    },
    watch: {
        cIndex() {
            this.resetFocus()
        },
        lastCode(val) {
            if (val) {
                this.$refs.input[this.nums - 1].blur()
                const code = this.code.join('')
                if (!/^[A-Za-z0-9]+$/.test(code)) {
                    this.$toast('请输入正确的企业识别码')
                    this.reset()
                }
                this.$emit('send', code)
            }
        }
    },
    mounted() {
        const initCode = []
        for (let i = 0; i < this.nums; i++) {
            initCode.push('')
        }
        this.code = initCode
    },
    methods: {
        // 重置光标
        resetFocus() {
            const lastIndex = this.nums - 1
            this.$refs.input[lastIndex].focus()
        },
        onInput(val, index) {
            const value = val.replace(/\s/g, '')
            const lastIndex = this.nums - 1
            if (index === lastIndex) {
                // 最后一格
                this.code[lastIndex] = value[0]
            } else if (value.length > 1) {
                for (let i = index; i < this.nums && i - index < value.length; i++) {
                    this.code[i] = value[i]
                }
            }
        },
        onFocus() {
            this.$refs.input[this.cIndex].focus()
        },
        // 删除
        onKeydown(index) {
            if (index > 0) {
                if (index === this.nums - 1 && this.code[index]) {
                    // 最后一格需要删除当前格数据
                    this.code.splice(index, 1, '')
                } else {
                    this.code.splice(index - 1, 1, '')
                    this.$refs.input[index - 1].focus()
                }
            }
        },
        // 重置验证码框
        reset() {
            this.code = this.code.map(() => '')
            this.resetFocus()
        }
    }
}
</script>

<style lang="scss" module="s">
.code {
    &_input {
        display: flex;
        align-items: center;
        justify-content: space-between;
        input {
            width: 1rem;
            padding: 0.13rem 0.27rem;
            color: #1c2438;
            font-size: 0.44rem;
            border-bottom: 1px solid #bbbec4;
            border-radius: 0;
            outline: none;
        }
    }
}
</style>
