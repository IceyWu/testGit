<template>
  <div>对比结果：{{ isDiff }}</div>
</template>

<script setup>
import { ref } from 'vue'
let array1 = [
  { name: 1, id: 1 },
  { name: 2, id: 2 },
]
let array2 = [
  { name: 1, id: 1 },
  { name: 2, id: 2 },

  { name: 1, id: 1 },
  { name: 2, id: 2 },
]
let isDiff = ref(false)

// 判断两个数组的name属性是交集还是并集
let intersection = (array1, array2) => {
  let arr = []
  array1.forEach((item) => {
    array2.forEach((item2) => {
      if (item.name === item2.name) {
        arr.push(item)
      }
    })
  })
  return arr
}
// 对比两数组的对应属性是否相等，返回true或false
const getArrayDiff = (oldArray, newArray, fieldArray) => {
  let isEqual = false
  if (oldArray.length !== newArray.length) {
    isEqual = false
  } else {
    isEqual = true
    oldArray.forEach((item, index) => {
      fieldArray.forEach((field) => {
        let findRe = newArray.find((item2) => {
          return item2[field] === item[field]
        })
        if (!findRe) {
          isEqual = false
        }
      })
    })
  }

  return isEqual
}
isDiff.value = getArrayDiff(array1, array2, ['name', 'id'])
console.log('--对比结果--', isDiff.value)
</script>

<style scoped></style>
