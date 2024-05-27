
<script setup>
import { onMounted, ref ,defineEmits} from 'vue'

import powerSet from '../power-set'


  const res = defineProps({

  goods:{
    type:Object,
    default:{}
  },

})
const emit = defineEmits(['getSkuClick'])
// 商品数据
const goods = ref({})
let pathMap = {}
const getGoods = () => {

  goods.value = res.goods
  pathMap = getPathMap(goods.value)
  initDisabledStatus(goods.value.specs,pathMap)
}
onMounted(() => getGoods())

//点击选项
const changeSeletedState = (item,val)=>{

  if(val.disabled)return;
    if(val.selected){

        val.selected = false;
    }else{

        item.values.forEach(v => v.selected = false );
        val.selected = true;
    }

    //更新禁止状态
    upDateDisabledStatus(goods.value.specs,pathMap)

    //产出选中的有效skuID结果

    const selectedValues = getSeletedValues(goods.value.specs)

    const index = selectedValues.findIndex(item => item === undefined)
    if(index > -1){
      //找到了undefined,没有全部选完

    }else{//完成
      const key = selectedValues.join('-')
      const skuIds = pathMap[key]
      const skuObj = goods.value.skus.find(item => item.id === skuIds[0])
      // console.log(skuObj);

      // defineEmits('get')
      

      emit('getSkuClick',skuObj)
    }

}

//生成有效路径字典对象

const getPathMap = (goods)=>{

  const pathMap = {}
  //1根据shus字段生成有效的sku数组
  // console.log(goods?.skus,'======');
  const effectiveSkus = goods.skus.filter(sku => sku.inventory >0);

  
  effectiveSkus.forEach(sku =>{
    //2.1
    const vals = sku.specs.map(val => val.valueName)
    //2.2
    const arr1 = powerSet(vals)
    // console.log(arr1,'hahahahah');

    //3
    arr1.forEach(arr=>{

      const key = arr.join('-')
      if(pathMap[key]){

        pathMap[key].push(sku.id)
      }else{

        pathMap[key] = [sku.id]
      }
    })
  })

  // console.log(pathMap);
  return pathMap
}

//初始化禁用状态
const initDisabledStatus = (specs,pathMap)=>{

  specs.forEach(spec=>{

    spec.values.forEach(val =>{

      if(pathMap[val.name]){

        val.disabled = false;

      }else{

        val.disabled = true;
      }
    })

  })
}

//获取选中的匹配数组
const getSeletedValues = (specs)=>{

  const arr = []

  specs.forEach(spec =>{

    const selectedVal = spec.values.find( val => val.selected)

    arr.push(selectedVal? selectedVal.name :undefined)

  })

  return arr
}

//切换时更新禁用状态

const upDateDisabledStatus = (specs,pathMap)=>{

  
  specs.forEach((spec,index)=>{

    const seletedValues = getSeletedValues(specs)
    spec.values.forEach(val =>{
      //对每一项都进行验证

      seletedValues[index] = val.name
      const key = seletedValues.filter(value=>value).join('-')

      if(pathMap[key]){

        val.disabled = false;
      }else{

        val.disabled = true;
      }
    })

  })

}


</script>

<template>
  <div v-show="goods?.specs" class="goods-sku">
    <dl v-for="item in goods?.specs" :key="item.id">
      <dt>{{ item.name }}</dt>
      <dd>
        <template v-for="val in item.values" :key="val.name">
          <!-- 图片类型规格 -->
          <img :class="{selected:val.selected ,disabled:val.disabled}" @click="changeSeletedState(item,val)" v-if="val.picture" :src="val.picture" :title="val.name">
          <!-- 文字类型规格 -->
          <span :class="{selected:val.selected,disabled:val.disabled}" @click="changeSeletedState(item,val)" v-else>{{ val.name }}</span>
        </template>
      </dd>
    </dl>
  </div>
</template>

<!--  -->
<style scoped lang="scss">
@mixin sku-state-mixin {
  border: 1px solid #e4e4e4;
  margin-right: 10px;
  cursor: pointer;

  &.selected {
    border-color: #27ba9b;
  }

  &.disabled {
    opacity: 0.6;
    border-style: dashed;
    cursor: not-allowed;
  }
}

.goods-sku {
  padding-left: 10px;
  padding-top: 20px;

  dl {
    display: flex;
    padding-bottom: 20px;
    align-items: center;

    dt {
      width: 50px;
      color: #999;
    }

    dd {
      flex: 1;
      color: #666;

      >img {
        width: 50px;
        height: 50px;
        margin-bottom: 4px;
        @include sku-state-mixin;
      }

      >span {
        display: inline-block;
        height: 30px;
        line-height: 28px;
        padding: 0 20px;
        margin-bottom: 4px;
        @include sku-state-mixin;
      }
    }
  }
}
</style>