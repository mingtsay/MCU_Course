<template>
  <div id='ActionPanel'>
    <v-tabs id="tabs" grow>
        <v-tab-item href="#tabs-1" key="1" ripple slot="activators">已選課程</v-tab-item>
        <v-tab-item href="#tabs-2" key="2" ripple slot="activators">加入課程</v-tab-item>
        <v-tab-item href="#tabs-3" key="3" ripple slot="activators">結果</v-tab-item>
        <v-tab-content  id="tabs-1" slot="content" key="1">
          <v-card class="ma-0">
            <v-card-text>
              <p class="ma-0" style="display:inline;"> 目前總共學分數：{{ allCredit }}</p>
              <div style="display:inline-block;width:50px;height:15px;"></div>
              <input type="checkbox" id="note" v-model="note">
              <label for="checkbox"> 顯示備註</label>
              <course-list :headers="headers">
                <list-item v-for="(course,index) in choosedList" :course="course" v-bind:key="course" :note="note">
                  <v-btn icon class="teal--text" @click.native="del(index)" ripple>
                    <v-icon>delete</v-icon>
                  </v-btn>
                </list-item>
              </course-list>
            </v-card-text>
          </v-card>
        </v-tab-content>
        <v-tab-content id="tabs-2" slot="content" key="2">
          <v-card class="ma-0">
            <v-card-text>
              <add-panel @openComment="openComment"></add-panel>
            </v-card-text>
          </v-card>
        </v-tab-content>
        <v-tab-content id="tabs-3" slot="content" key="3">
          <v-card class="ma-0">
            <v-card-text>
              <h5 class="text-xs-center ma-0">結果</h5>
              <p class="ma-0">會分析退選哪些課，加選哪些課的結果</p>
              <p class="ma-0" style="display:inline;"> 目前總共學分數：{{ allCredit }}</p>
              <div style="display:inline-block;width:50px;height:15px;"></div>
              <input type="checkbox" id="note" v-model="note">
              <label for="checkbox">顯示備註</label>
              <course-list :headers="headers">
                <list-item v-for="(course,index) in diffList.del" :course="course" v-bind:key="course" class="through" :note="note">
                  <p class="ma-0">退選</p>
                </list-item>
                <list-item v-for="(course,index) in diffList.add" :course="course" v-bind:key="course" class="addRed" :note="note">
                  <p class="ma-0">加選</p>
                </list-item>
              </course-list>
              <br>
              <v-btn block v-ripple="{ class: 'white--text' }" success @click.native="share" :loading="disabled">分享</v-btn>
              <br>
              <v-container fluid v-if="token!=''">
                <v-row>
                  <v-col xs2>
                    <label for="text">分享網址:</label>
                  </v-col>
                  <v-col xs8>
                    <input type="text" id="share" :value="urlToken" style="width: 100%;" onMouseOver="this.focus()" onFocus="this.select()"  readonly>
                  </v-col>
                  <v-col xs2>
                    <iframe :src="facebook" width="52" height="20" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true"></iframe>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>
          </v-card>
        </v-tab-content >
    </v-tabs>
  </div>
</template>
<script>
  import * as api from '../api/axios.js'
  export default {
    name: 'ActionPanel',
    data() {
      return {
        choosedList: this.$store.state.choosedList,
        note: false,
        disabled: false,
        token: "",
      }
    },
    computed: {
      headers: function () {
        if(this.note){
          return ["制別", "班級代號<br>科目代號", "科目<br>名稱", "選課/<br>開班<br>人數", "任課<br>教師", "時間", "地點", "學分", "備註", "加入"]
        }else{
          return ["制別", "班級代號<br>科目代號", "科目<br>名稱",  "選課/<br>開班<br>人數", "任課<br>教師", "時間", "地點", "學分", "加入"]
        }
      },
      diffList: function () {
        var self = this;
        var loginList = self.$store.state.loginCourse.filter((element) => { return (self.choosedList.indexOf(element) == -1) });
        var choose = self.choosedList.filter((element) => { return (self.$store.state.loginCourse.indexOf(element) == -1) })
        var result = {
          del: loginList,
          add: choose
        }
        return result;
      },
      urlToken: function () {
        return 'https://course.csie.mcu.edu.tw/share/' + this.token;
      },
      facebook: function () {
        return 'https://www.facebook.com/plugins/share_button.php?href='+this.urlToken+'&layout=button&mobile_iframe=true&width=52&height=20&appId'
      },
      allCredit: function () {
        return this.choosedList.reduce((acc,val) => { return acc+val.credit },0)
      }
    },
    methods: {

      del(index) {
        //console.log(index)
        this.$store.commit("removeChoosed", index)
      },
      share() {
        this.disabled = true;
        api.postShare(this.choosedList).then((response) => {
          this.token = response.data.token;
          this.disabled = false;
        }).catch((error) => {
          this.disabled = false;
          this.$vuetify.toast.create(...["請求失敗，請檢查網路狀態。", "bottom"])
        })
      },
      openComment(url){
        this.$emit('openComment',url)
      },
    }
  }

</script>
<style scoped>
  #ActionPanel {
    min-height: 800px;
  }

  .through {
    text-decoration: line-through;
  }
  
  .addRed {
    color: red;
  }
</style>