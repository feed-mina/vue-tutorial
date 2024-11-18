<script setup> 
import { ref, computed, watchEffect, reactive } from 'vue';

import { marked } from 'marked';

import { debounce } from 'vue-debounce'
import DemoGrid from './Grid.vue';// 반응형 목록 생성



const groceryList = ref([
  { id: 0, text: '야채' },
  { id: 1, text: '치즈' },
  { id: 2, text: '아무거나' }
])



const text = ref('Edit me')
const checked = ref(true)
const checkedNames = ref(['Jack'])
const picked = ref('One')
const selected = ref('A')
const multiSelected = ref(['A']) 
const message = ref('Hello')
const isRed = ref(true)
const color = ref('green')


const show = ref(true)
const list = ref([1,2,3])

function toggleRed(){
  isRed.value = !isRed.value
}

function toggleColor(){
  color.value = color.value === 'green' ? 'blue':'green' 
}
function reverseMessage(){
  message.value = message.value.split('').reverse().join('')
}

function notify(){
  alert('navigation was prevented.')
}


/* Markdown편집기 */
const input = ref('# hello')
const output = computed(()=> marked(input.value))

const update = debounce((e)=>{
  input.value = e.target.value
}, 100)

/* 데이터 가져오기 */

const API_URL = `https://api.github.com/repos/vuejs/core/commits?per_page=3&sha=`

const branches = ['main', 'v2-compat']

const currentBranch = ref(branches[0])
const commits = ref(null)
// watchEffect는 즉시 실행됩니다. 즉, 현재 Branch값이 변경될때마다 다시 실행됩니다.
watchEffect(async()=>{
  const url = `${API_URL}${currentBranch.value}`
  commits.value = await(await fetch(url)).json()
})

function truncate(v){
  const newline = v.indexOf('\n')
  return newline > 0 ? v.slice(0, newline) : v
}

function formatDate(v){
  return v.replace(/T|Z/g, '')
}


/* 정렬과 필터가 있는 그리드 */
const searchQuery = ref('')
const gridColumns = ['name', 'power']

const gridData = [
  { name: 'Chuck Norris', power: Infinity },
  { name: 'Bruce Lee', power: 9000 },
  { name: 'Jackie Chan', power: 7000 },
  { name: 'Jet Li', power: 8000 }
]

 

/** Tree 뷰  */
import TreeItem from './TreeItem.vue'
const treeData = ref({
  name : 'My Tree',
  children : [
  {name : 'hello'},
  { name : 'World'},
  { name : 'child folder',
  children:[{name : 'hello' }, { name : 'world'}]},
  { name : 'child folder', children : [{ name : 'hello' },{ name : 'world'}]}
  ] 

})


/*모달 컴포넌트 */
import Modal from './Modal.vue'

const showModal = ref(false)
const msg = ref('Hello World!')
/* 트렌직션으로 리스트 구현하기 */
import { shuffle as _shuffle } from 'lodash-es'
const getInitialItems = () => [1,2,3,4,5] 
const items = ref(getInitialItems())
let id = items.value.length  + 1

function insert(){
  const i = Math.round(Math.random() * items.value.length)
  items.value.splice(i, 0, id++)
}

function reset(){
  items.value = getInitialItems()
  id = items.value.length +1
}

function shuffle(){
  items.value = _shuffle(items.value)
}

function remove(item){
  const i = items.value.indexOf(item)
  if ( i > -1){
    items.value.splice( i , 1)
  }  
}
/* todoMVC */
const STORAGE_KEY = 'vue-todomvc'
const filters = {
  all : (todos) => todos,
  active : (todos) => todos.filter((todo) => !todo.completed),
  completed : (todos) => todos.filter((todo)=> todo.completed)
}
// state
const todos = ref(JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'))
const visibility = ref('all')
const editedTodo = ref()

// derived state
const filteredTodos = computed(() =>
filters[visibility.value](todos.value))

const remaining = computed(()=> filters.active(todos.value).length)

// handle routing 
window.addEventListener('hashchange', onHashChange)
onHashChange()

//persist state
watchEffect(()=>{
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
})

function onHashChange(){
  const route = window.location.hash.replace(/#\/?/, '')
  if(filters[route]){
    visibility.value = route
  } else{
    window.location.hash = ''
    visibility.value = 'all'
  }
}
function toggleAll(e){
  todos.value.forEach((todo) => (todo.completed = e.target.checked))
}

function addTodo(e){
  const value = e.target.value.trim()
  if(value){
    todos.value.push({
      id : Date.now(),
      title : value,
      completed : false
    })
    e.target.value = ''
}
}
function removeTodo(todo){
  todos.value.splice(todos.value.indexOf(todo), 1)
}
let beforeEditCache = ''
function editTodo(todo){
  beforeEditCache = todo.title
  editedTodo.value = todo
}

function cancelEdit(todo){
  editedTodo.value = null
  todo.title = beforeEditCache
}

function doneEdit(todo){
  if(editedTodo.value){
    editedTodo.value = null
    todo.title = todo.title.trim()
    if(!todo.title) rmoveTodo(todo)
  }
}

function removeCompleted(){
  todos.value = filters.active(todos.value)
}



/**SVG 그래프 */
import PolyGraph from './PolyGraph.vue';
const newLabel=ref('')
const stats = reactive([
  { label: 'A', value: 100 },
  { label: 'B', value: 100 },
  { label: 'C', value: 100 },
  { label: 'D', value: 100 },
  { label: 'E', value: 100 },
  { label: 'F', value: 100 }
])

function add(e){
  e.preventDefault()
  if(!newLabel.value) return
  stats.push({
    label : newLabel.value,
    value : 100
  })
  newLabel.value = ''
}

function removeSVG(stat) {
  if(stat.length > 3){
    stats.splice(stats.indexOf(stat), 1)
  } else{
    alert("Can't delete more!")
  }
}
</script>


<template>
    <div>
      <div class="selectSection">
        <div >
          <ol>
            <TodoItem
            v-for="item in groceryList"
            :todo="item"
            :key="item.id"
            >
            </TodoItem>
          </ol>


            <h2>Text Input  </h2>
              <input v-model="text">
              {{text}}
              
            <h2>checkedbox</h2>
              <input type="checkbox" id="checkbox" v-model="checked">
              <label for="checkbox">Checked :{{checked }}</label>
                    
            <h2> Multi checkbox  </h2>         
              <input type="checkbox" id="jack" value="Jack" 
              v-model="checkedNames">
              
              <label for="jack">Jack</label>
              <input type="checkbox" id="john" value="John"
              v-model="checkedNames">
              
              <label for="john">
                John
              </label>
              <input type="checkbox" id="mike" value="Mike" 
              v-model="checkedNames">
              <label for="mike">Mike</label>
              <p>checkedNames : {{checkedNames}}</p>
              <h2>Radio</h2>
              <input type="radio" id="one" value="One" v-model="picked">
              <label for="one">One</label>
              
              
              <input type="radio" id="two" value="Two" v-model="picked">
              <label for="two">Two</label>
              <p>picked : {{picked}}</p>
              
              <h2>Select</h2>
              <select v-model="selected">
                <option disabled value="">select one</option>
                <option>A</option>
                <option>B</option>
                <option>C</option>
              </select>
              <p>selected : {{selected}}</p>
              
              <h2>Multi select</h2>
              <select v-model="multiSelected" multiple style="width: 100px;">
                <option>A</option>
                <option>B</option>
                <option>C</option>
              </select>
              <p>selected : {{multiSelected}}</p> 
              
              <h1>{{ message }}</h1> 
              <h1>{{message}}</h1>
              <button @click="reverseMessage">reverseMessage</button>
              <button @click="message += '!'">Append "!" 
              </button>
              <!-- <a href="www.naver.com" @click.prevent="notify">
                A link with e.preventDefault()
              </a> -->

              <p>
                <span :title="message">
            여기에 마우스를 두면 몇 초 후에 타이틀이 뜹니다!
                </span>
              </p>

            <p :class="{ red :isRed}" @click="toggleRed">
              이 문장은 빨간색입니다. 클릭하면 토글되어 검정색 글자가 됩니다.
            </p>

            <p :style="{color}" @click="toggleColor">
              이 문장을 클릭하면 초록색과 파랑색의 글자가 바뀝니다
            </p>

            <button @click="show = !show">Toggle list</button>
            <button @click="list.push(list.length + 1)">push number</button>
            <button @click="list.pop()">pop number</button>
            <button @click="list.reverse()">Reverse List</button>
            <ul v-if="show && list.length">
              <li v-for="(item, index) in  list" :key="index">{{ item }}</li>
            </ul>
            <p v-else-if="list.length"> list is not empty , but hidden</p>
            <p v-else>list is empty</p>
        </div>
      </div>
      
    <!--Markdown 편집기 -->

        <div class="MarkdownSection">
          <div class="editor">
            <textarea class="input" :value="input" @input="update">

            </textarea>
            <div class="output" v-html="output"></div>
          </div>
        </div>


    <!-- 데이터 가져오기  --> 
        <div class="listSection"  v-for="branch in branches" :key="branch">
          <input
            type="radio"
            :id="branch"
            :value="branch"
            name="branch"
            v-model="currentBranch" 
          />     
        <label :for="branch">{{ branch }}</label>  

          <p>vuejs/vue@{{ currentBranch }}</p>
          <ul>
              <!-- commits 반복에서 기본값 추가 -->
              <li v-for="{ html_url, sha, author, commit } in commits" :key="sha || html_url">
                <a :href="html_url" target="_blank" class="commit">
                  {{ sha ? sha.substring(0, 7) : 'N/A' }}
                </a>
                - 
                <span class="message">{{ commit?.message ? truncate(commit.message) : 'No message' }}</span><br>
                
                by 
                <span class="author">
                  <a :href="author?.html_url || '#'" target="_blank"> 
                    {{ commit?.author?.name || 'Unknown Author' }}
                  </a>
                </span>
                
                at 
                <span class="date">
                  {{ commit?.author?.date ? formatDate(commit.author.date) : 'Unknown Date' }}
                </span>
              </li>
          </ul> 
      </div>

        <div class="fliterSection">
              <!-- 정렬과 필터가 있는 그리드 -->
              <form id="search">
              Search <input name="query" v-model="searchQuery">
            </form>
            <DemoGrid
            :data="gridData"
            :columns="gridColumns"
            :filter-key="searchQuery">
            </DemoGrid>
        </div>

        <div class="treeSection">
          <!-- 트리뷰 -->
          <ul>
            <TreeItem class="item" :model="treeData"></TreeItem>
          </ul>
        </div>

        <div class="modalSection">
          <!--모달 컴포넌트-->
          <button id="show-modal" @click="showModal = true">
            show Modal
          </button>
          <Teleport to="body">
            <!--use the modal component, pass in the prop-->
            <modal :show="showModal" @close="showModal = false">
              <template #header>
                <h3>Custom Header</h3>
              </template>
            </modal>
          </Teleport>
        </div>

        <!--트렌직션으로 리스트 구현하기-->
        <div class="listSection">
          <button @click="insert">Insert at random index</button>
          <button @click="reset">Reset</button>
          <button @click="shuffle">Shuffle</button>
          <TransitionGroup tag="ul" name="fade" class="container">
            <li v-for="item in items" class="item" :key="item.id">
              {{ item }}
              <button @click="remove(item)">X</button>
            </li>
          </TransitionGroup>
        </div>

        <div class="todoSection"> 
        <!--Todo-->
          <section class="todoapp">
            <header class="header">
              <h1>Todos</h1>
              <input class="new-todo" autofocus placeholder="What needs to be done?" @keyup.enter="addTodo" />
            </header>
            <section class="main" v-show="todos.length">
              <input id="toggle-all" class="toggle-all" type="checkbox" :checked="remaining === 0" 
              @change="toggleAll" />
              <label for="toggle-all">Mark all as complete</label>
              <ul class="todo-list">
                <li v-for="todo in filteredTodos" class="todo" :key="todo.id" :class="{ completed: todo.completed, editing: todo === editedTodo }">
                  <div class="view">
                    <input class="toggle" type="checkbox" v-model="todo.completed" />
                    <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
                    <button class="destroy" @click="removeTodo(todo)"></button>
                  </div>
                  <input
                    v-if="todo === editedTodo"
                    class="edit"
                    type="text"
                    v-model="todo.title"
                    @blur="doneEdit(todo)"
                    @keyup.enter="doneEdit(todo)"
                    @keyup.escape="cancelEdit(todo)"
                  />
                </li>
              </ul>
            </section>

              <footer class="footer" v-show="todos.length">
                <span class="todo-count">
                  <strong>{{ remaining }}</strong>
                  <span>{{ remaining === 1 ? 'item' : ' items' }} left</span>
                </span>
                <ul class="filters">
                  <li>
                    <a href="#/all" :class="{ selected: visibility === 'all' }">All</a>
                  </li>
                  <li>
                    <a href="#/active" :class="{ selected: visibility === 'active' }">Active</a>
                  </li>
                  <li>
                    <a href="#/completed" :class="{ selected: visibility === 'completed' }">Completed</a>
                  </li>
                </ul>
                <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">
                  Clear completed
                </button>
              </footer>
          </section>
        </div>
    </div>
</template>




<style>
@import "https://unpkg.com/todomvc-app-css@2.4.1/index.css";

.selectSection,
.modalSection,
.treeSection,
.fliterSection,
.listSection,
.todoSection{	
    border-radius: 15px;
    border-bottom: 3px solid darkgray;
    width: 100%;
    margin-bottom: 1em;
    padding: 20px 10px;
}


</style>

 