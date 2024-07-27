<template>
  <div class="lg:p-10 p-4">
   

    <div >
      <div class="flex justify-between text-indigo-950">
        <h1 class="text-3xl font-bold ">Todo</h1>
      <div class="flex space-x-2" >
        <p class=" font-semibold text-lg" v-if="!user"> User</p>
        <p>{{user.email}}</p>
        <div class=" ">
          <Icon name="memory:user-box" class=" w-6 h-6"/>
        </div>
      </div>
      </div>
      <hr>
      <div class="flex justify-between py-5">
          <h1 class="text-indigo-950 font-semibold">
            <NuxtTime :datetime="Date.now()" year="numeric" month="long" day="numeric" />
          </h1>
         <div class="grid gap-3">
          <div v-if="formOpened" class="grid gap-2">
            <form action="" @submit.prevent="addTask" class="grid gap-2">Add a todo todo
              <input v-model="todo" type="text" placeholder="Enter Todo Task" required class="border-2 border-red-400 rounded-md p-1">
              <input v-model="datetime" type="datetime-local" placeholder="Year-Month-Day & Time" required class="border-2 border-red-400 rounded-md p-1">
             <button type="submit"  class="bg-indigo-600 p-2  text-white rounded-lg">Add Task</button>
             <button @click="formOpened=false" class="bg-red-400 p-1 rounded-md">close</button>
            </form>
           
           
          </div>
          <div class="" v-if="!formOpened">
              <button @click="openForm" class="bg-indigo-600 p-2 w-fit h- text-white rounded-lg">New Task</button>
          </div>
         </div>
         
      </div>
    </div>
    <div v-if="error">{{ error }}</div>
    <div v-if="isLoading">Loading...</div>
    <div class="grid lg:grid-cols-3 gap-4 text-indigo-950 md:grid md:grid-cols-2  ">
      <div v-for="todo in todos" :key="todo.id"
      :class="[
        'border-2 rounded-lg border-slate-200 w-96 h-20 p-3 md:w-11/12 md:h-20',
        { 'bg-indigo-600 text-white': todo.completed, 'bg-white': !todo.completed }
      ]" >
          <div class="flex justify-between">
              <div>
                <div class="flex space-x-2" v-if="!editFormOpened">
                  <Icon @click="completedTodo(todo)"  name="fluent:square-48-filled" :class="['pt-8 hover:cursor-pointer',{' ': todo.completed, 'text-indigo-600': !todo.completed}]"/>
                  <p class="font-semibold lg:text-lg">{{ todo.todo }}</p>
              </div>
              <div class="">
                {{ todo.datetime }}
              </div>
              </div>
              <div class="flex space-x-2"> 
                <div  v-if="!editFormOpened">
                  <Icon @click="editTodo(todo)" name="mdi:edit" class=" hover:cursor-pointer hover:text-yellow-400"/>
                </div>
                <div>

              <div v-if="editFormOpened" class="fixed inset-0 bg-gray-500 bg-opacity-75 flex justify-center items-center">
                <div class="bg-white p-4 rounded-md">
                  <form @submit.prevent="updateTodo" class="grid gap-2">
                    <h2>Edit Todo</h2>
                    <input v-model="editTodoData.todo" type="text" placeholder="Enter Todo Task" required class="border-2 border-red-400 rounded-md p-1">
                    <input v-model="editTodoData.datetime" type="datetime-local" placeholder="Year-Month-Day & Time" required class="border-2 border-red-400 rounded-md p-1">
                    <button type="submit" class="bg-indigo-600 p-2 w-fit text-white rounded-lg">Update Task</button>
                  </form>
                  <button @click="closeEditForm" class="bg-red-400 p-1 rounded-md mt-2">Close</button>
                </div>
              </div>

                </div>
                <div v-if="!editFormOpened">
                  <Icon @click="deleteTodo(todo)" name="ic:round-delete" class="hover:cursor-pointer hover:text-red-400"/>
                </div>
               
              </div>
          </div>
          
        </div>
    </div>
  </div>
</template>

<script setup>
definePageMeta({
  middleware: ["auth"],
})
const supabase = useSupabaseClient()
const user= useSupabaseUser()

const todos = ref([])
const todo = ref('')
const datetime = ref('')



// addTask form (open)
const formOpened = ref(false)

function openForm() {
  formOpened.value = true
}
// edit form
const editTodoData = reactive({
  id: null,
  todo: '',
  datetime: ''
})
const editFormOpened = ref(false)

function closeEditForm() {
  editFormOpened.value = false
}


// add todo 
 async function addTask() {
  try {
    if (!user) {
      alert('user not authnticated')
      return
    }
 // Insert the todo data into the "todos" table
    const {data, error } = await supabase
    .from ('todos')
    .insert([{
     userId: user.value.id,
     todo:todo.value, 
     datetime: datetime.value, 
     completed:false}])

    if (error) {
      console.error(error);
    }else{
      console.log('todo is saved:',data);
      getTodoData()
      formOpened.value= false
    }
  } catch (error) {
    console.error(error);
  }
  todo.value = ''
  datetime.value = ''
 }

//  show the todo data from the supabase table
 async function getTodoData() {
  try {
    const {data, error} = await supabase
    .from('todos')
    .select('*')
    .eq('userId', user.value.id) //to filter tasks by their original users only

    if (error) {
      console.error('Error retrieving todo data:', error);
      return
    }
    console.log('todo data:', data);
    todos.value = data
  } catch (error) {
    console.error('Error retrieving todo data:', error.message);
  }
 }
//  edit todo data 
 function editTodo(todo) {
  editTodoData.id = todo.id
  editTodoData.todo = todo.todo
  editTodoData.datetime = todo.datetime
  editFormOpened.value = true

}
// update todo
async function updateTodo() {
  try {
    const {data, error} = await supabase
    .from('todos')
    .update({
      todo: editTodoData.todo,
      datetime: editTodoData.datetime
    })
    .eq('id', editTodoData.id)

    if(error) throw error
    console.log('todo updated', data);
    getTodoData()
    closeEditForm()

  } catch (error) {
    console.error('error updateing todo:', error);
  }
}

// toggle complete
async function completedTodo(todo) {
  todo.completed = !todo.completed

  try {
    const {data, error} = await supabase
    .from('todos')
    .update({completed: todo.completed})
    .eq('id', todo.id)
    .single()

    if (error) throw error
    console.log('Todo updated:', data);

  } catch (error) {
    console.error('error updating todo', error);
  }
}

// delete todo 
async function deleteTodo(todo){
  try {
    const {data, error} = await supabase
    .from('todos')
    .delete()
    .eq('id', todo.id)
    

    if (error) 
   { console.log('error deleting todo:', data);}
   else{
    console.log('todo deleted:', data);
   }
    // Update the todo data in the UI
   getTodoData()

  } catch (error) {
    console.error('error deleting todo', error);
  }
}

onMounted(() => {
  getTodoData()
})
</script>