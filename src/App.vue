<script setup lang="ts">

import {computed, reactive, Ref, ref} from "vue";

interface Spec {
  title: string;
  list: string[];
}

interface SpecCombination {
  id: string;
  specs: string[];
  weight?: number;
}

interface SpecWeight {
  [key: string]: {
    weight: number;
    sort: number;
  };
}

interface SelectedSpec {
  name: string;
  weight: number;
  sort: number;
}

const specList: Spec[] = [
  {title: "颜色", list: ["红色", "紫色"]},
  {title: "套餐", list: ["套餐一", "套餐二"]},
  {title: "内存", list: ["64G", "128G", "256G"]},
  {title: "尺寸", list: ["5.5寸", "6寸", "6.5寸"]}
]


const specs = ref<SpecWeight>({});
const selectedSpecs = reactive<SelectedSpec[]>([]);

const specCombinationList: Ref<SpecCombination[]> = ref([
  {id: "1", specs: ["紫色", "套餐一", "64G", "5.5寸"]},
  {id: "2", specs: ["紫色", "套餐一", "128G", "5.5寸"]},
  {id: "3", specs: ["紫色", "套餐二", "128G", "6寸"]},
  {id: "4", specs: ["红色", "套餐一", "128G", "6寸"]},
  {id: "4", specs: ["红色", "套餐二", "256G", "6.5寸"]}
]);


const init = () => {
  // 构造规格权重
  let i = 0, j = 0;
  specList.forEach(spec => {
    spec.list.forEach(item => {
      specs.value[item] = {
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
      weight: item.specs.reduce((prev, current) => prev + specs.value[current].weight, 0),
    };
  })
}

init();

const select = (item: string) => {
  const spec = specs.value[item];
  const eleIndex = selectedSpecs.findIndex(o => o.name === item);
  const sortIndex = selectedSpecs.findIndex(o => o.sort === spec.sort);
  if (eleIndex > -1) {
    selectedSpecs.splice(eleIndex, 1);
    return;
  }
  if (sortIndex > -1) {
    selectedSpecs.splice(sortIndex, 1);
  }
  selectedSpecs.push({name: item, weight: spec.weight, sort: spec.sort});
}

const disabled = computed(() => {
  return (item: string) => {
    // 每次选中之后要判断其他规格是否能够被选中
    const weights = selectedSpecs.reduce((prev, current) => prev + current.weight, 0);
    if (weights === 0) return false;
    const lastSelected = selectedSpecs[selectedSpecs.length - 1];
    const current = specs.value[item];
    console.log(lastSelected, current);
    const weight = current.weight;
    let sum = weight + weights;
    if (current.sort === lastSelected.sort) {
      sum -= lastSelected.weight;
    }
    for (let i = 0; i < specCombinationList.value.length; i++) {
      const sWeight = specCombinationList.value[i].weight!;
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
    <div>您选中了：{{ selectedSpecs.sort((a, b) => a.sort - b.sort).map(o => o.name).join("，") }}</div>
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
