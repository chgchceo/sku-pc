<script setup>
import Sku from './components/Sku.vue'
import {onMounted,ref} from 'vue'
import axios from 'axios'

// 商品数据
const goods = ref({})

const getGoods = async () => {
  // 1135076  初始化就有无库存的规格
  // 1369155859933827074 更新之后有无库存项（蓝色-20cm-中国）
  const res = await axios.get('http://pcapi-xiaotuxian-front-devtest.itheima.net/goods?id=1369155859933827074')
  goods.value = res.data.result

}
onMounted(() => getGoods())


const getSkuResult = (obj)=>{

  console.log(obj);
}



</script>

<template>
  
  <Sku v-if="goods.specs" :goods="goods" @getSkuClick="getSkuResult"/>

</template>

<style>



</style>
