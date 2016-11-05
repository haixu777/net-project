<template>
  <div class="global-actions clearfix" >


      <div class="btn-group btn-group-justified file-btn-group">
        <a href="#" class="btn btn-default">批量导入</a>
        <a href="javascript:void(0);" class="btn btn-default" @click="exportClick">批量导出</a>
        <a href="#" class="btn btn-default">批量删除</a>
      </div>


  </div>
  <div class="form-actions ">
      <button class="action-button" @click="chooseAll">全选</button>
      <button class="action-button" @click="chooseReverse">反选</button>
  </div>
  <div class="title-panels">
      <div class="panel panel-success">
        <div class="panel-heading">
          <input type="checkbox" value="high danger" v-model="checkedWords"/>
          <h3 class="panel-title">最近一段时间的高危敏感词</h3>
        </div>
        <div class="panel-body">
          <div class="filter-radios">
              <div class="radio-item" v-for="item in duraionList">
                <input type="radio" :value="item.value" v-model="duration" :checked="$index===0"/>
                <label >{{item.zh_value}}</label>
              </div>
          </div>
          <div class="keywords">
            <ul>
            <li v-for="keyword in highDanger" class="tag">
                <span class="label">{{keyword}}</span>
            </li>
            </ul>
            <button class="more">查看详情</button>

          </div>
        </div>
      </div>

      <div class="panel panel-success" v-for="tempCard in cardList">
        <div class="panel-heading">
          <input type="checkbox" :value="tempCard.category" v-model="checkedWords"/>
          <h3 class="panel-title">{{tempCard.title}}</h3>
        </div>
        <card :category="tempCard.category" :topic="topic" :title="tempCard.title"></card>
      </div>


      <div class="panel panel-success">
        <div class="panel-heading">
          <input type="checkbox" value="issued words" v-model="checkedWords" />
          <h3 class="panel-title">下发词</h3>
        </div>
        <div class="panel-body">
            <div>
              "审核"
              <ul>
                  <li v-for="keyword in childData['check words']" class="tag">
                      <span class="label label-default" >{{keyword}}</span>
                  </li>
              </ul>
            </div>
            <div>
              "低危"
              <ul>
                  <li v-for="keyword in childData['low danger']" class="tag">
                      <span class="label label-default" >{{keyword}}</span>
                  </li>
              </ul>
            </div>


        </div>
      </div>




  </div>
  <exportbox title="导出称谓词"
    :word-list= "titleWordList"
    :title-list= "titleWordList.join(' ')"
    :show.sync="showExportBox"
    :package-list.sync = "packageList"
    @export-to-sys = "exportToSys"></exportbox>

</template>

<script>

import Card from "../Card/Card";
import PopModal from "../PopModal/PopModal";
import ExportBox from "../ExportBox/ExportBox";

import {server_path} from "../../../Constants/serverUrl.js";


let sentWordRequest = {"get":null,"update":null,"delete":null,"patch":null};
let checkboxWords = ["high danger","forbidden words","check words","low danger","issued words"];
let cardList = [
  {title:"禁发词",category:"forbidden words"},
  {title:"审核词",category:"check words"},
  {title:"事件通配词",category:"low danger"},
];

export default {

  components:{
    "card":Card,
    "exportbox":ExportBox

   // "modal":PopModal
  },


  data () {

    let duraionList = [
        {name:"halfHour",value:0.5,zh_value:"12小时"},
        {name:"oneDay",value:1,zh_value:"1天"},
        {name:"oneWeek",value:7,zh_value:"一周"},
      ];
    return {

      topic: this.$parent.topic,
      duration:[],
      cardList:cardList,
      duraionList:duraionList,
      childData:{},

      checkedWords:[], //多选框结果

      showExportBox:false,
      titleWordList:[],
      packageList:[]

    }
  },


  events:{
    "child-wordList":function(msg){

        this.childData = Object.assign({},this.childData,msg);
        console.log("----dispath ",this.childData);
    },
    "getTitleWordList":function(msg){
        this.titleWordList = this.titleWordList.concat(msg);
    },
    "clearTitleCache":function(){
        this.titleWordList = [];
    }
  },


  ready () {

    //留作高危敏感词的接口


    },

  methods:{
      exportClick(){
        if(!this.checkedWords.length)return;
        this.$broadcast('getCategoryFromParent',this.checkedWords);
        this.fetchPackageData();
        this.showExportBox = !this.showExportBox;
      },
      exportToSys(data){
        console.log("--------topic",this.topic);
        var postBody = Object.assign({},data,{
          action:"push",
          topic:this.topic
        });
        this.$http.post(server_path+"/transfer",postBody)
             .then(response=>{
                  alert('推送到系统成功');
                  console.log("推送到系统成功");
              },(err)=>{
                  alert('推送到系统失败');
                  console.log("推送到系统失败");
              });

      },
      fetchPackageData(){
          this.$http.get(server_path+"/theme",
              {
              params:{
                topic : this.topic,
                value : "package"
              },
              before(request){
                if(sentWordRequest["get"]){
                  sentWordRequest["get"].abort();
                }
                sentWordRequest["get"] =request;
              }
          })
          .then((response)=>{
            console.log("packages",response.json().packages);
              this.packageList = response.json().packages;
          },(err)=>{
              console.log("请求服务器失败");
          });
      },
    //全选
    chooseAll(){
      if(this.checkedWords.length==checkboxWords.length)
        this.checkedWords = [];
      else this.checkedWords = checkboxWords;
    },
    //反选
    chooseReverse(){
      this.checkedWords = checkboxWords.filter((word)=>{
        return this.checkedWords.indexOf(word)===-1;
      });
    },

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
@import "./Title.scss";
h1 {
  color: #42b983;
}
.tag{
      display: inline-block;
        margin-left: 16px;
}

</style>
