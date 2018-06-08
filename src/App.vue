<template>
  <div id="app">
    <header class="pageTitle">
      <ul id="menu" @click="menuClick">
        <li class="active">My Tasks</li>
        <li >In Progess</li>
        <li>Completed</li>
      </ul>
    </header>

    <main>
      <div id="addTask">
        <Button @click="addTask" class="addTask" v-show="!add">
          <Icon type="plus"></Icon> Add Task
        </Button>
      </div>
      <todoitemEdit ref="addTask" v-show="add" :Item="newItem" @cancel="cancel" @save="save"></todoitemEdit>


      <todoitemEdit v-for="(item,index) in filterItemList" :key="item.id" :Item="item" :index="index" draggable="true" 
        @save="save" @filterItemList="filterItemList" @putTodoItem="putTodoItem"
        @dragover.native.stop="allowDrop($event)" @drop.native.stop="drop($event, item.id)" @dragstart.native="dropStart($event, item.id)"></todoitemEdit>

    </main>
  </div>
</template>

<script>
import todoitemEdit from './components/todoitemEdit'

export default {
  name: 'App',
  components: {
    todoitemEdit
  },
  data: function(){
    return {
      add: false,
      item: {
        complete: false,
        star: false,
        edit: true,
        title: '',
        deadLine: '',
        files: [],
        comment: ''
      },
      newItem: {},
      itemList: [],
      viewItemList: [],
      nowPage: 'My Tasks',
      order: []
    }
  },
  created () {
    let body = document.getElementsByTagName('body')[0]
    body.addEventListener('drop', this.drop)
    body.addEventListener('dragover', this.allowDrop)
    this.getOrder()
    this.getTodoItems()
  },
  computed: {
    filterItemList() {
      let filterItemList = {
        'In Progess': () => {
          return this.itemList.filter(item => !item.complete)
        },
        'Completed': () => {
          return this.itemList.filter(item => item.complete)
        },
        'My Tasks': () => {
          return [ ...this.itemList ]
        }
      }
      return filterItemList[this.nowPage]()
    }
  },
  methods: {
    getOrder() {
      this.axios.get('http://localhost:3000/order/1').then((res) => {
        console.log(res)
        this.order = res.data.order
        this.sortByOrder()
      }) 
    },
    putOrder() {
      this.axios.put('http://localhost:3000/order/1', {order:this.order}).then((res) => {
        console.log(res)
      })
    },
    getTodoItems() {
      this.axios.get('http://localhost:3000/todoitems/').then((res) => {
        console.log(res)
        this.itemList = res.data
        this.sortByOrder()
      })
    },
    postTodoItem(body) {
      body.edit = false
      this.axios.post('http://localhost:3000/todoitems/', body).then((res) => {
        console.log(res)
        this.itemList.unshift(res.data)
        this.order.unshift(res.data.id)
        this.putOrder()
      })
    },
    putTodoItem(id, body) {
      body = {...body,edit:false}
      this.axios.put(`http://localhost:3000/todoitems/${id}`,  body).then((res) => {
        console.log(res)
      })
    },

    sortByOrder() {
      if(this.itemList.length !== 0 && this.order.length !== 0) {
        this.itemList = this.order.map(o => {
          return this.itemList.find(item => item.id === o)
        })
      }
    },

    drop(e, id) {
      let originId = e.dataTransfer.getData("id");
      let originIndex = getIndexByOrder.call(this, originId)
      
      if(id === undefined){
        if(e.clientY <= 145) {
          arraymove(this.itemList, originIndex, 0)
          arraymove(this.order, originIndex, 0)
          this.putOrder()
        }
      }
      else {
        let targetIndex = getIndexByOrder.call(this,id)
        targetIndex = e.offsetY < 50 ? targetIndex : targetIndex + 1
        arraymove(this.itemList, originIndex, targetIndex)
        arraymove(this.order, originIndex, targetIndex)
        this.putOrder()
      }
      
      function arraymove(arr, fromIndex, toIndex) {
        var element = arr[fromIndex];
        arr.splice(fromIndex, 1);
        arr.splice(toIndex, 0, element);
      }
      function getIndexByOrder(id) {
        let index
        this.itemList.every((item,i) => {
          if(item.id != id) return true
          index = i
          return false
        });
        return index
      }
    },
    allowDrop(e) {
      e.preventDefault()
    },
    dropStart(e, id) {
      e.dataTransfer.setData("id", id);
    },

    menuClick(e) {
      if(e.target.tagName === 'LI') {
        menu.getElementsByClassName('active')[0].classList.remove('active')
        e.target.className = 'active'
        this.nowPage = e.target.innerText
      }
    },
    addTask() {
      this.add = !this.add
      this.newItem = Object.assign({}, this.item)
      this.$refs.addTask.editing()
    },
    cancel() {
      this.add = !this.add
    },
    save(newItem, index) {
      if(index !== undefined) {
        let item = Object.assign(this.itemList[index], newItem)
        this.putTodoItem(this.itemList[index].id, item)
      }
      else {
        this.add = !this.add
        let item = {...this.newItem, ...newItem}
        this.postTodoItem(item)
      }
    }
  }
}
</script>

<style>
html {
  height: 100%;
}
body {
  margin: 0px 0px;
  height: 100%;
  background: #E1E1E1;
}
main {
  margin-top: 10px;
  /* display: flex;
  flex-direction:column; */
};

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.pageTitle {
  background: #4A90E2;
  height: 76px;
}
ul {
  width: 700px;
  height: 6.2%;
  margin: auto;
  padding-left: 0px;
  text-align: center;
}
ul li{
  font-family: 'Roboto-Regular';
  font-size: 24px;
  color: #00408B;
  width: 150px;
  height: 76px;
  line-height: 76px;
  margin: 0 20px;
  display: inline-block;
  transition: border-bottom 1s ease;
  border-bottom: 6px solid rgba(255, 255, 255, 0);
}
ul li:hover {
  color: #fff;
  cursor: pointer;
}
.active {
  color: #fff;
  border-bottom: 6px solid #00408B;
}

#addTask {
  width:700px;
  margin: auto;
  z-index: 0;
  position:relative;
}
.addTask {
  width: 100%;
  background: #FFFFFF;
  border: 2px solid #C8C8C8;
  border-radius: 5px;
  font-family: Roboto-Regular;
  font-size: 24px;
  color: #C8C8C8;
  text-align: left;
  margin-bottom: 15px;
}
</style>
