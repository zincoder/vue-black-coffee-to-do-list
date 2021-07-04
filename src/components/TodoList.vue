<!--localStorage 값형태 json object형식으로 변경 -->
<!--localStorage내부 completed는 변경 안하고 dynamicLists에 completed만 변경하고 있음 그래서 새로고침시 다 false -->
<template>
   <div class="todoapp">
      <h1>TODOS</h1>
      <input
        id="new-todo-title"
        class="new-todo"
        placeholder="할일을 추가해주세요"
        autofocus
        @keyup.enter="enterKeyFunction"
        v-model="todoInput"
      />
      <ul id="todo-list" class="todo-list" v-if="showTodoList"> 
          <li v-for="(list, index) in dynamicLists[0]" :key="index" v-bind:class="{completed : isCompletedArr[index] === true, hidden : isHidden[index]}">
            <div class="view">
            <input class="toggle" type="checkbox" @change="toggleCompleted(index)" v-model="isCompletedArr[index]"  value="index"/>
            <label class="label">{{list.item}}</label>
            <button class="destroy" @click="toggleDestroyed(index)"></button> 
            </div>
            <input class="edit" value="새로운 타이틀" />
        </li>
      </ul>
      <main>
        <input class="toggle-all" type="checkbox" v-model="completedChecked"/>
        <ul id="todo-list" class="todo-list"></ul>
        <div class="count-container">
          <span class="todo-count">총 <strong id="todoItemTotCnt">{{totalCount}}</strong> 개</span>
          <ul class="filters" id="filtersGroup">
            <li>
              <a v-bind:class="{all, filterBtn, selected:isSelected[0]}" href="#" id="allBtn" @click="toggleFiltered('all')">전체보기</a>
            </li>
            <li>
              <a v-bind:class="{active, filterBtn, selected:isSelected[1]}" href="#active" @click="toggleFiltered('active')">해야할 일</a>
            </li>
            <li>
              <a v-bind:class="{completed, filterBtn, selected:isSelected[2]}" href="#completed" @click="toggleFiltered('completed')">완료한 일</a>
            </li>
          </ul>
        </div>
      </main>
    </div>
</template>

<script>
//import DynamicList from './DynamicList.vue';
export default {
    // components:{
    //     DynamicList
    // },
    data(){
        return {
            todoInput :'',  
            dynamicLists:[{}],
            completedChecked:[],
            isShowTodoList:false,
            isCompletedArr:{},
            isHidden:{},
            isSelected:[true],
            totalCount:0,
            toggleStatus:'',
            localStorageLength:0,
            now:"0000년00월00일00시00분00초", //입력시간세팅
        }
    },
    created(){
        let totCnt = 0;
        this.localStorageLength = localStorage.length
        for(let i=0; i<this.localStorageLength; i++){
            if(localStorage.key(i).indexOf('loglevel:') === -1){
             this.dynamicLists[0][localStorage.key(i)] = JSON.parse(localStorage.getItem(localStorage.key(i)));
             if( this.dynamicLists[0][localStorage.key(i)].completed){
                 this.isCompletedArr[localStorage.key(i)] = true;
             }
             totCnt++;
            }
        }
        this.totalCount = totCnt;
    },
    computed:{

    },
    methods:{
        getCurrTime(){
            let date = new Date();
            this.now = `${date.getFullYear()}년 ${date.getMonth()+1}월 ${date.getDate()}일 ${date.getHours()}시 ${date.getMinutes()}분 ${date.getSeconds()}초 `;
                      
        },
        enterKeyFunction(){
            this.getCurrTime();
            if(this.todoInput.trim()!=="" && !localStorage.getItem(this.todoInput) ){
            let value ={
                        item:this.todoInput
                        ,completed:false
                        ,time:this.now
            };
             localStorage.setItem(this.todoInput,JSON.stringify(value));
             this.dynamicLists[0][this.todoInput] = JSON.parse(localStorage.getItem(this.todoInput));
            //  localStorage.setItem(this.todoInput,this.todoInput);
            //  this.dynamicLists.push(localStorage.getItem(this.todoInput));
            }
            console.log(this.dynamicLists[0])
            console.log(this.dynamicLists)
            this.todoInput = '';
            this.isNotHiddenCnt();
        },
        showTodoList(){
            if(this.dynamicLists.length > 0){
                this.isShowTodoList = true;
            }else{
                this.isShowTodoList = false;
            }
            return this.isShowTodoList;
        },
        toggleCompleted(idx){
              if(!this.isCompletedArr[idx]){
                  localStorage.removeItem(idx);
                   let value = {
                        item:this.dynamicLists[0][idx].item
                        ,completed:false
                        ,time:this.dynamicLists[0][idx].time
                    };
                  localStorage.setItem(idx,JSON.stringify(value));
                  this.dynamicLists[0][idx].completed = false
              } else{
                   localStorage.removeItem(idx);
                   let value = {
                        item:this.dynamicLists[0][idx].item
                        ,completed:true
                        ,time:this.dynamicLists[0][idx].time
                    };
                  localStorage.setItem(idx,JSON.stringify(value));
                  this.dynamicLists[0][idx].completed = true
              }
            //  if(this.isCompletedArr[idx]){
            //      this.isCompleted = true;
            //  }else{
            //      this.isCompleted = false;
            //  }
            this.toggleFiltered(this.toggleStatus);
        },
        toggleDestroyed(idx){
            delete this.dynamicLists[0][idx];
            localStorage.removeItem(idx);
            
        },
        toggleFiltered(status){
            let dynamicListKeys = Object.keys(this.dynamicLists[0]);
            this.toggleStatus = status;
            for(let i=0; i<dynamicListKeys.length; i++){             
                this.isHidden[dynamicListKeys[i]]=false;
                this.isSelected.splice(0,0,true);
                this.isSelected.splice(1,0,false);
                this.isSelected.splice(2,0,false);
                if(this.toggleStatus  === 'active'){
                    this.isSelected.splice(0,0,false);
                    this.isSelected.splice(1,0,true);
                    this.isSelected.splice(2,0,false);
                     if(this.dynamicLists[0][dynamicListKeys[i]].completed){
                         console.log(dynamicListKeys[i]);
                        this.isHidden[dynamicListKeys[i]]=true
                    }
                 }
                 else if(this.toggleStatus === 'completed'){
                    this.isSelected.splice(0,0,false);
                    this.isSelected.splice(1,0,false);
                    this.isSelected.splice(2,0,true);
                     if(!this.dynamicLists[0][dynamicListKeys[i]].completed){
                        this.isHidden[dynamicListKeys[i]]=true
                    }
                 }
                }

            // if(this.toggleStatus  === 'active'){
            //     for(let i=0; i< dynamicListKeys.length; i++){
            //         if(this.isHidden[dynamicListKeys[i]]){
            //             this.isHidden[dynamicListKeys[i]]=true
            //         }
            //     }
            // } else if(this.toggleStatus  === 'completed'){
            //      for(let i=0; i< dynamicListKeys.length; i++){
            //         if(!this.isHidden[dynamicListKeys[i]]){
            //             this.isHidden[dynamicListKeys[i]]=true
            //         }
            //     }
            // }
            this.isNotHiddenCnt();
        },
         isNotHiddenCnt(){
            let hiddenCnt = 0 ;
            const isHiddenLength = Object.keys(this.isHidden).length;
            const todoListLength = Object.keys(this.dynamicLists[0]).length;
            for(let i=0; i<isHiddenLength; i++){
                if(this.isHidden[Object.keys(this.dynamicLists[0])[i]]){
                    hiddenCnt++;
                }
            }
            this.totalCount = todoListLength - hiddenCnt;
            return this.totalCount;
        }
    }
}
</script>

<style scoped>
html,
body {
    margin: 0;
    padding: 10px;
}

button {
    margin: 0;
    padding: 0;
    border: 0;
    background: none;
    font-size: 100%;
    vertical-align: baseline;
    font-family: inherit;
    font-weight: inherit;
    color: inherit;
    -webkit-appearance: none;
    appearance: none;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}

body {
    font: 14px 'Helvetica Neue', Helvetica, Arial, sans-serif;
    line-height: 1.4em;
    background: #f5f5f5;
    color: #4d4d4d;
    min-width: 230px;
    max-width: 550px;
    margin: 0 auto;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    font-weight: 300;
}

:focus {
    outline: 0;
}

.hidden {
    display: none;
}

.todoapp {
    background: #fff;
    margin: 130px 0 40px 0;
    position: relative;
    box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.2),
    0 25px 50px 0 rgba(0, 0, 0, 0.1);
}

.todoapp input::-webkit-input-placeholder {
    font-style: italic;
    font-weight: 300;
    color: #e6e6e6;
}

.todoapp h1 {
    position: absolute;
    top: -125px;
    width: 100%;
    font-size: 60px;
    text-align: center;
    color: dimgray;
    font-weight: 100;
    font-family: Helvetica Neue, Helvetica, Arial, sans-serif;
}

.new-todo,
.edit {
    position: relative;
    margin: 0;
    width: 100%;
    font-size: 24px;
    font-family: inherit;
    font-weight: inherit;
    line-height: 1.4em;
    border: 0;
    color: inherit;
    padding: 6px;
    box-shadow: inset 0 -1px 5px 0 rgba(0, 0, 0, 0.2);
    box-sizing: border-box;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}

.new-todo {
    padding: 16px 16px 16px 60px;
    border: none;
    background: rgba(0, 0, 0, 0.003);
    box-shadow: inset 0 -2px 1px rgba(0,0,0,0.03);
}

main {
    position: relative;
    z-index: 2;
    border-top: 1px solid #e6e6e6;
}

.toggle-all {
    width: 1px;
    height: 1px;
    border: none;
    opacity: 0;
    position: absolute;
    right: 100%;
    bottom: 100%;
}

.toggle-all + label {
    width: 60px;
    height: 34px;
    font-size: 0;
    position: absolute;
    top: -52px;
    left: -13px;
    -webkit-transform: rotate(90deg);
    transform: rotate(90deg);
}

.toggle-all + label:before {
    content: '❯';
    font-size: 22px;
    color: #e6e6e6;
    padding: 10px 27px 10px 27px;
}

.toggle-all:checked + label:before {
    color: #737373;
}

.todo-list {
    margin: 0;
    padding: 0;
    list-style: none;
}

.todo-list li {
    position: relative;
    font-size: 24px;
    border-bottom: 1px solid #ededed;
}

.todo-list li:last-child {
    border-bottom: none;
}

.todo-list li.editing {
    border-bottom: none;
    padding: 0;
}

.todo-list li.editing .edit {
    display: block;
    width: calc(100% - 43px);
    padding: 12px 16px;
    margin: 0 0 0 43px;
}

.todo-list li.editing .view {
    display: none;
}

.todo-list li .toggle {
    text-align: center;
    width: 40px;
    height: auto;
    position: absolute;
    top: 0;
    bottom: 0;
    margin: auto 0;
    border: none;
    -webkit-appearance: none;
    appearance: none;
}

.todo-list li .toggle {
    opacity: 0;
}

.todo-list li .toggle + label {
    background-image: url('data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A//www.w3.org/2000/svg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%23ededed%22%20stroke-width%3D%223%22/%3E%3C/svg%3E');
    background-repeat: no-repeat;
    background-position: center left;
}

.todo-list li .toggle:checked + label {
    background-image: url('data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A//www.w3.org/2000/svg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%23bddad5%22%20stroke-width%3D%223%22/%3E%3Cpath%20fill%3D%22%235dc2af%22%20d%3D%22M72%2025L42%2071%2027%2056l-4%204%2020%2020%2034-52z%22/%3E%3C/svg%3E');
}

.todo-list li label {
    word-break: break-all;
    padding: 15px 15px 15px 60px;
    display: block;
    line-height: 1.2;
    transition: color 0.4s;
}

.todo-list li.completed label {
    color: #d9d9d9;
    text-decoration: line-through;
}

.todo-list li .destroy {
    display: none;
    position: absolute;
    top: 0;
    right: 10px;
    bottom: 0;
    width: 40px;
    height: 40px;
    margin: auto 0;
    font-size: 30px;
    color: #cc9a9a;
    margin-bottom: 11px;
    transition: color 0.2s ease-out;
    cursor: pointer;
}

.todo-list li .destroy:hover {
    color: #af5b5e;
}

.todo-list li .destroy:after {
    content: '×';
}

.todo-list li:hover .destroy {
    display: block;
}

.todo-list li .edit {
    display: none;
}

.todo-list li.editing:last-child {
    margin-bottom: -1px;
}

.count-container {
    color: #777;
    padding: 10px 15px;
    height: 20px;
    text-align: center;
    border-top: 1px solid #e6e6e6;
}

.count-container:before {
    content: '';
    position: absolute;
    right: 0;
    bottom: 0;
    left: 0;
    height: 50px;
    overflow: hidden;
    box-shadow: 0 1px 1px rgba(0, 0, 0, 0.2),
    0 8px 0 -3px #f6f6f6,
    0 9px 1px -3px rgba(0, 0, 0, 0.2),
    0 16px 0 -6px #f6f6f6,
    0 17px 2px -6px rgba(0, 0, 0, 0.2);
}

.todo-count {
    float: left;
    text-align: left;
}

.todo-count strong {
    font-weight: 300;
}

.filters {
    margin: 0;
    padding: 0;
    list-style: none;
    position: absolute;
    right: 0;
    left: 0;
}

.filters li {
    display: inline;
}

.filters li a {
    color: inherit;
    margin: 3px;
    padding: 3px 7px;
    text-decoration: none;
    border: 1px solid transparent;
    border-radius: 3px;
}

.filters li a:hover {
    border-color: rgba(175, 47, 47, 0.1);
}

.filters li a.selected {
    border-color: rgba(175, 47, 47, 0.2);
}

.clear-completed, html .clear-completed:active {
    float: right;
    position: relative;
    line-height: 20px;
    text-decoration: none;
    cursor: pointer;
}

.clear-completed:hover {
    text-decoration: underline;
}

.info {
    margin: 65px auto 0;
    color: #bfbfbf;
    font-size: 10px;
    text-shadow: 0 1px 0 rgba(255, 255, 255, 0.5);
    text-align: center;
}

.info p {
    line-height: 1;
}

.info a {
    color: inherit;
    text-decoration: none;
    font-weight: 400;
}

.info a:hover {
    text-decoration: underline;
}

</style>