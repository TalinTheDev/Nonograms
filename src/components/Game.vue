<script setup>
import {reactive  } from 'vue';
import {Timer} from '../helpers/timer.js'
const props = defineProps(['size'])

const rowCount = parseInt(props.size.split('x')[0])
const columnCount = parseInt(props.size.split('x')[1])
var notStarted = true;
var matrix = Array(parseInt(props.size.split('x')[0])).fill(null).map(() => Array(parseInt(props.size.split('x')[1])).fill(0));
var values = {
  rows: new Array(parseInt(props.size.split('x')[0])).fill(0),
  columns: new Array(parseInt(props.size.split('x')[1])).fill(0),
}
const timer = reactive(new Timer());
var columns = [];
for (var i = 0; i < columnCount; i++) {
  columns.push([])
}

for (var i = 0; i < matrix.length; i++) {
  var row = matrix[i];
  for (var j = 0; j < row.length; j++) {
    row[j] = (Math.floor(Math.random() * 24) + 1) % 2 != 0 ? true : false;
    columns[j][i] = row[j]
  }
  var rowValues = row.join("").replaceAll("false", " ")
  rowValues = rowValues.split(" ")
  rowValues.forEach((filledInSection, counter) => {
    var length = (filledInSection.match(/true/g) || []).length
    if (length != 0) rowValues[counter] = length;
  })
  var value = rowValues.join().replaceAll(",", " ").trim()
  values.rows[i] = value != "" ? value : "0"
}

for (var i = 0; i < columns.length; i++) {
  var column = columns[i];
  var columnValues = column.join("").replaceAll("false", " ")
  columnValues = columnValues.split(" ")
  columnValues.forEach((filledInSection, counter) => {
    var length = (filledInSection.match(/true/g) || []).length
    if (length != 0) columnValues[counter] = length;
  })
  var value = columnValues.join().replaceAll(",", " ").trim().replaceAll(" ", "<br/>");
  value = value.replace(/(<br\/>\s*)+/g, '<br/>');
  values.columns[i] = value != "" ? value : "0"
}

function check() {
  var correct = true
  for (var i = 0; i < matrix.length; i++) {
    var row = matrix[i];
    for (var j = 0; j < row.length; j++) {
        var block = document.getElementById(`${i+1}x${j+1}`).classList.contains("bg-green-300");
        var selected = matrix[i][j];

        if(selected && block){ // If it is supposed to be selected and is
          continue
        }
        else if(!selected && block){ // If it isn't supposed to be and is
          correct = false;
          return correct;
        }
        else if(selected && !block){ // If it is suppsoed to be and it isn't
          correct = false;
          return correct;
        }
    }
  }
  return correct
}

function clicked(blockID) {
  if(notStarted){
    notStarted = false;
    timer.start()
  }
  var block = document.getElementById(blockID)
  if (block.classList.contains("bg-green-300")) {
    block.classList.remove("bg-green-300")
    block.classList.add("bg-red-500")
  }
  else if (block.classList.contains("bg-red-500")) {
    block.classList.remove("bg-red-500")
  }
  else {
    block.classList.add("bg-green-300")
  }
  if(check()){
    timer.stop()
    setTimeout(function(){document.getElementById("modal").classList.remove("hidden")}, 500)
  }
}

function restart(){
  document.getElementById("modal").classList.add("hidden");
}
</script>

<template>
 <div class="grid place-items-center mt-[12vh]">
   <h1 class="text-5xl underline underline-offset-[13px] font-extrabold text-center mb-[35px]">Play Nonograms</h1>
   <table class="table-auto border-spacing-1 border-separate">
     <thead>
       <tr>
         <td></td>
         <td class="font-bold text-center" v-for="f in columnCount" v-html="values.columns[f - 1]"></td>
       </tr>
     </thead>
     <tbody>
       <tr v-for="n in rowCount">
         <td class="font-bold text-center">{{ values.rows[n - 1] }}</td>
         <td class="border border-black w-[50px] h-[50px]" :id="`${n}x${f}`" v-for="f in columnCount" @click="clicked(`${n}x${f}`)"></td>
       </tr>
     </tbody>
   </table>
 </div>
 <div class="fixed hidden inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full" id="modal">
   <div class="relative top-[25vh] mx-auto p-5 border w-[50vw] shadow-lg rounded-md bg-white">
     <div class="mt-3 text-center">
       <div class="mx-auto flex items-center justify-center h-12 w-12 rounded-full bg-green-100">
         <svg class="h-6 w-6 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
           <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
         </svg>
       </div>
       <h3 class="text-lg leading-6 font-medium text-gray-900">You Won!</h3>
       <div class="mt-2 px-7 py-3">
         <p class="text-sm text-gray-500">Nonogram solved in: {{ new Date(timer.getTime()).getSeconds() }} Seconds</p>
       </div>
       <div class="items-center px-4 py-3">
         <button @click="restart()" class="px-4 py-2 bg-green-500 text-white text-base font-medium rounded-md w-full shadow-sm hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-300"> Restart (doesn't yet work)</button>
       </div>
     </div>
   </div>
 </div>
</template>

<style scoped>

</style>
