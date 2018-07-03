<template>
  <div class="hello">
  1txD0R34T5sdkMh9KJK0LpUqjjyIGP6_R
    <div style="margin-top: 1vh;">            
        <el-input placeholder="Please input folder id" v-model="input0" class="input-with-select">
            <template slot="prepend">
              <img src="../assets/drive-icon-gray.png" style="height: 50%; margin:auto;">
            </template>
            <el-button slot="append" icon="el-icon-search" v-on:click="startSearch"></el-button>
        </el-input>
    </div>
    <div style="margin-top: 1vh;">            
        <el-input placeholder="Please input spreadsheet id" v-model="input1" class="input-with-select">
            <template slot="prepend">
              <img src="../assets/spreadsheet-icon-gray.png" style="height: 50%; margin:auto;">
            </template>
            <el-button slot="append" icon="el-icon-search" v-on:click="querySheet"></el-button>
        </el-input>
    </div>
    <div class="custom-tree-container" style="margin-top:2vh;">
      <div class="block">
        <el-tree
          :data="data"
          v-loading="loading"
          node-key="id"
          default-expand-all
          @node-click="handleNodeClick"
          :expand-on-click-node="false"
        >

        <span class="custom-tree-node" slot-scope="{ node, data }">
          
          <template v-if="node.label.isFolder === true">
            <span><i class="el-icon-goods"></i></span>
            <span>
              {{ node.label["Folder Name"] }}
            </span>
            <span style='margin-right: 5%;'></span>
            <span><i class="el-icon-goods" style="color:gray;"></i></span>
            <span>{{ node.label["foldercnt"] }}</span>
            <span><i class="el-icon-document" style="color:gray;"></i></span>
            <span>{{ node.label["itemcnt"] }}</span>
            <span>
              <el-tooltip v-bind:content="node.label.URL" placement="top">
                <i class="el-icon-view" v-on:click="handleViewClick(node)"></i>
              </el-tooltip>
            </span>
            <span v-if="node.label.isEpmty === true">
              <i class="el-icon-warning" style="color:red;"> Empty</i>
            </span>
          </template>
          <template v-else>
            <span><i class="el-icon-document"></i></span>
            <span>{{ node.label["Item Name"] }}</span>
            <span style='margin-right: 5%;'></span>
            <span>
              <el-tooltip v-bind:content="node.label.URL" placement="top">
                <i class="el-icon-view" v-on:click="handleViewClick(node)"></i>
              </el-tooltip>
            </span>
          </template>
        </span>
        </el-tree>
      </div>
    </div>

  </div>
</template>

<script>
import axios from 'axios'

function makeTreeStructure(tdata) {
  let self = {}
  let holder = []
  var foldercnt = 0
  for(var item of tdata['child_folder']){
    let childobj = makeTreeStructure(item['child'])
    item.isFolder = true
    item.foldercnt = childobj['foldercnt']
    item.itemcnt = childobj['itemcnt']
    
    if(item.foldercnt + item.itemcnt == 0){
      item.isEpmty = true
    }else{
      item.isEpmty = false
    }
    holder.push(
      {
        label: item
        , children: childobj.holder
      })
    foldercnt++
  }
  var itemcnt = 0
  for(var item of tdata.items) {
    item.isFolder = false
    holder.push({label: item})
    itemcnt++
  }

  self['holder'] = holder
  self['itemcnt'] = itemcnt
  self['foldercnt'] = foldercnt

  return self;
}

function fetchTreeData(url) {
  this.$jsonp(url).then(json => {
          // Success.
          console.log(json)
          return makeTreeStructure(json).holder
        }).catch(err => {
          // Failed.
          console.log(err)
        });
}

export default {
  name: 'HelloWorld',
  data () {
    return {
      data: [{label:"N/A"}],
      loading : false,
      input0: '',
      input1: '',
    }
  },
  methods: {
      handleNodeClick(data) {

      },
      handleViewClick(data) {
        console.log(data)
        var url = ""
        if(data.label.isFolder){
          url = data.label['URL'];
        }else{
          url = data.label['URL'];
        }

        window.open(url);
      },
      startSearch(event) {
        var surl = 'https://script.google.com/a/mis.doshisha.ac.jp/macros/s/AKfycbyoA-cfZH5dQRltGxIL1SNdiFg9DTRi57Zv81-K1qgP/dev?id=' + this.input0;
    
        this.loading = true;
        this.$jsonp(surl).then(json => {
              // Success.
              console.log(json)
              this.data = makeTreeStructure(json).holder
              this.loading = false;
            }).catch(err => {
              // Failed.
              console.log(err)
              this.loading = false;
            })
      },
      querySheet(event) {
        var baseurl = 'https://script.google.com/a/mis.doshisha.ac.jp/macros/s/AKfycbzwieN06w3RSngiNLxhQ6CCKepyKBYOpe1cwXhfhty4zk7wPJI/exec?id=' + this.input1;
        this.loading = true;
        this.$jsonp(baseurl).then(json => {
              // Success.
              console.log(json)
              //this.data = makeTreeStructure(json).holder
              this.loading = false;
            }).catch(err => {
              // Failed.
              console.log(err)
              this.loading = false;
            })
      }
    },
  created () {

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
