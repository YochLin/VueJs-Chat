<template>
    <div class="container">
        <div class="name">
            <h3>Name : {{userName}}</h3>
            <div class="reset" @click="setName()">Reset Name</div>
        </div>
        <!-- name -->
        <div v-show="userNameSet || userName == ''" class="modal">
            <div class="modal_container">
                <div class="modal_header">
                    <header>輸入名稱</header>
                </div>
                <div class="modal_body">
                    <input type="text" id="js-userName" class="userName" @keydown.enter="saveName()">
                    <button @click="saveName()" class="button">設定</button>
                </div>
            </div>
        </div>
        <!-- js-modal -->
        <div class="chatRoom">
            <div class="chatHeader uk-flex uk-flex-left">
                <img src="https://lorempixel.com/50/50/" class="room-image" draggable="true" alt="頭貼">
                <div class="room-title">聊天室</div>
            </div>
            <!-- chatHeader -->
            <div class="chatBody">
                <div v-for="item in messages" :key="item">
                  <!-- 非使用者介面 -->
                  <div v-if="item.userName != userName">
                    <div class="message-box">
                      <img src="https://lorempixel.com/40/40/" alt="頭貼">
                      <div class="message-content">
                        <div class="message-name">{{item.userName}}</div>
                        <div class="message-text" v-if="item.type == 'text'">
                          <div class="message-message">{{item.message}}</div>
                        </div>
                        <div class="message-time">{{item.timeStamp}}</div>
                        <div class="message-image" v-if="item.type == 'image'"><img :src="item.message"></div>
                      </div>
                    </div>
                  </div>
                  <!-- 使用者介面 -->
                  <div v-if="item.userName == userName">
                    <div class="message-box message-user">
                      <div class="message-content">
                        <div class="message-time">{{item.timeStamp}}</div>
                        <div class="message-text" v-if="item.type == 'text'">
                          <div class="message-message">{{item.message}}</div>
                        </div>
                        <div class="message-image" v-if="item.type == 'image'"><img :src="item.message"></div>
                      </div>
                    </div>
                  </div>
                </div>
            </div>
            <div class="loadDataArea">
                <div class="loadData">
                    <input type="file" @change="sendImage($event)" multiple accept="image/*">
                    <img src="../../img/upload.png" alt="">
                </div>
            </div>
            <div class="ChatArea">
                <textarea class="ChatArea-message" id="js-message" @keydown.enter="sendMessage($event)"></textarea>
            </div>
        </div>
        <!-- chatRoom -->
    </div>
    <!-- container -->
</template>

<script>
import firebase from 'firebase'
// Initialize Firebase
var config = {
  apiKey: 'AIzaSyD4BeOXxM0npxuZZFLG26-UOKwp8ELHTQo',
  authDomain: 'yoch-6b6be.firebaseapp.com',
  databaseURL: 'https://yoch-6b6be.firebaseio.com',
  projectId: 'yoch-6b6be',
  storageBucket: 'yoch-6b6be.appspot.com',
  messagingSenderId: '698152557825'
}
firebase.initializeApp(config)

const msgRef = firebase.database().ref('/messages/')
const storageRef = firebase.storage().ref('/images/')
export default {
  name: 'ChatRoom',
  data () {
    return {
      userName: '',
      messages: [],
      userNameSet: false,
      upload: false,
      progress: ''
    }
  },
  methods: {
    /* 設定名稱 */
    setName () {
      const vm = this
      vm.userNameSet = true
    //   document.querySelector('#js-modal').style.display = 'block'
    },
    /* 儲存修改的名稱 */
    saveName () {
      const vm = this
      const userName = document.querySelector('#js-userName').value
      if (userName.trim() === '') { return }
      vm.userName = userName
      vm.userNameSet = false
    //   document.querySelector('#js-modal').style.display = 'none'
    },
    getTime () {
      const now = new Date()
      const hours = now.getHours()
      const minutes = now.getMinutes()
      return `${(hours >= 12) ? '下午' : '上午'} ${hours}:${(minutes < 10) ? '0' + minutes : minutes}`
    },
    sendMessage (e) {
      const vm = this
      const userName = document.querySelector('#js-userName')
      let message = document.querySelector('#js-message')
      if (e.shiftKey) {
        return false
      }
      if (message.value.length <= 1 && message.value.trim() === '') {
        e.preventDefault()
        return false
      }
      msgRef.push({
        userName: userName.value,
        type: 'text',
        message: message.value,
        timeStamp: vm.getTime()
      })
      message.value = ''
      e.preventDefault()
    },
    sendImage (e) {
      const vm = this
      const userName = document.querySelector('#js-userName')
      const file = e.target.files[0]
      const fileName = Math.floor(Date.now() / 1000) + `_${file.name}`
      const metadata = {
        contentType: 'image/*'
      }
      let progressBar = document.querySelector('#js-progressBar')
      const uploadTask = storageRef.child('images/' + fileName).put(file, metadata)
      uploadTask.on(firebase.storage.TaskEvent.STATE_CHANGED,
        function (snapshot) {
          let progress = Math.floor((snapshot.bytesTransferred / snapshot.totalBytes) * 100)
          if (progress < 100) {
            vm.upload = true
            vm.progress = `${progress}%`
            progressBar.setAttribute('style', `width:${progress}%`)
          }
        },
        function (error) {
          msgRef.child('bug/').push({
            userName: userName.value,
            type: 'image',
            message: error.code,
            timeStamp: vm.getTime()
          })
        },
        function () {
          var downloadURL = uploadTask.snapshot.downloadURL
          msgRef.push({
            userName: userName.value,
            type: 'image',
            message: downloadURL,
            timeStamp: vm.getTime()
          })
          vm.upload = false
        })
    }
  },
  mounted () {
    const vm = this
    msgRef.on('value', function (snapshot) {
      const val = snapshot.val()
      vm.messages = val
    })
  },
  updated () {
    const chatBody = document.querySelector('.chatBody')
    chatBody.scrollTop = chatBody.scrollHeight
  }
}
</script>

<style scoped>
*{
  font-family: '微軟正黑體';
  margin: auto;
}
/********************************************************顯示名稱**************************************************************/
.name{
    margin: 20px;
    position: relative;
}
/********************************************************更改名稱部分***********************************************************/
.userName{
    margin: 0;
    height: 30px;
    font-size: 16px;
    margin-bottom: 10px;
    border: solid 1px #cbcbcb;
    width: 70%;
    text-align: center;
    display: inline-block;
}
.reset{
    margin-top: 10px;
    padding: 5px 10px;
    border-radius: 10px;
    font-weight: 600;
    color: #333333;
    background-color: #CCCCCC;
    display: inline-block;
    cursor: pointer;
}
.button{
    font-size: 14px;
    color: white;
    background-color: #2B364B;
    padding: 10px 20px;
    display: inline-block;
    cursor: pointer;
}
.modal{
    background-color: rgba(0,0,0,0.4);
    width: 100%;
    height: 100%;
    text-align: center;
    position: fixed;
    z-index: 1;
    left: 0%;
    top: 0%;
    animation-name: animatetop;
    animation-duration: 0.8s;
}
.modal_container{
    position: relative;
    padding: 30px;
    max-width: 300px;
    outline: 0;
}
.modal_header{
    color: white;
    background-color: #2B364B;
    padding: 10px;
    text-align: center;
    border-radius: 5px 5px 0px 0px;
}
.modal_body{
    background-color: white;
    padding: 20px 50px;
    text-align: center;
}
@keyframes animatetop {
    from {top:-300px; opacity:0}
    to {top:20; opacity:0.8}
}
/**********************************************************************聊天室*****************************************************/
.chatRoom{
    border-radius: 5px;
    max-width: 500px;
    box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22);
}
.chatHeader{
    width: 100%;
    height: 85px;
    color: white;
    background-color: #2B364B;
    border-radius: 5px 5px 0px 0px;
    position: relative;
}
.room-image{
    margin: 15px;
    border-radius: 50%;
    cursor: pointer;
}
.room-title{
    font-size: 16px;
    font-weight: bolder;
    margin: 30px 0px 0px 5px;
}
/*顯示聊天內容部分*/
.chatBody{
    padding: 10px 0px;
    background-color: #fff;
    height: 55vh;
    overflow-y: auto;
    overflow-x: hidden;
}
.message-box{
  padding: 10px;
  position: relative;
  text-align: left;
}
.message-box img{
  display: inline-block;
  vertical-align: top;
  border-radius: 50%;
}
.message-content{
  display: inline-block;
}
.message-name{
  text-align: left;
}
.message-text{
  display: inline-block;
  font-size: 12px;
  color: #35393D;
  letter-spacing: 0.6px;
  background-color: #E3E8EB;
  border-radius: 12px;
  line-height: 1.5;
  text-align: left;
  word-break: break-all;
  white-space: pre-line;
}
.message-message{
  padding: 8px 10px 9px 11px;
  max-width: 200px;
  overflow: hidden;
}
.message-time{
  display: inline-block;
  font-size: 12px;
}
/*使用者顯示內容部分*/
.message-user{
  position: relative;
  text-align: right;  
}
/*上傳檔案*/
.loadDataArea{
    border-top: solid 1px #E7E7E7;
    border-bottom: solid 2px #E7E7E7;
    background-color: #F6F6F6;
    height: 35px;
    padding: 0px 5px;
    position: relative;
}
.loadData{
    padding: 5px;
    display: inline-block;
    overflow: hidden;
}
.loadData:hover{
    border: solid 1px #DCDCDC;
}
.loadData input{
    position: absolute;
    top: 0;
    left: 0;
    opacity: 0;
    cursor: pointer;
    /* 讓input file可以支援pointer要加pl100% */
    padding-left: 100%;
}
.loadData img{
    width: 20px;
    height: 20px;
}
/*聊天打字區域*/
.ChatArea{
    padding: 5px;
    height: 8vh;
}
.ChatArea-message{
    width: 100%;
    border: none;
    resize: none;
    outline: none;
}
</style>
