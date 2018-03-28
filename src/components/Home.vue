<template>
  <div class="container">
    <div class="row">
      <div class="col-md-4 board-list" v-for="(list, indexList) in lists" :key="list['.key']" >
        <div class="card" >
          <div class="card-header">
            {{ list.title }} <button @click="deleteList(list['.key'])" class="btn btn-danger btn-sm float-right"><i class="fas fa-trash"></i></button>
          </div>
          <ul class="list-group list-group-flush">
            <li class="list-group-item" v-for="(task, taskIndex) in tasks" :key="task['.key']" v-if="task.list_id == list['.key']">
               {{ task.title }}
               <button @click="deleteTask(task['.key'])" class="btn btn-danger btn-sm float-right"><i class="fas fa-trash"></i></button>
               <button @click="moveTask(taskIndex, indexList + 1)" v-if="indexList < lists.length - 1" class="btn btn-primary btn-sm float-right"><i class="fas fa-arrow-right"></i></button>
               <button v-else="" class="btn btn-primary btn-sm float-right" disabled><i class="fas fa-arrow-right"></i></button>
               <button @click="moveTask(taskIndex, indexList - 1)" v-if="indexList >= 1" class="btn btn-primary btn-sm float-right"><i class="fas fa-arrow-left"></i></button>
               <button  v-else="" class="btn btn-primary btn-sm float-right" disabled><i class="fas fa-arrow-left"></i></button>
            </li>
            <li class="list-group-item">
              <form  >
                <label class="sr-only" for="inlineFormInputName2">Name</label>
                <input type="text" v-model="list.newTask" class="form-control  " id="inlineFormInputName2" placeholder="Add A Task">
                <button type="submit" @click="saveTask(list['.key'], list.newTask, indexList)" class="btn btn-primary " style="margin-top: 12px">Save</button>
              </form>
            </li>
          </ul>
        </div>
      </div>
      <div class="col-md-4 board-list">
        <div class="card" >
          <div class="card-header">
            <form >
              <label class="sr-only" for="inlineFormInputName2">Name</label>
              <input type="text" v-model="newList.title" class="form-control  " id="inlineFormInputName2" placeholder="Add A List">
              <button type="submit" @click="saveList" class="btn btn-primary " style="margin-top: 12px">Save</button>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import * as firebase from 'firebase'
const config = {
  databaseURL: 'https://fir-notification-37d85.firebaseio.com',
  projectId: 'fir-notification-37d85'
}
const firebaseApp = firebase.initializeApp(config)
const db = firebaseApp.database()
const tasksRef = db.ref('tasks')
const listsRef = db.ref('lists')

export default {
  name: 'Home',
  data () {
    return {
      newList: { title: '', newTask: '' },
      newTask: { title: '', list_id: '' },
      formTask: {}
    }
  },
  firebase: {
    tasks: tasksRef,
    lists: listsRef
  },
  methods: {
    saveList () {
      if (this.newList.title === '') {
        this.$swal('New List Field is required to fill')
      } else {
        listsRef.push(this.newList)
        this.newList.title = ''
      }
    },
    saveTask (key, title, index) {
      if (title === '') {
        this.$swal('New Task Field is required to fill')
      } else {
        this.newTask.list_id = key
        this.newTask.title = title
        tasksRef.push(this.newTask)
        this.newTask.title = ''
        this.lists[index].newTask = ''
      }
    },
    deleteList (key) {
      listsRef.child(key).remove()
      this.tasks.forEach(task => {
        if (task.list_id === key) {
          tasksRef.child(task['.key']).remove()
        }
      })
    },
    deleteTask (key) {
      tasksRef.child(key).remove()
    },
    moveTask (taskIndex, moveIndex) {
      const moveList = this.lists[moveIndex]['.key']
      this.tasks[taskIndex].list_id = moveList
      const taskKey = this.tasks[taskIndex]['.key']
      const updatedTask = {...this.tasks[taskIndex]}
      delete updatedTask['.key']
      tasksRef.child(taskKey).set(updatedTask)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.board-list {
  margin-top: 25px;
}
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
