<template>
  <div class="container">
    <form style="width: 80%;margin: 0 auto">
      <textarea type="text" class="form-control" v-model="words" />
      <h1>{{oldWords}} 转换成 ✨emoji✨
        <button @click="copy()" id="copyButton">
          复制
        </button>
      </h1>
      <div class="picker" v-for="(item, itemIndex) in cnList" :key="itemIndex">
        <picker v-if="translateKey[item]" class="picker-select" @change="bindPickerChange" :value="translateKey[item]['index']" :range="translateKey[item]['list']" @click="selectKey(item)">
          <view class="picker-item">
            {{translateKey[item]['list'][translateKey[item]['index']]}}
          </view>
        </picker>
        <span v-else>{{item}}</span>
      </div>
    </form>
    <picker @change="bindXianluChange" class="picker-select" :value="index" :range="translateList">
      <view>
        当前线路: {{translateList[index]}} , 转换不正常点击切换线路
      </view>
    </picker>
    <p style="font-size: 14px;margin: 0 10px">
      注意: 输入中/英文句子即可开始转换, 最好用空格分开，系统分词不够准确
    </p>
  </div>
</template>

<script>
  import card from '@/components/card'
  import EmojiTranslate from './emoji-translate.js'
  export default {
    data () {
      return {
        words: '',
        translateKey: {},
        result: {},
        triggerKey: '',
        isSearching: false,
        isUsingTimmer: false,
        Timmer: null,
        index: 0,
        translateList: [
          'baidu',
          'youdao',
          'google'
        ]
      }
    },
    components: {
      card
    },
    computed: {
      oldWords () {
        if (this.result) {
          if (this.result && this.result.cn) {
            return this.result.cn.join(' ')
          } else {
            return ''
          }
        } else {
          return ''
        }
      },
      enList () {
        if (this.result) {
          if (this.result && this.result.cn) {
            return this.result.en
          } else {
            return []
          }
        } else {
          return []
        }
      },
      cnList () {
        if (this.result) {
          if (this.result && this.result.cn) {
            return this.result.cn
          } else {
            return []
          }
        } else {
          return []
        }
      }
    },
    watch: {
      async words () {
        await this.computeWords()
      }
    },
    methods: {
      bindPickerChange (key) {
        this.translateKey[this.triggerKey].index = parseInt(key.mp.detail.value)
      },
      bindXianluChange (e) {
        this.index = e.mp.detail.value
      },
      async computeWords (usingOneTimes) {
        console.log('start')
        if (!this.isUsingTimmer) {
          console.log(2)
          clearTimeout(this.Timmer)
        }
        if (!this.isSearching) {
          this.translateKey = {}
          this.result = await this.getWord()
          this.lastWords = this.words
          const emojiList = this.cnList.map((word, index) => {
            const enWord = this.enList[index]
            const res = EmojiTranslate.getAllEmojiForWord(enWord)
            if (res !== enWord) {
              return res
            } else {
              return word
            }
          })
          this.isSearching = false
          emojiList.map((emojiList, index) => {
            if (Object.prototype.toString.call(emojiList) === '[object String]') {
              return emojiList
            } else {
              const key = this.cnList[index]
              this.translateKey[key] = {
                list: emojiList,
                index: 0
              }
              return key
            }
          })
          console.log(this.translateKey)
          if (usingOneTimes) {
            return
          }
          this.Timmer = setTimeout(async () => {
            this.isUsingTimmer = true
            this.computeWords(1)
          }, 1000)
        }
      },
      selectKey (key) {
        this.triggerKey = key
      },
      async getWord () {
        if (this.words.trim() === '') {
          return
        }
        this.isSearching = true
        return new Promise((resolve, reject) => {
          wx.request({
            url: `http://localhost:4000/words/${this.translateList[this.index]}/${this.words}`,
            methods: 'GET',
            success (res) {
              resolve(res.data)
            },
            fail (res) {
              reject(res)
            }
          })
        })
      },
      copy () {
        const list = this.cnList.map(word => this.translateKey[word] ? this.translateKey[word].list[this.translateKey[word].index] : word)
        wx.setClipboardData({
          data: list.join(''),
          success: function (res) {
            // self.setData({copyTip:true}),
            wx.showModal({
              title: '提示',
              content: '复制成功'
            })
          }
        })
      }
    },
    created () {
      // 调用应用实例的方法获取全局数据
    }
  }
</script>

<style scoped>
  textarea {
    width: 100%;
    font-family: 'Helvetica';
    font-weight: 300;
    font-size: 16px;
    height: 150px;
    line-height: 24px;
  }
  .picker, picker-item{
    display: inline-block;
  }
  button {
    border: none;
    outline: none;
    cursor: pointer;
    text-align: center;
    background: #E91E63;
    color: white;
    font-weight: normal;
    transition: all .4s;
    height: 30px;
    border-radius: 2px;
    text-transform: uppercase;
    vertical-align: middle;
    font-size: 14px;
    margin: 20px;
    width: 180px;
  }
  .userinfo {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .userinfo-avatar {
    width: 128rpx;
    height: 128rpx;
    margin: 20rpx;
    border-radius: 50%;
  }

  .userinfo-nickname {
    color: #aaa;
  }

  .usermotto {
  }

  .form-control {
    display: block;
    margin-bottom: 5px;
    border: 1px solid #ccc;
  }

</style>
