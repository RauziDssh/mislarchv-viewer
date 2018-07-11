<template>
  <div class="hello">
    <div style="margin-top: 1vh;">            
        <el-input placeholder="Please input folder URL" v-model="input0" class="input-with-select">
            <template slot="prepend">
              <img src="../assets/drive-icon-gray.png" style="height: 50%; margin:auto;">
            </template>
            <el-button slot="append" icon="el-icon-search" v-on:click="startSearch"></el-button>
        </el-input>
    </div>
    <div style="margin-top: 1vh;">            
        <el-input placeholder="Please input spreadsheet URL" v-model="input1" class="input-with-select">
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
          empty-text="N/A"
          default-expand-all
          :expand-on-click-node="false"
        >

        <span class="custom-tree-node" slot-scope="{ node, data }">
          
          <template v-if="node.label.isFolder === true">
            <span><i class="el-icon-goods"></i></span>
            <span>
              {{ node.label["FolderName"] }}
            </span>
            <span style='margin-right: 5%;'></span>
            <span><i class="el-icon-goods" style="color:gray;"></i></span>
            <span>{{ node.label["foldercnt"] }}</span>
            <span><i class="el-icon-document" style="color:gray;"></i></span>
            <span>{{ node.label["itemcnt"] }}</span>
            <span>
              <el-popover
                placement="right"
                trigger="hover"
                width="auto">
                <span><i class="el-icon-edit-outline" v-on:click="handleCopyClick(node)" style="cursor : pointer;"></i></span>
                <span>{{ node.label["URL"] }}</span>
                <i class="el-icon-view" slot="reference" v-on:click="handleViewClick(node)"></i>
              </el-popover>
            </span>
            <span v-if="node.label.isEpmty === true">
              <i class="el-icon-warning" style="color:red;"> Empty</i>
            </span>
            <span v-if="node.label.cells.length > 0">
              <template v-for='cell in node.label.cells'>
                <span :key="cell">
                   <el-tag size="small" style='margin-right: 1%;'>{{cell.name}}</el-tag>
                </span>
              </template>
            </span>
          </template>
          <template v-else>
            <span><i class="el-icon-document"></i></span>
            <span>{{ node.label["ItemName"] }}</span>
            <span style='margin-right: 5%;'></span>
            <span>
              <el-popover
                placement="right"
                trigger="hover"
                width="auto">
                <span><i class="el-icon-edit-outline" v-on:click="handleCopyClick(node)" style="cursor : pointer;"></i></span>
                <span>{{ node.label["URL"] }}</span>
                <i class="el-icon-view" slot="reference" v-on:click="handleViewClick(node)"></i>
              </el-popover>
            </span>
            <span v-if="node.label.cells.length > 0">
                          <template v-for='cell in node.label.cells'>
                            <span :key="cell">
                              <el-tag size="small" style='margin-right: 1%;'>{{cell.name}}</el-tag>
                            </span>
                          </template>
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

var re1 = /(?=https:\/\/drive.google.com\/open\?id=).*$/
var re2 = /(?=https:\/\/drive\.google\.com\/drive\/folders\/).*$/
var re3 = /(?=https:\/\/drive\.google\.com\/a\/mis\.doshisha\.ac\.jp\/file\/d\/).*(?=\/view\?)/
var re4 = /(?=https:\/\/drive\.google\.com\/drive\/u\/0\/folders\/).*$/
var re5 = /(?=https:\/\/docs\.google\.com\/spreadsheets\/d\/).*(?=\/edit)/
var re6 = /(?=https:\/\/drive\.google\.com\/open\?id=).*$/
var re7 = /(?=https:\/\/docs\.google\.com\/document\/d\/).*(?=\/edit)/
var re8 = /(?=https:\/\/drive\.google\.com\/drive\/u\/1\/folders\/).*$/

function findSignature(url) {
  var sig = -1
  if(url.match(re1)){sig = url.match(re1)[0].replace(/https:\/\/drive.google.com\/open\?id=/,"")}
  if(url.match(re2)){sig = url.match(re2)[0].replace(/https:\/\/drive\.google\.com\/drive\/folders\//,"")}
  if(url.match(re3)){sig = url.match(re3)[0].replace(/https:\/\/drive\.google\.com\/a\/mis\.doshisha\.ac\.jp\/file\/d\//,"")}
  if(url.match(re4)){sig = url.match(re4)[0].replace(/https:\/\/drive\.google\.com\/drive\/u\/0\/folders\//,"")}
  if(url.match(re5)){sig = url.match(re5)[0].replace(/https:\/\/docs\.google\.com\/spreadsheets\/d\//,"")}
  if(url.match(re6)){sig = url.match(re6)[0].replace(/https:\/\/drive\.google\.com\/open\?id=/,"")}
  if(url.match(re7)){sig = url.match(re7)[0].replace(/https:\/\/docs\.google\.com\/document\/d\//,"")}
  if(url.match(re8)){sig = url.match(re8)[0].replace(/https:\/\/drive\.google\.com\/drive\/u\/1\/folders\//,"")}
  if(sig == -1) {console.log('not matched:' + url)}
  return sig
  // var match1 = url.match(/(?<=https:\/\/drive.google.com\/open\?id=).*$/);
  // var match2 = url.match(/(?<=https:\/\/drive\.google\.com\/drive\/folders\/).*$/);
  // var match3 = url.match(/(?<=https:\/\/drive\.google\.com\/a\/mis\.doshisha\.ac\.jp\/file\/d\/).*(?=\/view\?)/);
  // var match4 = url.match(/(?<=https:\/\/drive\.google\.com\/drive\/u\/0\/folders\/).*$/);
  // var match5 = url.match(/(?<=https:\/\/docs\.google\.com\/spreadsheets\/d\/).*(?=\/edit)/);
  // var match6 = url.match(/(?<=https:\/\/drive\.google\.com\/open\?id=).*$/);  

  // if(match1){return match1[0]}
  // if(match2){return match2[0]}
  // if(match3){return match3[0]}
  // if(match4){return match4[0]}
  // if(match5){return match5[0]}
  // if(match6){return match6[0]}
  // return -1
}

function findCellBySignature(_signature, sheet){
  var cells = sheet.filter((cell) => {return cell.signature == _signature});
  if(cells){return cells}
  else {return null}
}

var cells_associated = []
var cells_not_associated = []

function associateCell(__data,__sheet, self){
  if(!__data) return;
  if(!__sheet) return;

  var cells_associated = []

  var func = function associateCellRecursive(_data,sheet){
    if(!_data) return;
    if(! sheet) return;

    for(var item of _data) {
      var cells = findCellBySignature(item.label.signature, sheet);
      item.label.cells = cells;
      cells.forEach((c) => {cells_associated.push(c.signature)})
      associateCellRecursive(item.children, sheet)
    }
  }
  
  func(__data,__sheet)
  self.cells_not_associated = __sheet.filter((c) => {return cells_associated.indexOf(c.signature) == -1})

  if(self)
  {
    if(self.cells_not_associated.length > 0)
    {
      var names = '<div style="height: 50vh; overflow-y: scroll;">'
      self.cells_not_associated.forEach(v => {names = names + '<a href=' + v.url + '>・' + v.name + '</a><br>'})
      names = names + "</div>"
      const h = self.$createElement;
      self.$notify({
        title: self.cells_not_associated.length + ' unlinked files were detected.',
        duration: 0,
        dangerouslyUseHTMLString: true,
        type: 'warning',
        datas: self.cells_not_associated,
        useListings: true,
        message: names
        });
    }
  }

  return {associated: cells_associated, not_associated: cells_not_associated}
}

function makeTreeStructure(tdata) {
  let self = {}
  let holder = []
  var foldercnt = 0
  if (tdata['child_folder'])
  {
    for(var item of tdata['child_folder']){
      let childobj = makeTreeStructure(item['child'])
      item.isFolder = true
      item.cells = []
      item.signature = findSignature(item.URL)
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
  }
  var itemcnt = 0
  for(var item of tdata.items) {
    item.isFolder = false
    item.cells = []
    item.signature = findSignature(item.URL)
    holder.push({label: item})
    itemcnt++
  }

  self['holder'] = holder
  self['itemcnt'] = itemcnt
  self['foldercnt'] = foldercnt
  self.isFolder = true
  self.FolderName = tdata.Foldername

  return self;
}

export default {
  name: 'HelloWorld',
  data () {
    return {
      data: [],
      sheet: [],
      loading : false,
      input0: '',
      input1: '',
      cells_not_associated:[],
    }
  },
  methods: {
      handleNodeClick(data) {

      },
      handleViewClick(data) {
        var url = ""
        if(data.label.isFolder){
          url = data.label['URL'];
        }else{
          url = data.label['URL'];
        }

        window.open(url);
      },
      startSearch(event) {
        var urls = this.input0.split("\,")
        this.loading = true;

        // var base = 'https://script.google.com/a/mis.doshisha.ac.jp/macros/s/AKfycbzAjci2w1he3hHc4RiyG6rzIDqvAGgSGvDUbZpOO71AUqQbK_s0/exec?id=' + findSignature(this.input0)
        // console.log(base)

        // this.$jsonp('https://script.google.com/a/mis.doshisha.ac.jp/macros/s/AKfycbzAjci2w1he3hHc4RiyG6rzIDqvAGgSGvDUbZpOO71AUqQbK_s0/exec?id=' + findSignature(this.input0))
        // .then((v) => {console.log(v)})

        // this.$jsonp('https://script.google.com/a/mis.doshisha.ac.jp/macros/s/AKfycbzAjci2w1he3hHc4RiyG6rzIDqvAGgSGvDUbZpOO71AUqQbK_s0/exec?id=' + findSignature(this.input0))
        // .then((v) => {console.log(v)})

        Promise.all(
          urls.map((__url) => {return this.$jsonp('https://script.google.com/a/mis.doshisha.ac.jp/macros/s/AKfycbyoA-cfZH5dQRltGxIL1SNdiFg9DTRi57Zv81-K1qgP/dev?id=' + findSignature(__url))})
        ).then(msg => 
        {
          console.log(msg)
          this.loading=false
          var _datas = msg.filter(v => {return v != null}).map(v => {return makeTreeStructure(v).holder})
          _datas.forEach(v => 
          {
            v.forEach(x => {this.data.push(x)})
          })
          associateCell(this.data,this.sheet,this)
        }).catch(err => 
          {
            console.log(err)
            this.loading = false;

              const h = this.$createElement;
              this.$notify.error({
                duration: 0,
                title: 'Error',
                message: err
              });
          }
        )

        // Promise.resolve().then(
        //   () => {
        //     urls.forEach((__url) => {
        //       console.log(__url)
        //       var surl = 'https://script.google.com/a/mis.doshisha.ac.jp/macros/s/AKfycbyoA-cfZH5dQRltGxIL1SNdiFg9DTRi57Zv81-K1qgP/dev?id=' + findSignature(__url)
        //       this.$jsonp(surl).then(json => {
        //         // Success.
        //         var _tdata = makeTreeStructure(json);
        //         console.log(_tdata)
        //         return _tdata.holder
        //       })
        //       .catch(err => {
        //         // Failed.
        //         console.log(err)
        //         return []
        //       })
        //     })
        //   }
        // ).then(
        //   (msg) => {console.log(msg);this.loading = false}
        // )
      },
      querySheet(event) {
        var baseurl = 'https://script.google.com/a/mis.doshisha.ac.jp/macros/s/AKfycbzwieN06w3RSngiNLxhQ6CCKepyKBYOpe1cwXhfhty4zk7wPJI/exec?id=' + findSignature(this.input1)
        this.loading = true;
        this.$jsonp(baseurl).then(json => {
              // Success.
              var result = json.map(item => 
              { 
                var _signature = findSignature(item.url)
                return {type: item.type, name: item.name, url: item.url, signature: _signature};
              });
              this.sheet = result
              associateCell(this.data,this.sheet,this)
              this.loading = false;
            }).catch(err => {
              // Failed.
              console.log(err)
              this.loading = false;

              const h = this.$createElement;
              this.$notify.error({
                title: 'Error',
                message: err
              });
            })
      },
      handleCopyClick(data) {
        const text = data.label['URL'];
        const textarea = document.createElement('textarea');
        textarea.style.position = 'fixed';
        textarea.style.opacity = 0;
        textarea.value = text;

        document.body.appendChild(textarea);

        textarea.select();
        document.execCommand('Copy');

        document.body.removeChild(textarea);

        const h = this.$createElement;
        this.$notify.info({
          title: 'URL has been copied to clipboard.',
        });
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

<style>
  .el-tree-node__content:hover {
    cursor: default !important;
  }

  .el-tree-node__content:hover {
    cursor: default !important;
  }

  .el-icon-view {
    cursor: pointer !important;
  }

  .el-icon-view:hover {
    cursor: pointer !important;
  }
</style>

