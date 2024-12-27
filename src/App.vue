<script setup>
  import { computed, ref } from 'vue'
  import RowModal from './components/RowModal.vue'


  const list_title = ref('Light/Web Novels List')
  const status_dict = ref({
    'Finished':'green', 
    'Dropped':'red', 
    'On Hold':'cyan', 
    'Current':'white'
  })

  let entry_id = 0

  // const novel_list = ref([
  //   {
  //     id: entry_id++,
  //     title: 'testTitle', 
  //     chapter: '99',
  //     read_status: 'finished'
  //   },
  //   {
  //     id: entry_id++,
  //     title: 'abracadabra', 
  //     chapter: '',
  //     read_status: 'dropped'
  //   },
  //   {
  //     id: entry_id++,
  //     title: 'q', 
  //     chapter: '1',
  //     read_status: ''
  //   },
  //   {
  //     id: entry_id++,
  //     title: 'John Wick', 
  //     chapter: 'Chpt 7',
  //     read_status: 'On hold; looks interesting'
  //   }
  // ])
  const novel_list = ref([])

  const title = ref('')
  const chapter = ref('')
  const read_status = ref('')

  const filters = ref(["Finished", "Dropped", "On Hold", "Current"])
  const filtered_novel_list = computed(() => {
    return novel_list.value.filter(
      (e) => filters.value.includes(getStatus(e.read_status))
    )
  })

  const show_row_settings = ref(false)
  const current_entry = ref({
    id: -999,
    title: '',
    chapter: '',
    read_status: '',
  })

  const file_lines = ref([])
  

  function resetEntry() {
    title.value =''
    chapter.value =''
    read_status.value = ''
  }

  
  function getStatus(read_status) {
    if (read_status == undefined) {
      alert('ERROR: undefined read_status')
      return 'Current';
    }
    if (read_status.match(/^Finished.*/i)) {
      return 'Finished';
    }
    else if (read_status.match(/^Dropped.*/i)) {
      return 'Dropped';
    }
    else if (read_status.match(/^On Hold.*/i)) {
      return 'On Hold';
    }
    else {
      return 'Current';
    }
  }

  function addEntry() {
    novel_list.value.push({
      id: entry_id++, 
      title: title.value, 
      chapter: chapter.value, 
      read_status: read_status.value
    })
    resetEntry()
  }

  function removeEntry(selected_e) {
    novel_list.value = novel_list.value.filter(
      (e) => e.id !== selected_e.id
    )
  }

  function setCurrentEntry(selected_e) {
    current_entry.value = {
      id: selected_e.id,
      title: selected_e.title,
      chapter: selected_e.chapter,
      read_status: selected_e.read_status
    }
  }

  function updateEntry(selected_e) {
    const i = novel_list.value.findIndex((e) => e.id === selected_e.id)
    novel_list.value[i].title = selected_e.title
    novel_list.value[i].chapter = selected_e.chapter
    novel_list.value[i].read_status = selected_e.read_status
  }

  function saveToLocal() {
    localStorage.setItem('saved_novel_list', JSON.stringify(novel_list.value))
    localStorage.setItem('current_id', entry_id)
  }

  function loadFromLocal() {
    novel_list.value = JSON.parse(localStorage.getItem('saved_novel_list'))
    entry_id = parseInt(localStorage.getItem('current_id'))
  }

  function addEntriesFromFile(replaceTable) {
    if (replaceTable) {
      entry_id = 0
      novel_list.value = []
    }
    let file_len = file_lines.value.length
    file_len -= (file_len % 3)
    for (let i = 0; i < file_len; i = i + 3) {
      title.value = file_lines.value[i]
      chapter.value = file_lines.value[i+1]
      read_status.value = file_lines.value[i+2]
      addEntry()
    }
  }

  async function loadFileToTable() {
    file_lines.value = null
    try {
      const response = await fetch('../web_novels_list.txt')
      const data = await response.text()
      file_lines.value = data.split(/\r?\n/)
      addEntriesFromFile(true)

    } catch (err) {
      console.error(err)
      alert(err)
    }
  }

  function downloadTable() {
    const table_str = convertTableToString()
    const blob = new Blob([table_str], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'downloaded_table.txt';
    a.click();
    URL.revokeObjectURL(url);
  }

  function convertTableToString() {
    let content = ''
    for (let i = 0; i < novel_list.value.length; i++) {
      content += novel_list.value[i].title + '\n' 
              + novel_list.value[i].chapter + '\n' 
              + novel_list.value[i].read_status + '\n'
    }
    return content.trimEnd()
  }

</script>


<template>
  
  <RowModal 
    :show="show_row_settings" 
    :entry="current_entry"
    @close="show_row_settings = false"
    @remove="removeEntry(current_entry)"
    @update="updateEntry(current_entry)">
  </RowModal>
  
  <header>
    <h1>{{ list_title }}</h1>
  </header>
  
  <hr>

  <div class="center">
    <button class="save-button" @click="saveToLocal">Save Local</button>
    <button class="load-button" @click="loadFromLocal">Load Local</button>
    ||
    <button class="save-button" @click="downloadTable">Save to File</button>
    <button class="load-button" @click="loadFileToTable">Load from File</button>
  </div>
  
  <hr>

  <br>

  <form class="center" @submit.prevent="addEntry">
    <input v-model="title" required placeholder="Enter title">
    <input v-model="chapter" placeholder="Enter current chapter">
    <input v-model="read_status" placeholder="Enter reading status">
    <input type="submit" value="Add Entry">
  </form>

  <div class="center" id="filters">
    <label for="filters">Status filters: </label>
    <input type="checkbox" id="finished" value="Finished" v-model="filters">
    <label for="finished" :class="status_dict['Finished']" >Finished</label>

    <input type="checkbox" id="dropped" value="Dropped" v-model="filters">
    <label for="dropped" :class="status_dict['Dropped']">Dropped</label>

    <input type="checkbox" id="onhold" value="On Hold" v-model="filters">
    <label for="onhold" :class="status_dict['On Hold']">On Hold</label>

    <input type="checkbox" id="current" value="Current" v-model="filters">
    <label for="current" :class="status_dict['Current']">Current</label>
  </div>

  <div class="table-scroll">
    <table>
      <thead>
        <tr>
          <th class="title-col">Title</th>
          <th class="chapter-col">Current Volume/Chapter</th>
          <th class="status-col">Status</th>
        </tr>
      </thead>
      <tbody>
        <tr :class="status_dict[getStatus(e.read_status)]" 
        class="row-hover" 
        
        id="show-row-modal"
        @click="show_row_settings = true; setCurrentEntry(e)"
        v-for="e in filtered_novel_list" :key="e.id">
          <td>{{ e.id }} - {{ e.title }}</td>
          <td>{{ e.chapter }}</td>
          <td>{{ e.read_status }}</td>
          <!-- <td><button @click="removeEntry(e)">X</button></td> -->
        </tr>
      </tbody>
    </table>
  </div>

</template>

<style scoped>
  * {
    box-sizing: border-box;
  }
  .green {
    background-color: #00ff00;
  }
  .red {
    background-color: #ff0000;
  }
  .cyan {
    background-color: #00ffff;
  }
  .white {
    background-color: #ffffff;
  }
  .center {
    margin-left: auto;
    margin-right: auto;
    text-align: center;
  }
  .title-col {
    width: 50%;
  }
  .chapter-col {
    width: 25%;
  }
  .status-col {
    width: 25%;
  }

  .row-hover:hover {
    background-color: yellow;
    
  }

  .save-button {
    box-sizing: border-box;
    background-color: lightgreen;
    border-radius: 12px;
    padding: 10px;
    margin: 5px;
    font-size: 16px;
  }
  .load-button {
    box-sizing: border-box;
    background-color: lightblue;
    border-radius: 12px;
    padding: 10px;
    margin: 5px;
    font-size: 16px;
  }

  .table-scroll {
    box-sizing: border-box;
    overflow-y: auto;
    max-height: 50vh;
    width: 80%;
    margin: 50px auto;
  }

  #filters {
    margin: 20px;
    font-size: 18px;
  }

  h1 {
    text-align: center;
  }
  form {
    margin: 10px auto;
  }
  input {
    margin-right: 5px;
  }
  table, th, td {
    margin: 10px;
    border: 2px solid;
    border-collapse: collapse;
    padding: 5px;
  }
  table {
    box-sizing: border-box;
    margin: 0;
    width: 100%;
    height: auto;
  }
  thead {
    background-color: lightgray;
  }
  
</style>