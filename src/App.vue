<script setup>
import { ref,watch,h } from 'vue'
import {ElNotification} from 'element-plus'
let uid = ()=>{
  return (new Date().getTime() * Math.random() * 100000)
}

const initialData = () => {
  return {
    name: "",
    score: 0,
    class: "A",
    id: uid()
  }
};

const filterInitalData = () =>{
  return {
    min:0,
    max:0,
    class:"A"
  }
}

const alertFun = (content) =>{
  ElNotification({
    message: content,
    type:"warning"
  });
}



const record = ref(initialData());
const originalArray = ref([]);

const createRecord = () => {

  if(record.value.name==""){
    alertFun(getResource("missingNameContent"))
    return;
  }

  originalArray.value.push({ ...record.value });
  console.log("push:",record.value)
  Object.assign(record.value, initialData());
}

const deleteRow = (id) => {
  //slice if filter will have problem
  let index = originalArray.value.findIndex(object=>{
    return object.id === id;
  })
  console.log("index",index)
  if(index===-1){
    return;
  }
  originalArray.value.splice(index, 1)

  clearFilter()
}


//array for display
const displayArray = ref([]);
watch(
  () => originalArray.value,
  () => { displayArray.value = [...originalArray.value] },
  {deep: true}
)




//filter
const filter = ref(filterInitalData());

const filterRecord= () =>{
  //filter rules
  let condition  = filter.value;


  //check min and max 
  if(parseInt(condition.min) > parseInt(condition.max)){
    alertFun(getResource("minLargeThanMax"))
    return
  }


  console.log(condition);
  var newArray = originalArray.value.filter(el=>{
    console.log(el)
    return parseInt(el.score) >= parseInt(condition.min) &&
           parseInt(el.score) <= parseInt(condition.max) &&
           el.class === condition.class
  })

   displayArray.value = newArray;

   buttonToggle.value = false;
}

const clearFilter = ()=>{
  displayArray.value = [...originalArray.value];
  Object.assign(filter.value, filterInitalData());

  buttonToggle.value = true;
}

//filter and clear toggle
const buttonToggle = ref(true);




//language switch
const lang = ref("eng")
const langOptions = ref([{"key":"zh","label":"中文"},{"key":"eng","label":"English"}])
const zh ={
  addRecord:"增加紀錄",
  searching:"過濾條件",
  name:"名字",
  score:"分數",
  class:"等級",
  filter:"過濾",
  clear:"清除",
  stuName:"學生姓名",
  createRecord: "創建記錄",
  deleteRecord:"刪除記錄",
  missingNameContent:"學生姓名為空",
  minLargeThanMax:"最小值大於最大值"
};
const eng ={
  addRecord: "Add Record",
  searching: "Filter Condition",
  name:"Name",
  score:"Score",
  class:"Class",
  filter:"Filter",
  clear:"Clear",
  stuName:"Student Name",
  createRecord: "Create Record",
  deleteRecord:"Delete Record",
  missingNameContent:"Student Name is Empty",
  minLargeThanMax:"Minimum large than Maximum"
}
const getResource =(key)=>{
  if(lang.value === "eng"){
    return eng[key];
  }else {
    return zh[key];
  }
}



</script>

<template>
  <el-row justify="end">
    <el-col :span="4">
      <el-select v-model="lang" >
        <el-option v-for="item in langOptions.values()" :key="item.key" :label="item.label" :value="item.key">
        </el-option>
      </el-select>
    </el-col>

  </el-row>


  <div>
    <span>{{ getResource("addRecord") }}:</span>
  </div>
  <el-form :inline="true" :model="record" label-width="120px">
    <el-form-item :label="getResource('stuName')">
      <el-input v-model="record.name" />
    </el-form-item>
    <el-form-item :label="getResource('score')">
      <el-input type="number" v-model="record.score" :min="0" />
    </el-form-item>
    <el-form-item :label="getResource('class')">
      <el-radio-group v-model="record.class">
        <el-radio label="A" />
        <el-radio label="B" />
        <el-radio label="C" />
      </el-radio-group>
    </el-form-item>
    <el-form-item>
      <el-button @click="createRecord" type="primary">{{ getResource('createRecord') }}</el-button>
    </el-form-item>
  </el-form>

  <br/>

  <div>
    <span>{{ getResource("searching") }}:</span>
  </div>
  <el-form :inline="true" :model="filter" label-width="120px">
    <el-form-item :label="getResource('score')">
      <div>
        <el-input v-model="filter.min" style="display:inline"  type="number" :min="0"  :disabled="!buttonToggle"  />
        -
        <el-input v-model="filter.max" style="display:inline" type="number" :min="0" :disabled="!buttonToggle"/>
      </div>
    </el-form-item>
    <el-form-item :label="getResource('class')">
      <el-radio-group v-model="filter.class" :disabled="!buttonToggle">
        <el-radio label="A" />
        <el-radio label="B" />
        <el-radio label="C" />
      </el-radio-group>
    </el-form-item>
    <el-form-item>
      <el-button @click="filterRecord" v-if="buttonToggle">{{ getResource('filter') }}</el-button>
      <el-button @click="clearFilter" type="danger" v-if="!buttonToggle">{{ getResource('clear') }}</el-button>
    </el-form-item>
  </el-form>

  <el-table :data="displayArray" border style="width: 100%">
    <el-table-column prop="name" :label="getResource('name')" />
    <el-table-column prop="score" :label="getResource('score')" />
    <el-table-column prop="class" :label="getResource('class')" />
    <el-table-column fixed="right"  width="120">
      <template #default="scope">
        <el-button link type="primary" size="small" @click.prevent="deleteRow(scope.row.id)">
          {{ getResource('deleteRecord') }}
        </el-button>
      </template>
    </el-table-column>
  </el-table>


</template>

<style scoped>

</style>
