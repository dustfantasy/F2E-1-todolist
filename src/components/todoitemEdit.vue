<template>
  <div id='todoitem'>
    <section>
      <header :class="titleClass">
        <Row>
          <Col span="1" offset="3">
            <Checkbox v-model="Item.complete" size="large" @click.stop.native="setComplete"></Checkbox>
          </Col>
          <Col span="15" id="titleInput">
            <Input v-model="newItem.title" placeholder="Type Something Here..." v-show="Item.edit" :disabled="Item.complete"></Input>
            <span v-show="!Item.edit" :style="{textDecoration: Item.complete ? 'line-through' : ''}">{{Item.title}}</span>
          </Col>
          <Col span="2" offset="1">
            <Icon type="ios-star-outline" style="cursor: pointer" v-show="!Item.star" @click.stop="setStar"></Icon>
            <Icon type="ios-star" style="color: #F5A623;cursor: pointer" v-show="Item.star" @click.stop="setStar"></Icon>
          </Col>
          <Col span="2">
            <Icon type="edit" style="color:blue" v-show="Item.edit"></Icon>
            <Icon type="edit" style="cursor: pointer" v-show="!Item.edit" @click="editing"></Icon>
          </Col>
        </Row>

        <Row v-show="!Item.edit">
          <Col span="2" :offset="index === 0 ? 4:0" v-for="(notice, index) in notice" :key="index">
          
            <i class="fa fa-calendar" v-show="notice==='deadLine'"></i>
            <i class="fa fa-file-o" v-show="notice==='files'"></i>
            <i class="fa fa-commenting-o" v-show="notice==='comment'"></i>
          </Col>
        </Row>

      </header>

      <div class="todobody" v-show="Item.edit">
        <Row>
          <Col span="1" offset="4"><i class="fa fa-calendar"></i></Col>
          <Col span="16">Deadline</Col>
        </Row>
        <Row>
          <Col span="16" offset="5">
            <DatePicker type="datetime" :disabled="Item.complete" v-model="newItem.deadLine"></DatePicker>
            <!-- <TimePicker type="time" :disabled="Item.complete" v-model="date"></TimePicker> -->
          </Col>
        </Row>
        <Row>
          <Col span="1" offset="4"><i class="fa fa-file-o"></i></Col>
          <Col span="16">File</Col>
        </Row>
        <Row>
          <Col span="15" offset="5">

            <Row>
              <Col span="8" v-for="(file, index) in newItem.files" :key="index">{{file.name}}</Col>
              <Col span="8">
                <Button type="ghost" icon="plus" @click="upload" :disabled="Item.complete">
                  <input id="upload" type="file" hidden>
                </Button>
              </Col>
            </Row>

          </Col>
        </Row>
        <Row>
          <Col span="1" offset="4"><i class="fa fa-commenting-o"></i></Col>
          <Col span="16">Comment</Col>
        </Row>
        <Row>
          <Col span="16" offset="5">
            <Input v-model="newItem.comment" type="textarea" :rows="4" placeholder="Enter something..." :disabled="Item.complete"></Input>
          </Col>
        </Row>
      </div>

      <footer class="todofooter" v-show="Item.edit">
        <Row>
          <Col span="12" class="cancel-button">
            <div @click="cancel">
              <Icon type="ios-close-empty"></Icon> Cancel
            </div>
          </Col>
          <Col span="12" class="add-button">
            <div @click="save">
              <Icon type="ios-plus-empty"></Icon> Save
            </div>
          </Col>
        </Row>
      </footer>
    </section>
  </div>
</template>
<script>
  export default {
    name: 'todoitem',
    props: {
      Item: {
        type: Object
      },
      index: {
        type: Number
      }
    },
    data: function() {
      return {
        newItem: {
          title: '',
          deadLine: '',
          files: [],
          comment: ''
        },
        titleClass: {
          todotitle: true,
          'todotitle-star': this.Item.star,
          'todotitle-edit': false
        },
        notice: []
      }
    },
    created() {
      this.updateNotice(this.Item)
      this.init()
    },
    methods: {
      init() {
        this.newItem = { ...this.Item }
        delete this.newItem.complete
        delete this.newItem.edit
        delete this.newItem.star
        delete this.newItem.order
      },
      upload() {
        let upload = document.getElementById('upload')
        upload.click()
        upload.addEventListener('change', (e) => {
          this.newItem.files = upload.files
        })
      },
      setStar() {
        this.Item.star = !this.Item.star
        this.titleClass['todotitle-star'] = this.Item.star
      },
      setComplete() {},
      editing() {
        if(this.Item.edit) return;
        this.Item.edit = true
        this.titleClass['todotitle-edit'] = this.Item.edit
        this.init()
      },
      cancel() {
        this.Item.edit = false
        this.titleClass['todotitle-edit'] = this.Item.edit
        this.$emit('cancel')
        this.init()
      },
      save() {
        if(!this.newItem.title) {
          alert('標題不能為空')
          return
        }
        this.Item.edit = false
        this.titleClass['todotitle-edit'] = this.Item.edit
        this.$emit('save', this.newItem, this.index)
        this.updateNotice(this.newItem)
        this.init()
      },
      updateNotice(item) {
        this.notice = []
        let keys = ['deadLine', 'files', 'comment']
        for (const key of keys) {
          if (item.hasOwnProperty(key)) {
            if((key === 'files' && item[key].length !== 0))
              this.notice.push(key)
            else if (key!== 'files' && item[key] !== '')
              this.notice.push(key)
          }
        }
      }
    }
  }
</script>
<style >
section {
  width: 700px;
  /* height: 70px; */
  margin: 0 auto 5px;
  /* text-align: left; */
  transition: all 0.5s;position:relative;
}

section:hover{
  margin-top: -50px;
  margin-bottom: 65px;
  box-shadow: 0px 0px 10px 4px rgba(110, 110, 110, 0.335);
  z-index: 10;
}

.todotitle {
  padding-top: 20px;
  font-size: 24px;
  height: 100px;
  /* line-height: 50px; */
  margin-bottom: 5px;
  background: #F2F2F2;
  border-radius: 5px 5px 5px 5px;z-index: 999;
}
.todotitle-edit {
  height: 60px;
  line-height: 60px;
  padding-top: 0px;
  border-radius: 5px 5px 0px 0px;
}
.todotitle-star {
  background: #FFF2DC;
}
#titleInput input[type=text] {
  font-size: 24px;
  background-color: rgba(0, 0, 0, 0);
  border: 0px;
  border-radius: 0px;
}
#titleInput input[type=text]:focus {
  outline: none;
}

.todobody {
  font-size: 20px;
  background: #F2F2F2;
  height: auto;
  background: #F2F2F2;
  padding-top: 30px;
  padding-bottom: 30px;
}

/* footer */
.todofooter {
  font-size: 24px;
  height: 50px;
  width: 100%;
  text-align: center;
  line-height: 50px;
  box-shadow: 0 4px 4px 0 #C8C8C8;
  cursor: pointer;
}
.cancel-button {
  font-family: 'Roboto-Regular';
  font-size: 24px;
  color: red;
  background-color: #fff;
  border-radius: 0px 0px 0px 5px;
}
.cancel-button:hover {
  background-color: rgb(255, 233, 233);
}
.cancel-button:active {
   border: 2px solid #C8C8C8;
}
.add-button {
  font-family: 'Roboto-Regular';
  font-size: 24px;
  color: #fff;
  background-color: blue;
  border-radius: 0px 0px 5px 0px;
}
.add-button:hover {
  background-color: rgb(9, 6, 197)
}
.add-button:active {
  border: 2px solid #C8C8C8;
}
</style>

