<template>
  <div class="data-container" style="">
    <div class="treeWraper">

        <!--{{curVm&&curVm._data}}-->
        <div v-if="0">
            noReactiveData
            {{noReactiveData}}
        </div>
        <!--<button @click="createTreeData(curVm,true)">Check</button>-->
        <!--<span @click="viewDataSource" class="view-source-btn">viewDataSource</span>-->
        <!--<el-input  v-if="treeData.length>0" placeholder="Enter keywords to filter" v-model="filterText" size="mini"></el-input>-->

        <!--{{dataTree}}-->
        <el-tree
              v-show="viewType"
              ref="tree"
              :data="treeData"
              :expand-on-click-node="true"
              :highlight-current="true"
              :default-expanded-keys="treeIdArr"
              class="tree"
              node-key="id"
              :filter-node-method="filterNode"
              @node-expand="(node, data)=>{expandTree(node, data,treeIdArr)}"
              @node-collapse="(node, data)=>{collapseTree(node, data,treeIdArr)}">
                <span class="custom-tree-node" slot-scope="{ node, data }">
                        <span v-if="data.root === true" class="tree-root">$Data</span>
                        <span v-else><dataItem :item="data" type="dataTree"></dataItem></span>
                 </span>
        </el-tree>
        <div v-show="!viewType">
            <pre>{{dataSource}}</pre>
        </div>
    </div>

      <!--propstree-->
      <el-tree
              ref="tree"
              :data="propsTreeData"
              :expand-on-click-node="true"
              :highlight-current="true"
              :default-expanded-keys="propsTreeIdArr"
              class="tree"
              node-key="id"
              :filter-node-method="filterNode"
              @node-expand="(node, data)=>{expandTree(node, data,propsTreeIdArr)}"
              @node-collapse="(node, data)=>{collapseTree(node, data,propsTreeIdArr)}">>
                <span class="custom-tree-node" slot-scope="{ node, data }">
                       <span v-if="data.root === true" class="tree-root">$Props</span>
                       <span v-else><dataItem :item="data" type="propsTree"></dataItem></span>
                 </span>
      </el-tree>

  </div>
</template>
<script>
// import _ from 'lodash'
import { isPlainObject } from './../util/util'
import dataItem from './dataItem.vue'
const checkTimer = null
export default {
  name: 'Data',
  components: {
    dataItem
  },
  props: {
    curVm: {
      type: null,
      default () {
        return false
      }
    }
  },
  data () {
    return {
      // datatree
      treeData: [],
      filterText: '',
      treeIdArr: [],     // 展开id
      propsTreeData: [],
      propsTreeIdArr: [],     // 展开id
      viewType: true,    // 视图类型
      dataSource: null,  // data源格式
      noReactiveData: []   // 是否存在非响应式字段
    }
  },
  watch: {
    'curVm': {
      handler (val) {
        if (val) {
          // create TreeData
          this.createTreeData(val, false)
          // create PropsTreeData
          this.createPropsTreeData(val)
          // check noReactiveData
            checkTimer && clearInterval(checkTimer)
          /*                  checkTimer = window.setInterval(()=>{
                      this.mdJ21dJ(val._data,true)
                  },10000)*/
        }
      }
    },
    'curVm._data': {
      handler (val) {
        if (val) {
          // create TreeData
          this.createTreeData(this.curVm, false)
          // create PropsTreeData
          this.createPropsTreeData(this.curVm)
        }
      },
      deep: true
    },
    filterText (val) {
      this.$refs.tree.filter(val)
    }
  },
  mounted () {
  },
  methods: {
    expandTree (node, data, idArr) {
      idArr.push(node.id)
    },
    collapseTree (node, data, idArr) {
      const nodeId = node.id
      const shouldCollapseIdArr = []
      this.getChildrenId(node, shouldCollapseIdArr)
      shouldCollapseIdArr.push(nodeId)
      shouldCollapseIdArr.map((v, i) => {
        const index = idArr.indexOf(v)
        if (index !== -1) {
          idArr.splice(index, 1)
        }
      })
      //this.createTreeData(this.curVm, false)
    },
    filterNode (value, data) {
      if (!value) return true
      return data.label.toUpperCase().indexOf(value.toUpperCase()) !== -1
    },
    // 1Djson2tree
    odJ2treeJ (a, idStr, pidStr, chindrenStr) {
      var r = [], hash = {}, id = idStr, pid = pidStr, children = chindrenStr, i = 0, j = 0, len = a.length
      for (; i < len; i++) {
        hash[a[i][id]] = a[i]
      }
      for (; j < len; j++) {
        var aVal = a[j], hashVP = hash[aVal[pid]]
        if (hashVP) {
          !hashVP[children] && (hashVP[children] = [])
          hashVP[children].push(aVal)
        } else {
          r.push(aVal)
        }
      }
      return r
    },
    // MDjson21Djson
    mdJ21dJ (data, check) {
      const arr = []
      let id = 0
      function parseObj (data, pid) {
        for (const key in data) {
          if (!data.hasOwnProperty(key)) { continue }
          const field = data[key]
          const property = Object.getOwnPropertyDescriptor(data, key)
          if (!Array.isArray(data) && (!property.get || (property.get.name !== 'reactiveGetter')) && check) {
            this.noReactiveData.push(data[key])
          }
          const fieldWithproperty = {
            id: ++id,   // id
            pid: pid,   // fid
            key: key,
            value: field
          }
          Object.defineProperty(fieldWithproperty, 'value', property)
          if (isPlainObject(field) || Array.isArray(field)) {
            const fieldType = isPlainObject(field) ? 'Object' : 'Array'
            fieldWithproperty.label = fieldWithproperty.type = fieldType
            parseObj.call(this, field, id)
          } else {
            fieldWithproperty.label = fieldWithproperty.type = typeof field
          }
          arr.push(fieldWithproperty)
        }
      }
      parseObj.call(this, data, 0)
      if (this.noReactiveData.length > 0) {
        clearInterval(checkTimer)
      }
      return arr
    },
    viewDataSource () {
      //        this.viewType = !this.viewType
      if (this.curVm) {
        this.$emit('showData', JSON.stringify(this.curVm._data, null, 5), true)
      }
    },
    createTreeData (vm, isCheck) {
      if (!isCheck) {
        // 非数据检测
        this.treeData = [{
          id: 0,
          label: 'data',
          value: 'data',
          root: true,
          children: this.odJ2treeJ(this.mdJ21dJ(vm._data, false), 'id', 'pid', 'children')
        }]
      }
      this.$emit('showData', JSON.stringify(this.curVm._data, null, 5), false)
    },
    createPropsTreeData (vm) {
      this.propsTreeData = [{
        id: 0,
        label: 'prpos',
        value: 'prpos',
        root: true,
        children: this.odJ2treeJ(this.mdJ21dJ(vm.$props, false), 'id', 'pid', 'children')
      }]
    },
    getChildrenId (node, shouldCollapseIdArr) {
      node.children.map((node_v, i) => {
        shouldCollapseIdArr.push(node_v.id)
        if (node_v.children && node_v.children.length > 0) {
          this.getChildrenId(node_v, shouldCollapseIdArr)
        }
      })
    }

  }

}
</script>

<style scoped>
  .basic-container{

  }
  .view-source-btn{
      color: #519eff;
      font-weight: 500;
      cursor: pointer;
  }
  .view-source-btn:hover{
      color:#173dff;
      font-weight: 500;
      cursor: pointer;
  }
  .tree-root{
      color: #67c23a;
      font-size: 16px;
  }


</style>
<style>
</style>

