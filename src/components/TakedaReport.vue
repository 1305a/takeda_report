<template>
  <div class="date-wrapper">
    <div class="demo-date-picker">
      <div class="block">
        <span class="demonstration">Период времени</span>
        <el-date-picker
          v-model="timePeriod"
          type="daterange"
          range-separator="To"
          start-placeholder="Start date"
          end-placeholder="End date"
          :size="'large'"
        />
      </div>
    </div>
  </div>

  <el-button @click="getAnswers" type="primary">Сформировать</el-button>

  <el-table :data="tableData" style="width: 100%">
    <el-table-column prop="sessionId" label="Session id" width="300" />
    <el-table-column prop="startTime" label="Время начала" width="300" />
    <el-table-column prop="endTime" label="Время завершения" width="300" />
    <el-table-column prop="totalPoints" label="Набрано баллов" />
  </el-table>

</template>

<script setup>
  import { ref } from 'vue'
  import axios from 'axios'

  const timePeriod = ref('')
  const testId = ref(25)
  const serverURL = "https://dev1.med-game.ru/api/v1"
  
  const shortcuts = [
  {
    text: 'Last week',
    value: () => {
      const end = new Date()
      const start = new Date()
      start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
      return [start, end]
    },
  },
  {
    text: 'Last month',
    value: () => {
      const end = new Date()
      const start = new Date()
      start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
      return [start, end]
    },
  },
  {
    text: 'Last 3 months',
    value: () => {
      const end = new Date()
      const start = new Date()
      start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
      return [start, end]
    },
  },
]

const tableData = ref([])

//Загружаем данные
const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MDM5ODQxNjYuMDg3MjgsImlhdCI6MTcwMjkwNDE2Ni4wODcyODEsInVzZXJuYW1lIjoiYWRtaW4iLCJyb2xlcyI6WyJhZG1pbi1wYXJ0IiwibWFuYWdlLXVzZXJzIiwibWFuYWdlLXJvbGVzIl19.hgbMeRQGghGIarM_QhQ8paIwXPWV82wqdP-nVbc8jts'

const getAnswers = async () => {
  axios({
    url: `${serverURL}/admin/test/${testId.value}/answers`,
    method: 'get',
    headers: { Authorization: `Bearer ${token}` }    
  })
  .then((answersData) => {   
    const filteredData = answersData.data.data.filter((x) => !timePeriod.value[0] || ((timePeriod.value[0] && timePeriod.value[0] <= new Date(x.created_at)) && (( timePeriod.value[1] && timePeriod.value[1] >= new Date(x.created_at)) )))

    const newArray = filteredData.map((x) => {
      let startTime = ""
      let endTime = ""
      if (x.body.position === "start") {
        startTime = new Date(x.created_at)
      }
      if (x.body.position === "end") {
        endTime = new Date(x.created_at)
      }
      return {
        "sessionId": x.user_session_id,
        "startTime": startTime,
        "endTime": endTime,
        "totalPoints": x.body.totalPoints
      }
    })
    
    tableData.value = newArray

  })
  .catch(function (error) {
    console.log(error);
  })
}



</script>

<style scoped>

  .date-wrapper{
    width: 500px;
  }
  .demo-date-picker {
    display: flex;
    width: 100%;
    padding: 0;
    flex-wrap: wrap;
  }

  .demo-date-picker .block {
    padding: 30px 0;
    text-align: center;
    border-right: solid 1px var(--el-border-color);
    flex: 1;
  }

  .demo-date-picker .block:last-child {
    border-right: none;
  }

  .demo-date-picker .demonstration {
    display: block;
    color: var(--el-text-color-secondary);
    font-size: 14px;
    margin-bottom: 20px;
  }
</style>