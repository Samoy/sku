<script setup lang="ts">

import { computed, reactive, Ref, ref } from "vue";

type Spec = {
  title: string;
  list: string[];
}

type SpecCombination = {
  id: string;
  specs: string[];
  weight?: number;
}

type SpecWeight = Record<string, {
  weight: number;
  sort: number;
}>

type SelectedSpec = {
  name: string;
  weight: number;
  sort: number;
}

const specList: Spec[] = [
  { title: "颜色", list: ["红色", "紫色"] },
  { title: "套餐", list: ["套餐一", "套餐二"] },
  { title: "储存", list: ["64G", "128G", "256G"] },
  { title: "尺寸", list: ["5.5寸", "6寸", "6.5寸"] }
]


const specMap = ref<SpecWeight>({});
const selectedSpecs = reactive<SelectedSpec[]>([]);

const specCombinationList: Ref<SpecCombination[]> = ref([
  { id: "1", specs: ["紫色", "套餐一", "64G", "5.5寸"] },
  { id: "2", specs: ["紫色", "套餐一", "128G", "5.5寸"] },
  { id: "3", specs: ["紫色", "套餐二", "128G", "6寸"] },
  { id: "4", specs: ["红色", "套餐一", "128G", "6寸"] },
  { id: "4", specs: ["红色", "套餐二", "256G", "6.5寸"] }
]);


const init = () => {
  // 构造规格权重
  let i = 0, j = 0;
  specList.forEach(spec => {
    spec.list.forEach(item => {
      specMap.value[item] = {
        sort: i,
        weight: 1 << j
      }
      j++;
    })
    i++;
  })
  // 构造规格组合
  specCombinationList.value = specCombinationList.value.map(item => {
    return {
      ...item,
      weight: item.specs.reduce((prev, current) => prev + specMap.value[current].weight, 0),
    };
  })
}

init();

const select = (item: string) => {
  const spec = specMap.value[item];
  const eleIndex = selectedSpecs.findIndex(o => o.name === item);
  const sortIndex = selectedSpecs.findIndex(o => o.sort === spec.sort);
  // 如果当前元素已经存在，则从列表中删除
  if (eleIndex > -1) {
    selectedSpecs.splice(eleIndex, 1);
    return;
  }
  // 如果列表中的元素存在和当前元素的维度相同，则从列表中把那个元素删除，保证同一纬度只有一个元素。
  if (sortIndex > -1) {
    selectedSpecs.splice(sortIndex, 1);
  }
  selectedSpecs.push({ name: item, weight: spec.weight, sort: spec.sort });
}

const disabled = computed(() => {
  return (item: string) => {
    // 每次选中之后要判断其他规格是否能够被选中
    const weights = selectedSpecs.reduce((prev, current) => prev + current.weight, 0);
    // 所有是初始状态，则所有属性均可选，因此disabled为false.
    if (weights === 0) return false;
    const current = specMap.value[item];
    const weight = current.weight;
    // 假如选择该属性，这是选中后的“权重之和”
    let sum = weight + weights;
    // 找到是否存在同一维度的规格已被选中的规格
    const existEle = selectedSpecs.find(o => o.sort === current.sort);
    if (existEle) {
      // 如果存在，则减去即前选中的规格权重，即替换为当前权重。
      sum -= existEle.weight;
    }
    for (let i = 0; i < specCombinationList.value.length; i++) {
      const sWeight = specCombinationList.value[i].weight!;
      // 与“权重之和”按位与运算后与其进行比较，如果相等，说明该属性是可选的
      if ((sWeight & sum) === sum) {
        return false;
      }
    }
    return selectedSpecs.findIndex(o => o.name === item) == -1;
  };
})

const selected = computed(() => {
  return (item: string) => {
    return selectedSpecs.findIndex(o => o.name === item) > -1;
  };
})

</script>

<template>
  <div class="panel">
    <div class="row">
      <div v-for="(item) in specCombinationList">
        <div class="button" :key="item.id">
          可选规格： {{ item.specs.join("，") }}
        </div>
      </div>
    </div>
    <div class="row" v-for="spec in specList" :key="spec.title">
      {{ spec.title }}：
      <button v-for="(item) in spec.list" class="button" :key="item" @click="select(item)" :disabled="disabled(item)"
        :data-selected="selected(item)">
        {{ item }}
      </button>
    </div>
    <div>您选中了：{{ Object.assign(selectedSpecs, []).sort((a, b) => a.sort - b.sort).map(o => o.name).join("，") }}</div>
  </div>
</template>

<style scoped>
.panel {
  text-align: start;

  .button {
    margin: 0 12px 12px 0;

    &:disabled {
      pointer-events: none;
    }
  }
}
</style>
