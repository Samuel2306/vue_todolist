<template>
  <div id="app">
    <h2>{{title}}</h2>
    <div class="box clearfix">
      <div class="btn-group pull-right" role="group" aria-label="...">
        <button id="all" type="button" class="btn btn-default btn-xs" @click="changeType">All</button>
        <button id="active" type="button" class="btn btn-default btn-xs" @click="changeType">Active</button>
        <button id="completed" type="button" class="btn btn-default btn-xs" @click="changeType">Completed</button>
      </div>
      <h3>What needs to do</h3>
      
      <ul class="list-group">
        <li v-for="item in filteredItems" class="list-group-item" :class="{finished:item.isFinished}" v-on:dblclick="startEdit(item)">
          <div class="view" v-show="!item.isEditing">
            <input type="checkbox" v-model="item.isFinished">
            <span>{{item.label}}</span>
            <span class="badge" v-on:click="removeItem($index)">×</span>
          </div>
          <input class="edit" type="text"
               v-model="item.label"
               v-show="item.isEditing"
               v-todo-focus="item == editedItem"
               @blur="doneEdit(item)"
               @keyup.enter="doneEdit(item)"
               @keyup.esc="cancelEdit(item)">
        </li>
      </ul>
      <span class="count">{{count}} items need to do</span>
      <button class="btn btn-default pull-right clearItem" @click="clearCompleted" v-show="todos.length">clear completed</button>
    </div>
    <div class="input-group">
      <input type="text" class="form-control" placeholder="What needs to be done?"  v-model="newItem" v-on:keyup.enter="addItem" autofocus>
      <div class="input-group-addon"  v-on:click="addItem">add</div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import Store from './store'
var filters = {
  all: function (todos) {
   return todos;
  },
  active: function (todos) {
   return todos.filter(function (todo) {
      return !todo.isFinished;
   });
  },
  completed: function (todos) {
   return todos.filter(function (todo) {
     return todo.isFinished;
   });
  }
}
export default {
  data () {
    return {
      title:'This is a todolist',
      todos:Store.fetch(),
      showType:'all',
      newItem:'',
      oldItem:'',
      editedItem: null
    }
  },
  watch:{
    todos:{
      handler: function (todos) {
        Store.save(todos);
      },
      deep:true//设置该属性是为了改变数组的某个key属性时，也见检测到变化
    }
  },
  //钩子函数，用来在刷新浏览器时充重置各个数据的编辑状态
  compiled () {
    for(var i = 0;i<this.todos.length;i++){
      var item = this.todos[i];
      item.isEditing = false;
    };
  },
  computed: {
    filteredItems () {
      return filters[this.showType](this.todos);
    },
    labels () {
      var arr = [];
      for(var i = 0;i<this.todos.length;i++){
        var item = this.todos[i];
        arr.push(item.label);
      };
      return arr;
    },
    count () {
      return filters.active(this.todos).length;
    }
  },
  methods: {
    changeType (e) {
      var el = e.target;
      this.showType = el.getAttribute('id');
    },
    addItem () {
      if(this.labels.some(this.checkLabel)){
        alert("该数据已存在");
        return ;
      }
      if(this.newItem && this.newItem.trim()){
        this.todos.push({
          id:this.todos.length,
          label:this.newItem,
          isFinished:false,
          isEditing:false
        });

        this.newItem = '';
      }else{
        alert("不能是空数据");
        this.newItem = '';
        return;
      }
    },
    checkLabel (label) {
        return label == this.newItem;
    },
    removeItem (index) {
      this.todos.splice(index,1);
      for(var i=0; i<this.todos.length;i++){
        this.todos[i].id = i;
      }
    },
    clearCompleted () {
      this.todos = filters.active(this.todos);
    },
    startEdit (item) {
      this.editedItem = item;
      this.oldItem = item.label;
      item.isEditing = true;
    },
    doneEdit (item) {
      item.isEditing = false;
      this.oldItem = '';
      this.editedItem = null;
    },
    cancelEdit (item) {
      item.isEditing = false;
      item.label = this.oldItem;
      this.oldItem = '';
    }
  },
  directives: {
    'todo-focus': function (value) {
      if(value){
        var el = this.el;
        Vue.nextTick(function () {
          el.focus();
        });//在对dom直接进行改动的时候，因为异步更新队列的关系，所以我们要用nextTick不然可能会无效。
      }
       
    }
  }
}
</script>

<style>
.clearfix {
  *zoom: 1;
}
.clearfix:before,
.clearfix:after {
  display: table;
  line-height: 0;
  content: "";
}
.clearfix:after {
  clear: both;
}
#app{
  width:600px;
  margin:0 auto;
}
.box{
  border: 1px solid #ddd;
  padding:12px;
}
.box h3{
  margin:8px 0;
}
.box .btn-group{margin-top: 12px;}
.box .list-group{margin-top:10px;margin-bottom:0px;}
.list-group-item{cursor:pointer;}
.list-group-item.finished{color:#fff;background-color:#337AB7;}
.list-group-item.finished input{color:#333;}
.box .list-group .list-group-item:last-child {
    border-radius: 0px;
}
.view>.badge {
    background-color: #666;
    font-size: 26px;
    margin-top: -5px;
    display:none;
    float:right
}
.view:hover>.badge {
    display:block;
}
.input-group{margin-top: 14px;}
.input-group-addon{cursor:pointer;}

.count {
  display: inline-block;
  margin-top: 8px;
}
.clearItem{
  margin-top: 8px;
}
</style>
